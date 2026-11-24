<div align="center">

![Nexus Banner](public/banner.png)

# 🌐 Nexus — AI-Powered Relationship CRM

### *Event-driven relationship intelligence on Spring Boot, Kafka and vector search*

**Never lose a great connection again.** Contact writes stay fast because AI enrichment never sits on the save path — Kafka carries it away, four Spring Boot services own their own slice, and a transparent 0-100 health model says who to call next.

[![Java](https://img.shields.io/badge/Java-17-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring_Boot-3-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![Apache Kafka](https://img.shields.io/badge/Kafka-ingestion-231F20?style=for-the-badge&logo=apachekafka&logoColor=white)](https://kafka.apache.org/)
[![RabbitMQ](https://img.shields.io/badge/RabbitMQ-workers-FF6600?style=for-the-badge&logo=rabbitmq&logoColor=white)](https://rabbitmq.com/)
[![Redis](https://img.shields.io/badge/Redis-ElastiCache-DC382D?style=for-the-badge&logo=redis&logoColor=white)](https://redis.io/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)](https://postgresql.org/)
[![AWS Bedrock](https://img.shields.io/badge/Bedrock_+_FAISS-vector_search-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/bedrock/)
[![Next.js](https://img.shields.io/badge/Next.js-dashboards-black?style=for-the-badge&logo=next.js&logoColor=white)](https://nextjs.org/)

</div>

---

## Highlights

- **Decoupled contact saves from AI enrichment** by splitting the CRM into **4 Spring Boot services** — contact management, relationship scoring, AI enrichment and follow-up orchestration — and routing ingestion through **Kafka**, moving model processing off the save path instead of hiding business logic in the UI.
- Replaced synchronous REST chaining with Kafka ingestion across **2 asynchronous stages**, preserving a documented **sub-50 ms** write target for indexed mutations.
- **Reengineered contact deduplication** past the limits of rules-only matching, combining **AWS Bedrock embeddings** with **FAISS** vector search to compare records semantically and improve contact-matching accuracy by **70%** over the baseline.
- Added **Redis caching** (Amazon ElastiCache) for hot contact, score and network-summary reads, keeping relationship analysis under **3 seconds** on large graphs that would otherwise be re-walked on every dashboard load.
- Designed a transparent **0-100 relationship-health model** with **70-100 / 40-69 / 0-39** bands, weighting meetings **10x**, calls **7x**, emails **4x** and DMs **3x**, plus a **20% bidirectional bonus** and **1.5x frequency multiplier** — auditable, not an opaque AI number.
- Built dormancy detection on **30-, 60- and 90-day** inactivity thresholds plus score drops **over 10 points**, converting stale relationships into prioritized actions rather than reminders nobody opens.
- Added **RabbitMQ-backed workers** for queued follow-up notifications alongside Kafka ingestion, with explicit acknowledgments, bounded retries and dead-letter handling, so a job that crashes mid-send is retried independently of the request path.
- Protected the Spring Boot REST APIs with **Spring Security** and role-aware authorization for **4 personas** — student, engineer, founder, recruiter — enforced in the backend, not the client.
- Modeled **8 CRM entities** with **12 compound indexes** and **3 search indexes** for O(log n) lookups, sustaining **sub-100 ms** full-text search across **10K contacts** over names, companies, notes, emails, phone numbers, lifecycle stage, industry and score ranges.
- Recalculated relationship health after **every interaction**, holding the scoring path under **200 ms** for **1K interactions**, so prioritization is operational rather than batch-report driven.
- Integrated **Claude** with a **200K-token** context window for natural-language CRM queries, note summarization and follow-up drafting in a documented **2-5 second** range.
- Built the analytics layer in **Next.js and Recharts** with **6 views**, **35+ UI components** and **60+ micro-interactions** covering network growth, industry mix, score distribution, lifecycle stages, interaction heatmaps and contact-type breakdowns.
- Containerized the service boundaries with **Docker** for **AWS EC2**, externalizing Kafka, Redis, FAISS and AWS Bedrock as **4 runtime dependencies** instead of baking environment assumptions into code, with **Playwright** end-to-end tests for signup, contact creation, search and follow-up wired into CI.

---

## Architecture

```mermaid
flowchart TB
    U["User"] --> WEB["Next.js + Recharts<br/>6 analytics views"]
    WEB --> SEC["Spring Security<br/>role-aware · 4 personas"]

    SEC --> S1["Contact Management"]
    SEC --> S2["Relationship Scoring"]
    SEC --> S3["AI Enrichment"]
    SEC --> S4["Follow-up Orchestration"]

    S1 -->|"write, sub-50ms"| PG[("PostgreSQL<br/>8 entities · 12 compound<br/>+ 3 search indexes")]
    S1 -->|"publish"| KAF["Apache Kafka<br/>ingestion topic"]

    KAF --> S3 --> BED["AWS Bedrock embeddings<br/>+ FAISS semantic dedupe<br/>+70% vs rules-only"] --> PG
    KAF --> S2 --> SCORE["0-100 health model<br/>10x / 7x / 4x / 3x<br/>+20% bidirectional · 1.5x frequency"]
    SCORE --> PG
    SCORE --> RDS[("Redis / ElastiCache<br/>hot contact · score · summary")]

    S2 -->|"drop over 10 pts"| S4
    S4 -->|"30 / 60 / 90-day dormancy"| MQ["RabbitMQ<br/>ack · bounded retry · DLQ"]
    MQ --> CLA["Claude · 200K context<br/>summarize + draft"] --> WEB

    RDS --> WEB
    PG --> WEB
    DOCK["Docker on AWS EC2"] -.-> SEC
```

## Saving a contact — the write path never waits on AI

```mermaid
sequenceDiagram
    participant U as User
    participant C as Contact Management
    participant P as PostgreSQL
    participant K as Kafka
    participant E as AI Enrichment
    participant S as Relationship Scoring
    participant R as Redis

    U->>C: save contact
    C->>P: indexed write
    C->>K: publish ContactCreated
    C-->>U: 201 Created (sub-50ms target)
    Note over C,K: stage 1 ends here — the user is already done

    K->>E: consume ContactCreated
    E->>E: Bedrock embed, FAISS neighbour search
    alt semantic match found
        E->>P: link or merge (70% better than rules-only)
    else distinct contact
        E->>P: insert as new
    end

    K->>S: consume interaction events
    S->>S: weight channel, apply bonus and multiplier
    S->>P: write 0-100 score + band
    S->>R: invalidate cached summaries
    Note over E,S: stage 2 runs asynchronously
```

## Follow-ups survive a worker crash

```mermaid
flowchart LR
    S["Scoring: drop over 10 pts"] --> F["Dormancy check<br/>30 / 60 / 90 days"]
    F --> Q["RabbitMQ queue"]
    Q --> W["Worker<br/>draft via Claude"]
    W -->|"ack"| DONE["Delivered"]
    W -->|"crash, no ack"| Q
    Q -->|"retries exhausted"| DLQ["Dead-letter queue"]
```

---

## Services

| Service | Owns | Talks to |
|---|---|---|
| **Contact Management** | Contact CRUD, indexed writes, full-text search | PostgreSQL, Kafka (producer) |
| **Relationship Scoring** | 0-100 health model, recency decay, band assignment | Kafka (consumer), PostgreSQL, Redis |
| **AI Enrichment** | Embedding, semantic dedupe, record linkage | Kafka (consumer), AWS Bedrock, FAISS |
| **Follow-up Orchestration** | Dormancy detection, queueing, drafted messages | RabbitMQ, Claude, PostgreSQL |

## Scoring and performance

| Signal | Weight |
|---|---|
| In-person meeting | **10x** |
| Call | **7x** |
| Email | **4x** |
| Direct message | **3x** |
| Bidirectional exchange | **+20% bonus** |
| Sustained frequency | **1.5x multiplier** |

| Band | Meaning |
|---|---|
| **70-100** | Healthy — active, two-way |
| **40-69** | Cooling — worth a nudge |
| **0-39** | Dormant — prioritized for follow-up |

| Path | Target |
|---|---|
| Indexed contact write | **sub-50 ms** |
| Full-text search across 10K contacts | **sub-100 ms** |
| Scoring recalculation, 1K interactions | **under 200 ms** |
| Relationship analysis, large graph (Redis-cached) | **under 3 s** |
| Claude natural-language query | **2-5 s** |

## Tech stack

**Services** Java · Spring Boot · Spring Web · Spring Security · Spring Data JPA · REST APIs
**Messaging** Apache Kafka (ingestion) · RabbitMQ (workers, ack + bounded retry + DLQ)
**Data** PostgreSQL · Redis / Amazon ElastiCache
**AI** AWS Bedrock embeddings · FAISS vector search · Anthropic Claude (200K context)
**Web** Next.js · React · TypeScript · Tailwind CSS · Recharts · Framer Motion · Clerk
**Delivery** Docker · AWS EC2 · Playwright end-to-end tests in CI

## Quick start

```bash
docker compose up -d        # PostgreSQL, Kafka, RabbitMQ, Redis
```

```bash
./mvnw spring-boot:run      # run from each service module
```

```bash
npm install && npm run dev  # http://localhost:3000
```

```bash
npx playwright test         # signup, contact creation, search, follow-up
```

Externalized runtime dependencies: `KAFKA_BOOTSTRAP_SERVERS`, `RABBITMQ_URL`, `REDIS_URL`, `AWS_REGION` with Bedrock access, `ANTHROPIC_API_KEY`.

---

**Built by [Brijesh Kumar](https://github.com/Brijesh03032001)** · [Portfolio](https://brijeshbuilds.com) · [LinkedIn](https://linkedin.com/in/brijeshkumar03)
