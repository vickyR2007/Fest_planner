# FESTOS AI - Intelligent College Fest Planning Web Application

Built for Indian engineering colleges (IITs, NITs, BITS, and technical universities) to plan, schedule, budget, and execute multi-day techno-cultural festivals.

## Highlights & Features

- **Master Overview Dashboard**: Real-time KPI metrics for total events, INR budget consumption, venue occupancy rates, and active conflict alerts.
- **Fest Management**: Create and configure college festivals with multi-day date spans, committee allocation, and total budget allocations. Includes 1-click **TechNova 2026** demo loader.
- **Campus Venue Management**: Real-time management of campus auditoriums, open air theatres, computer centers, and seminar halls with capacity constraints, operating hours, and audiovisual equipment tags.
- **Event Management**: Complete CRUD for college events across technical, cultural, sports, workshops, and keynotes. Validates capacity, duration, and logistics requirements.
- **Deterministic Smart Scheduler**:
  - Automatically identifies overlapping bookings at the same venue (`max(startA, startB) < min(endA, endB)`).
  - Flags capacity overflows when expected footfall exceeds hall limits.
  - Detects turnaround violations (< 30 minutes buffer for sound check and stage reset).
  - 1-click **Auto-Schedule** greedy engine to place unassigned events into matching venue slots without conflicts.
- **Budget Planner & Analytics**:
  - Full Indian Rupee (INR ₹) formatting with Lakhs/Thousands breakdown.
  - Interactive Recharts breakdown by category (Tech Infrastructure, Prizes, Stage & AV, Logistics, Marketing, Reserves).
  - Over-budget warning triggers and burn rate trackers.
- **Fest Chatbot**:
  - Floating chat widget calls Groq with the active fest profile, events, venues, and budget for natural-language answers.
  - Falls back to local data lookups if Groq is unavailable and does not invent missing fest details.
- **Security & Authorization**:
  - Firebase Authentication with Google Sign-In.
  - Hardened Cloud Firestore Security Rules implementing Attribute-Based Access Control and Master Gate relational integrity.

## Tech Stack

- **Frontend**: React 19, TypeScript, Vite, Tailwind CSS v4, Lucide React, Recharts
- **Backend**: Express server with Vite middleware mode, Node 22+
- **Database & Auth**: Cloud Firestore, Firebase Authentication
- **AI Integration**: Groq Chat Completions API (model `openai/gpt-oss-20b`)
- **Testing**: `node:test` and `node:assert/strict` unit tests

## Getting Started

1. Install dependencies:
   ```bash
   npm install
   ```

2. To enable AI-generated answers, copy `.env.example` to `.env` and add your own `GROQ_API_KEY` to `.env`. Keep `.env` private and never commit it. Without the key, the local fest FAQ fallback remains available.

3. Run unit tests:
   ```bash
   npm test
   ```

4. Start development server:
   ```bash
   npm run dev
   ```
   Open `http://localhost:3000` in your browser.
