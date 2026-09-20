<div align="center">

![Smart Contact Manager Banner](public/banner.png)

# 🌐 Smart Contact Manager

### *AI-Powered Professional Network Intelligence Platform*

**Never lose a great connection again. Build, track, and nurture relationships with AI-driven insights.**

[![Next.js](https://img.shields.io/badge/Next.js_16-black?style=for-the-badge&logo=next.js&logoColor=white)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Convex](https://img.shields.io/badge/Convex-F5B301?style=for-the-badge&logo=convex&logoColor=black)](https://convex.dev/)
[![Claude AI](https://img.shields.io/badge/Claude_3.5_Sonnet-8B5CF6?style=for-the-badge&logo=anthropic&logoColor=white)](https://www.anthropic.com/)
[![Clerk](https://img.shields.io/badge/Clerk_Auth-6C47FF?style=for-the-badge&logo=clerk&logoColor=white)](https://clerk.com/)
[![Tailwind](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)](https://tailwindcss.com/)

[Documentation](#-getting-started) • [Architecture](#-system-architecture) • [Features](#-core-features)

---

</div>

## 📊 Project Overview

Smart Contact Manager is an **AI-powered CRM** designed for professionals, job seekers, founders, and networkers who value relationships. It combines a rule-based relationship-health engine, real-time database synchronization (via Convex), and a Claude 3.5 Sonnet-backed assistant to help you track contacts, spot relationships that need attention, and draft follow-ups.

### 🎯 Problem Statement

In today's hyper-connected world, maintaining meaningful professional relationships is challenging. Contacts get lost, follow-ups are forgotten, and network potential remains untapped. Traditional CRMs are bloated and enterprise-focused—not designed for individuals who need intelligent, automated relationship management.

### 💡 Solution

A lightweight, AI-first platform that:
- ✅ **Scores relationships** on a 0-100 scale based on interaction recency and frequency
- ✅ **Flags at-risk connections** using rule-based lifecycle logic (new → active → dormant → archived)
- ✅ **Generates follow-up suggestions** based on inactivity thresholds and contact type
- ✅ **Integrates Claude 3.5 Sonnet** for contact-graph-aware chat, email drafting, and query parsing
- ✅ **Visualizes network analytics** with interactive, real-time charts

---

## ✨ Core Features

### 🧠 **1. AI-Powered Relationship Intelligence**
- **Relationship Scoring**: Every contact carries a 0-100 `relationshipScore`, recalculated via a scheduled Convex function each time an interaction is logged. The scoring function considers interaction frequency, recency, and channel type
- **Lifecycle Rules**: A separate rule-based engine (`convex/lifecycle.ts`) moves contacts through `new → active → dormant → archived` based on days since last contact (60-day and 1-year thresholds)
- Visual heatmaps and trend indicators

### 🤖 **2. AI Assistant (Claude 3.5 Sonnet integration)**
- **Context Assembly**: `convex/aiAssistant.ts` builds a system prompt from the user's live contact graph (contact list, relationship scores, lifecycle stages, at-risk contacts) on every request
- **Chat UI**: Supports natural-language queries such as:
  - *"Show me dormant recruiters in fintech"*
  - *"Who should I reconnect with this week?"*
  - *"Draft an email to follow up with Sarah about the partnership"*
- **Claude-Powered Actions**: `generateEmailDraft` and `parseSearchQuery` call the real Anthropic API (`claude-3-5-sonnet-20241022`) to draft emails and turn natural-language filters into structured queries
- **Current Chat Behavior**: The live chat endpoint (`chatWithAssistant`) currently returns rule-based, templated responses keyed off the user's message rather than a live Claude call — the real Anthropic API integration is written but commented out pending a production API key

### 📈 **3. Advanced Analytics & Data Visualizations**
Built with **Recharts** and **Framer Motion** for smooth, interactive dashboards:

#### 📊 **Network Growth Chart** (Line Chart)
- Tracks total contact count over 6 months
- Gradient line with animated hover states
- Custom tooltips with yellow/black theme
- Growth rate calculations

#### 📊 **Industry Distribution** (Bar Chart)
- Top 10 industries in your network
- Gradient-filled bars with rounded corners
- Identifies industry concentration & diversity
- Interactive click-to-filter

#### 📊 **Relationship Score Distribution** (Pie Chart)
- Score ranges: Strong (70-100), Medium (40-69), Weak (0-39)
- Interactive legend with hover effects
- Percentage breakdown with labels
- Color-coded by strength

#### 📊 **Lifecycle Stage Distribution** (Donut Chart)
- Contact stages: New, Active, Dormant, Archived
- Color-coded segments with smooth animations
- Click-to-filter functionality
- Real-time updates

#### 📊 **Interaction Frequency Heatmap** (Calendar View)
- GitHub-style contribution graph
- Daily interaction intensity visualization
- Hover tooltips with detailed breakdowns
- Color gradients based on activity

#### 📊 **Contact Type Breakdown** (Horizontal Bar)
- Auto-tagged categories: Recruiter, Investor, Client (detected via keyword matching on title/company), falling back to Other. "Friend" is a supported category in the schema but isn't auto-assigned by the current tagging logic
- Sortable by count or average relationship score
- Visual percentage indicators

### 🎯 **4. Smart Follow-Up Engine**
- **Automated Detection**: A scheduled Convex job (`computeSmartFollowUps`) flags contacts needing attention based on:
  - Inactivity thresholds (30 days for recruiters/clients, 45 days for active contacts)
  - Dormant contacts that still have a healthy relationship score (worth reviving)
  - High-scoring contacts (70+) with no contact in 30+ days
- **Due-Date Prioritization**: Follow-ups get a due date 1-7 days out based on urgency, and are surfaced in due-date order
- **Templated Reasons**: Each follow-up includes a plain-text reason (e.g., "No contact in 52 days") generated from the trigger rule, not by the AI assistant
- **Flexible Management**: Skip or mark as done

### 🔍 **5. Intelligent Search & Advanced Filtering**
- **Full-Text Search**: Lightning-fast indexed search across:
  - Contact names (fuzzy matching)
  - Company names
  - Notes and interaction history
  - Email addresses and phone numbers
- **Multi-Criteria Filters**:
  - Lifecycle stage (new, active, dormant, archived)
  - Contact type (recruiter, client, investor, friend)
  - Industry, location, title
  - Relationship score ranges (0-100)
  - Last interaction date ranges
  - Custom tag combinations
- **Natural Language Search**: AI-powered query parsing (*coming soon*)

### 🔄 **6. Smart Deduplication System**
- **Fuzzy Matching Algorithm**: Uses:
  - Levenshtein distance for name similarity
  - Email/phone overlap detection
  - Company name matching
  - Heuristic scoring (0-100% confidence)
- **Automated Suggestions**: Background job identifies duplicates
- **Manual Review Interface**: Accept, ignore, or merge duplicates
- **Merge Operations**: Preserves interaction history & notes

### 📝 **7. Comprehensive Interaction Tracking**
- **Multi-Channel Logging**: Email, Call, Meeting, DM, Other
- **Direction Tracking**: Inbound vs Outbound
- **Rich Context**: Add free-text notes per interaction (`content` field); a `sentimentScore` field exists in the schema but isn't populated yet
- **AI Summarization**: Schema supports an `aiSummary`/`aiNotesSummary` field for Claude-condensed notes, but it's currently only populated by seed data — no runtime summarization call exists yet
- **Timeline View**: Chronological interaction history per contact
- **Quick Add**: Speed-optimized forms for rapid logging

### 🎨 **8. Premium UI/UX Design**
- **Custom Dark Theme**: Yellow (#F5B301) / Black gradient palette
- **60+ Micro-Interactions**: Framer Motion animations on every component
- **Fully Responsive**: Mobile-first design, optimized for all devices
- **Accessibility**: 
  - Built on Radix UI primitives, which provide keyboard navigation and focus management out of the box
  - No formal WCAG audit or explicit ARIA labeling has been done yet — noted here as a gap, not a compliance claim
- **Performance Optimized**: 
  - Code splitting with React.lazy
  - Image optimization with Next.js Image
  - Skeleton loading states
  - Debounced search inputs

---

## 🏗️ System Architecture

### **High-Level Architecture Diagram**

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER (CSR + SSR)                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │   Next.js    │  │  TypeScript  │  │  Tailwind    │              │
│  │  App Router  │  │   + React    │  │     CSS      │              │
│  │   (v16.1)    │  │    (v19.2)   │  │   (v3.4)     │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
│  ┌──────────────────────────────────────────────────────┐          │
│  │         Component Library (shadcn/ui)                │          │
│  │  • Radix UI Primitives  • Lucide Icons  • Recharts  │          │
│  │  • Framer Motion        • date-fns                   │          │
│  └──────────────────────────────────────────────────────┘          │
└─────────────────────────────────────────────────────────────────────┘
                              ↕ 
                    HTTP/WebSocket (Realtime Sync)
                              ↕
┌─────────────────────────────────────────────────────────────────────┐
│                     AUTHENTICATION LAYER                             │
│  ┌──────────────────────────────────────────────────────┐          │
│  │              Clerk Auth (v6.36.5)                    │          │
│  │     • OAuth (Google, GitHub, LinkedIn)               │          │
│  │     • Email/Password • JWT Tokens • SSO              │          │
│  │     • Session Management • Middleware Protection     │          │
│  └──────────────────────────────────────────────────────┘          │
└─────────────────────────────────────────────────────────────────────┘
                              ↕ 
                    Convex Realtime Database Sync
                              ↕
┌─────────────────────────────────────────────────────────────────────┐
│                    BACKEND/DATABASE LAYER (Convex BaaS)              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐       │
│  │ Queries  │  │Mutations │  │ Actions  │  │  Scheduled   │       │
│  │(Reactive │  │ (Write)  │  │ (HTTP)   │  │  Functions   │       │
│  │  Reads)  │  │          │  │          │  │  (Cron)      │       │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────┘       │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │           Database Collections (6 Tables)                   │   │
│  │  • users            • contacts        • interactions        │   │
│  │  • followUps        • notes           • dedupeCandidates    │   │
│  │  ↓ 11 Compound Indexes  ↓ 3 Full-Text Search Indexes       │   │
│  └─────────────────────────────────────────────────────────────┘   │
│           Performance: Convex indexed lookups (unbenchmarked)        │
└─────────────────────────────────────────────────────────────────────┘
                              ↕ 
                          HTTP REST API
                              ↕
┌─────────────────────────────────────────────────────────────────────┐
│                         AI LAYER (External)                          │
│  ┌──────────────────────────────────────────────────────┐          │
│  │         Anthropic Claude 3.5 Sonnet API              │          │
│  │  • Email Draft Generation (live)                     │          │
│  │  • Natural Language Query Parsing (live)             │          │
│  │  • Chat Interface (currently rule-based, not live)   │          │
│  │  • Note Summarization (schema only, not wired up)    │          │
│  │  • Sentiment Analysis (Future)                       │          │
│  └──────────────────────────────────────────────────────┘          │
└─────────────────────────────────────────────────────────────────────┘
```

### **Data Flow Example: Adding a New Contact**

```
User Input (Form) 
  → Next.js Client Component
    → Convex Mutation (createContact)
      → Validation & Sanitization
        → Database Write (contacts table)
          → Trigger: computeRelationshipScore
            → Background Job: checkForDuplicates
              → Realtime Sync to All Clients
                → UI Update (Optimistic + Confirmed)
```

---

## 🛠️ Technology Stack

### **Frontend** (Client-Side + Server-Side Rendering)

| Technology | Version | Purpose & Highlights |
|------------|---------|---------------------|
| **Next.js** | 16.1.1 | React framework with App Router, SSR, SSG, and API routes |
| **React** | 19.2.3 | UI library with Server Components & Concurrent Rendering |
| **TypeScript** | 5.x | Type-safe development with strict mode |
| **Tailwind CSS** | 3.4.19 | Utility-first CSS framework with JIT compiler |
| **Framer Motion** | 12.23.26 | Production-ready animation library (60fps) |
| **Recharts** | 3.6.0 | Composable charting library built on D3 |
| **shadcn/ui** | Latest | Radix UI + Tailwind component system (`components/ui/`: badge, button, card, input, label, switch) |
| **Lucide React** | 0.562.0 | Beautiful icon library (1000+ SVG icons) |
| **date-fns** | 4.1.0 | Modern date utility library (tree-shakeable) |
| **clsx + tw-merge** | Latest | Dynamic className composition with conflict resolution |

**Why This Stack?**
- **Next.js 16**: Latest App Router with Turbopack (faster builds)
- **React 19**: Concurrent features for smoother UX
- **TypeScript**: Static typing catches errors before runtime
- **Tailwind**: Utility-first styling without maintaining separate CSS files

### **Backend** (Serverless Backend-as-a-Service)

| Technology | Version | Purpose & Highlights |
|------------|---------|---------------------|
| **Convex** | 1.31.2 | Realtime database with built-in reactive queries |
| **Convex Functions** | — | Type-safe serverless functions (Queries, Mutations, Actions) |
| **Convex Scheduler** | — | Event-triggered background jobs via `ctx.scheduler.runAfter` (e.g., recompute score after each interaction) |
| **Convex Auth** | — | Clerk identity passed through to Convex functions via `ctx.auth.getUserIdentity()` |
| **Convex Search** | — | Built-in full-text search indexes (contact name, company, notes) |

**Why Convex?**
- **Realtime by Default**: WebSocket subscriptions, no polling
- **Zero Config**: No schema migrations, automatic indexing
- **Type Safety**: End-to-end TypeScript (client ↔ server)
- **Serverless**: Auto-scales from 0 to millions of requests

### **AI/ML** (External Services)

| Technology | Version | Purpose & Highlights |
|------------|---------|---------------------|
| **Anthropic Claude** | 3.5 Sonnet | Model used for query parsing and email drafting; 200K context window |
| **@anthropic-ai/sdk** | 0.71.2 | Official Node.js SDK (streaming supported by the SDK; not used in this app) |

**Why Claude 3.5 Sonnet?**
- **Context Window**: 200K tokens (entire conversation history)
- **Reasoning**: Strong at following structured instructions, used here for JSON-formatted query parsing
- **Safety**: Built-in harmful content filtering

### **Authentication & Authorization**

| Technology | Version | Purpose & Highlights |
|------------|---------|---------------------|
| **Clerk** | 6.36.5 | User authentication with OAuth + email/password |
| **@clerk/nextjs** | 6.36.5 | Next.js middleware for route protection |

**Why Clerk?**
- **10+ OAuth Providers**: Google, GitHub, LinkedIn, Microsoft, etc.
- **Prebuilt UI**: Sign-in/up components (customizable)
- **Session Management**: Secure JWT handling
- **User Metadata**: Store custom fields (role, industries, etc.)

### **Developer Experience**

| Technology | Version | Purpose |
|------------|---------|---------|
| **ESLint** | 9.x | Code linting with Next.js recommended rules |
| **PostCSS** | 8.5.6 | CSS transformations (autoprefixer, nesting) |
| **Autoprefixer** | 10.4.23 | Automatic vendor prefixing for cross-browser support |
| **Babel React Compiler** | 1.0.0 | Auto-optimizes React renders (memoization) |

---

## 🎨 Database Schema

### **6 Collections** with **11 Compound Indexes** + **3 Search Indexes**

```typescript
users (10 fields + 2 indexes)
├── clerkUserId: string (indexed, unique)
├── email: string (indexed, unique)
├── name?: string
├── avatar?: string
├── headline?: string
├── role?: "student" | "engineer" | "founder" | "recruiter"
├── goal?: string
├── industries?: string[]
├── createdAt: timestamp
└── updatedAt: timestamp

contacts (18 fields + 6 indexes [5 compound, 1 single-field] + 2 search indexes)
├── userId: Id<users> (indexed)
├── name: string (searchable, required)
├── firstName?: string
├── lastName?: string
├── emails: string[] (validated)
├── phones: string[] (formatted)
├── company?: string (searchable)
├── title?: string
├── location?: string
├── industry?: string (indexed)
├── tags: string[] (filterable)
├── relationshipScore: number (0-100, indexed)
├── lifecycleStage: "new" | "active" | "dormant" | "archived" (indexed)
├── lastInteractionAt?: timestamp (indexed)
├── aiNotesSummary?: string (intended for Claude-generated summaries; not populated by any runtime function yet)
├── autoType?: "recruiter" | "client" | "investor" | "friend" | "other" (only recruiter/client/investor/other are auto-assigned; "friend" isn't set by current logic)
├── createdAt: timestamp
└── updatedAt: timestamp

interactions (9 fields + 5 indexes [3 compound, 2 single-field])
├── contactId: Id<contacts> (indexed)
├── userId: Id<users> (indexed)
├── type: "email" | "call" | "meeting" | "dm" | "other"
├── channel?: string (e.g., "LinkedIn", "Zoom")
├── direction: "inbound" | "outbound"
├── content?: string (meeting notes, email subject)
├── timestamp: timestamp (indexed)
├── sentimentScore?: number (-1 to 1, schema field only, not computed yet)
└── createdAt: timestamp

followUps (7 fields + 3 indexes)
├── contactId: Id<contacts> (indexed)
├── userId: Id<users> (indexed)
├── reason: string (rule-generated or manually entered)
├── dueAt: timestamp (indexed)
├── status: "pending" | "done" | "skipped" (indexed)
├── createdAt: timestamp
└── updatedAt: timestamp

notes (6 fields + 2 indexes + 1 search index)
├── contactId: Id<contacts> (indexed)
├── userId: Id<users> (indexed)
├── rawText: string (searchable, markdown supported)
├── aiSummary?: string (intended for Claude TL;DR; not populated by any runtime function yet)
├── createdAt: timestamp
└── updatedAt: timestamp

dedupeCandidates (6 fields + 3 indexes)
├── userId: Id<users> (indexed)
├── primaryContactId: Id<contacts> (indexed)
├── duplicateContactId: Id<contacts> (indexed)
├── confidence: number (0-100, fuzzy match score)
├── status: "suggested" | "merged" | "ignored" (indexed)
└── createdAt: timestamp
```

**Index Notes:**
- **Lookups**: Convex serves reads through defined indexes (e.g., `by_user`, `by_user_score`) rather than full-table scans
- **Search**: Convex full-text search indexes power name, company, and notes search
- **Compound Indexes**: Composite indexes support common query patterns (user + score, user + lifecycle, user + industry, etc.)
- *(No independent latency benchmarking has been run against this schema — specific ms/complexity figures were removed rather than guessed.)*

---

## 🚀 Getting Started

### **Prerequisites**

Before you begin, ensure you have the following:

- ✅ **Node.js** 20+ and **npm** 10+ ([Download](https://nodejs.org/))
- ✅ **Convex account** ([Sign up at convex.dev](https://convex.dev))
- ✅ **Clerk account** ([Sign up at clerk.com](https://clerk.com))
- ✅ **Anthropic API key** ([Get from console.anthropic.com](https://console.anthropic.com))

### **Installation & Setup**

```bash
# 1. Clone the repository
git clone https://github.com/Brijesh03032001/SmartContactManager.git
cd smartcontactmanager

# 2. Install dependencies (takes ~2 minutes)
npm install

# 3. Set up environment variables
cp .env.example .env.local
```

**Edit `.env.local` with your credentials:**

```bash
# Clerk Authentication (from dashboard.clerk.com)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxx
CLERK_SECRET_KEY=sk_test_xxxxx

# Convex Backend (from dashboard.convex.dev)
CONVEX_DEPLOYMENT=prod:your-deployment-name
NEXT_PUBLIC_CONVEX_URL=https://your-deployment.convex.cloud

# Anthropic AI (from console.anthropic.com)
ANTHROPIC_API_KEY=sk-ant-xxxxx
```

```bash
# 4. Initialize Convex and push schema
npx convex dev
# This will:
#   ✓ Create a new Convex project (or link existing)
#   ✓ Push database schema (6 tables, 21 indexes + 3 search indexes)
#   ✓ Deploy functions (queries, mutations, actions)
#   ✓ Start realtime sync server
# Keep this terminal open!

# 5. Start Next.js dev server (in a NEW terminal)
npm run dev
```

**Open [http://localhost:3000](http://localhost:3000)** in your browser! 🎉

### **Quick Start Commands**

```bash
npm run dev          # Start development server (localhost:3000)
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint
npx convex dev       # Start Convex backend (separate terminal)
```

---

## 🎯 Key Features Demo

### **Relationship Scoring — Conceptual Sketch**

*The scoring function is invoked via `ctx.scheduler.runAfter(0, "scoring:computeRelationshipScore", ...)` from `convex/interactions.ts` and `convex/deduplication.ts`, but the `scoring.ts` module itself isn't part of this repo checkout, so the exact formula below is illustrative rather than a copy of the real implementation:*

```typescript
// Illustrative only — not the verified source of convex/scoring.ts
const score = (
  frequencyScore * frequencyMultiplier *
  recencyDecay * channelWeight *
  directionMultiplier * twoWayBonus
) / maxPossibleScore * 100;
```

### **AI Assistant Example Queries**

*These illustrate the intended interaction pattern. The live chat endpoint currently returns rule-based responses (see "AI Assistant" above), not real Claude output — labeled "Assistant" rather than "Claude" for that reason.*

```
User: "Show me my top 5 recruiters"
Assistant: "Here are your strongest recruiter connections:
1. Sarah Johnson at TechCorp (Score: 87/100)
2. Mike Davis at StartupX (Score: 82/100)
...

User: "Draft an email to Sarah about job opportunities"
Assistant: "Subject: Checking in - Open to new opportunities

Hi Sarah,

I hope you're doing well! It's been about 3 weeks since we 
last connected. I wanted to reach out because I'm currently 
exploring new opportunities in full-stack development...
```

---

## 🏆 Technical Highlights

### **Performance**
*No load-testing, Lighthouse, or benchmarking suite exists in this repo — the figures that previously appeared here (load time, sync latency, search speed, AI response time) weren't backed by any measurement, so they've been removed rather than restated as fact.*
- ⚡ **Realtime Sync**: Convex WebSocket subscriptions (no polling)
- ⚡ **Search**: Convex full-text search indexes on contact name, company, and notes
- ⚡ **AI Calls**: Email drafting and query parsing call the Claude API synchronously (non-streaming); the chat endpoint itself currently returns instant rule-based responses (see "AI Assistant" above)

### **Code Quality**
- ✅ **TypeScript throughout**: a small number of `any` types remain (mainly around a temporary hardcoded user ID used for testing, and a couple of error/catch blocks) — not a "zero `any`" codebase
- ✅ **Fully Responsive**: Mobile, tablet, desktop layouts
- ⚠️ **Accessibility**: built on Radix UI primitives (keyboard nav, focus management); no ARIA labels, screen-reader testing, or formal WCAG audit have been done yet
- 🔜 **Tested**: Unit + integration tests (coming soon)

### **Scalability**
- 📈 **Database**: Convex scales to millions of records
- 📈 **Serverless**: Next.js + Convex (0 to infinity)
- 📈 **Edge Network**: Global CDN for static assets
- 📈 **Cost**: Pay-per-use (free tier: 1GB data, 1M function calls)

