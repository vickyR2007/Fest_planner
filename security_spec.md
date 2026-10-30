# Security Specification: FESTOS AI Firestore Rules

## 1. Data Invariants

1. **Master Gate Relationship**:
   - Every `Event`, `Venue`, and `BudgetItem` subcollection document strictly belongs to a parent `/fests/{festId}`.
   - An event or venue cannot be created, modified, or accessed without ownership validation on `/fests/{festId}`.
   - Modifying sub-resources requires that `request.auth.uid` matches the owner of the parent fest or the document's `ownerId`.

2. **Identity Integrity**:
   - `ownerId` must equal `request.auth.uid`.
   - `ownerId` is immutable on update (`incoming().ownerId == existing().ownerId`).
   - Fest ID in document body must match path parameter (`incoming().festId == festId`).

3. **Temporal Integrity**:
   - For write operations, timestamp strings must exist and adhere to length and structure.

4. **Resource Bounds & Schema Hardening**:
   - IDs must match `^[a-zA-Z0-9_\-]+$` and length <= 128 characters.
   - Strings have bounded lengths (Title <= 150, Description <= 2000, Names <= 100).
   - Numerical values (`capacity`, `totalBudget`, `estimatedCost`, `expectedParticipants`) must be >= 0.

5. **Safe List Query Guard**:
   - `list` operations enforce `resource.data.ownerId == request.auth.uid` or parent fest ownership to prevent cross-tenant data scraping.

---

## 2. The "Dirty Dozen" Vulnerability Payloads

1. **Payload 1 (Shadow Field Injection)**:
   Attempt to write `{ ...validFest, __shadowRole: "superadmin" }`.
   *Expectation*: REJECTED due to strict `keys().hasOnly()`.

2. **Payload 2 (Identity Spoofing - Owner Hijack)**:
   Attacker UID `victim_123` tries to create fest with `ownerId: "attacker_456"`.
   *Expectation*: REJECTED because `incoming().ownerId != request.auth.uid`.

3. **Payload 3 (Unauthenticated Read)**:
   Guest request to `/fests/secret-fest-1`.
   *Expectation*: REJECTED because `request.auth == null`.

4. **Payload 4 (Cross-Tenant Event Tampering)**:
   User B tries to update User A's `/fests/festA/events/event1`.
   *Expectation*: REJECTED because User B is not the fest owner or event owner.

5. **Payload 5 (Path Traversal / Junk ID Injection)**:
   Attempt to create document at `/fests/../../root/hacked` or with a 2KB junk character ID.
   *Expectation*: REJECTED by `isValidId()` regex `^[a-zA-Z0-9_\-]+$`.

6. **Payload 6 (Owner Field Mutation on Update)**:
   User updates an event and attempts to reassign `ownerId: "new_owner"`.
   *Expectation*: REJECTED because `incoming().ownerId == existing().ownerId` is strictly enforced.

7. **Payload 7 (Massive String Resource Exhaustion / Denial of Wallet)**:
   Attempt to store 2MB event description.
   *Expectation*: REJECTED because `description.size() <= 2000`.

8. **Payload 8 (Negative Numerical Bounds Violation)**:
   Attempt to create event with `estimatedCost: -50000` or `expectedParticipants: -10`.
   *Expectation*: REJECTED by `>= 0` numerical range guard.

9. **Payload 9 (Parent Fest Mismatch / Orphaned Write)**:
   Creating an event under `/fests/festA/events/ev1` with `festId: "festB"`.
   *Expectation*: REJECTED by `incoming().festId == festId`.

10. **Payload 10 (Invalid Enum Status Injection)**:
    Setting fest status to `status: "arbitrary_pwned"`.
    *Expectation*: REJECTED because status must be in `['draft', 'planning', 'live', 'concluded']`.

11. **Payload 11 (Unbounded Array Injection)**:
    Submitting `facilities` list with 500 items.
    *Expectation*: REJECTED because list length must be `<= 30`.

12. **Payload 12 (Blanket Query Scraping)**:
    Querying `/fests` without filtering by owner.
    *Expectation*: REJECTED by secure list rule `resource.data.ownerId == request.auth.uid`.
