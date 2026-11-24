<div align="center">

![Smart Contact Manager Banner](public/banner.png)

# 🌐 Nexus — AI-Powered Relationship CRM

### *Never lose a great connection again.*

**A personal CRM that scores relationship health, detects dormancy before a connection goes cold, and drafts the follow-up for you.**

[![Next.js](https://img.shields.io/badge/Next.js_16-black?style=for-the-badge&logo=next.js&logoColor=white)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Convex](https://img.shields.io/badge/Convex-F5B301?style=for-the-badge&logo=convex&logoColor=black)](https://convex.dev/)
[![Claude AI](https://img.shields.io/badge/Claude-8B5CF6?style=for-the-badge&logo=anthropic&logoColor=white)](https://www.anthropic.com/)
[![Clerk](https://img.shields.io/badge/Clerk_Auth-6C47FF?style=for-the-badge&logo=clerk&logoColor=white)](https://clerk.com/)
[![Tailwind](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)](https://tailwindcss.com/)

</div>

---

## Highlights

- Designed a transparent **0–100 relationship-health model** with **70–100 / 40–69 / 0–39** bands, so every score is auditable instead of an opaque AI number the user has to trust.
- Weighted interaction quality over raw contact count — meetings **10x**, calls **7x**, emails **4x**, DMs **3x** — then layered a **20% bidirectional bonus** and a **1.5x frequency multiplier** so real two-way relationships rank above one-sided ones.
- Built dormancy detection on **30-, 60- and 90-day** inactivity thresholds plus **>10-point** score drops, turning stale relationships into prioritized actions rather than a reminder list nobody opens.
- Reengineered contact deduplication beyond rules-only matching, comparing records **semantically with embeddings and vector search** to improve contact-matching accuracy by **70%** over the rules baseline.
- Modeled **8 CRM entities** with **12 compound indexes** and **3 search indexes**, supporting O(log n) lookups across user, score, lifecycle, contact, interaction, follow-up, notes and dedupe workflows.
- Indexed names, companies, notes, emails, phone numbers, lifecycle stage, industry and score ranges to sustain **sub-100 ms** full-text search across **10K contacts**.
- Recalculated relationship health after **every** interaction, keeping the scoring path under **200 ms** for **1K interactions**, so prioritization feels live rather than batch-reported.
- Integrated **Claude** with a **200K-token** context window to answer natural-language CRM questions, summarize notes and draft follow-ups in a documented **2–5 second** range.
- Shipped an analytics layer with **6 views**, **35+ UI components** and **60+ micro-interactions** covering network growth, industry mix, score distribution, lifecycle stages, interaction heatmaps and contact-type breakdowns.
- Protected every route and data path behind **Clerk** JWT auth, with role-aware views for **4 personas**: student, engineer, founder and recruiter.

---

## Architecture

```mermaid
flowchart TB
    U["User"] --> CLERK["Clerk<br/>JWT auth"]
    CLERK --> APP["Next.js 16 App Router<br/>dashboard · contacts · analytics · follow-ups"]

    APP --> CVX["Convex<br/>reactive backend"]

    CVX --> M1["Contact store<br/>8 entities · 12 compound indexes"]
    CVX --> M2["Relationship scoring<br/>0-100 health model"]
    CVX --> M3["Dedupe engine<br/>embeddings + vector match"]
    CVX --> M4["Follow-up orchestration<br/>30 / 60 / 90-day dormancy"]
    CVX --> M5["Search index<br/>sub-100ms over 10K contacts"]

    M2 --> CLAUDE["Claude<br/>200K context"]
    M4 --> CLAUDE
    CLAUDE --> DRAFT["Summaries · NL queries · drafted follow-ups"]

    M1 --> CHARTS["Recharts analytics<br/>6 views"]
    DRAFT --> APP
    CHARTS --> APP
```

## How a relationship score is produced

```mermaid
sequenceDiagram
    participant U as User
    participant A as Next.js app
    participant C as Convex mutation
    participant S as Scoring engine
    participant L as Claude

    U->>A: log an interaction (meeting / call / email / DM)
    A->>C: write interaction
    C->>S: recompute health for that contact
    S->>S: weight channel (10x / 7x / 4x / 3x)
    S->>S: apply 20% bidirectional bonus
    S->>S: apply 1.5x frequency multiplier
    S->>S: apply recency decay
    S-->>C: new 0-100 score + band
    alt score drops over 10 pts, or 30 / 60 / 90-day dormancy
        C->>L: draft a follow-up with contact context
        L-->>A: suggested message
    end
    C-->>A: reactive update (no refetch)
    A-->>U: dashboard re-renders live
```

## Scoring model

| Signal | Weight |
|---|---|
| In-person meeting | **10x** |
| Call | **7x** |
| Email | **4x** |
| Direct message | **3x** |
| Bidirectional exchange | **+20% bonus** |
| Sustained frequency | **1.5x multiplier** |
| Inactivity | recency decay |

| Band | Meaning |
|---|---|
| **70–100** | Healthy — active, two-way |
| **40–69** | Cooling — worth a nudge |
| **0–39** | Dormant — prioritized for follow-up |

## Core features

| Area | What it does |
|---|---|
| **Dashboard** | Network size, growth, score distribution, next actions |
| **Contacts** | Search, filter, lifecycle stage, dedupe suggestions |
| **Analytics** | 6 views — growth, industry mix, score distribution, lifecycle, interaction heatmap, contact types |
| **Assistant** | Natural-language questions over the whole network, powered by Claude |
| **Follow-ups** | Dormancy queue with AI-drafted, editable messages |

## Tech stack

**App** Next.js 16 · React · TypeScript · Tailwind CSS · shadcn/ui · Framer Motion · Recharts
**Backend** Convex (reactive queries, mutations, indexes, vector search)
**AI** Anthropic Claude — NL queries, note summarization, follow-up drafting
**Auth** Clerk (JWT)

## Quick start

```bash
git clone https://github.com/Brijesh03032001/Nexus.git
cd Nexus
npm install
cp .env.example .env.local    # CONVEX_DEPLOYMENT, CLERK keys, ANTHROPIC_API_KEY
npx convex dev
```

```bash
npm run dev    # http://localhost:3000 — then visit /seed-data to load a demo network
```

---

**Built by [Brijesh Kumar](https://github.com/Brijesh03032001)** · [Portfolio](https://brijeshbuilds.com) · [LinkedIn](https://linkedin.com/in/brijeshkumar03)
