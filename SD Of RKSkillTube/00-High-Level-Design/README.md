# RKSkillTube — High-Level System Design & Tech Stack Specification

> **Platform Overview:** Next-generation online video and course learning platform designed for Tier-2 & Tier-3 engineering students, featuring creator channels, monetization, free/paid courses, community engagement, and AI-powered recommendations & search.

---

## 1. Scale & Non-Functional Specifications
* **Target User Base:** 10,000,000 Total Registered Users
* **Daily Active Users (DAU):** 1,000,000 DAU
* **Peak Traffic:** 2,000 Requests/sec (RPS)
* **Read / Write Ratio:** 100 : 1
* **Video Media Footprint:** 100 – 120 TB storage (50 TB daily bandwidth)
* **Latency SLO:** p95 API response < 200ms; Video startup time < 1.5s
* **Availability:** 99.9% uptime with zero single point of failure (SPOF)
* **Consistency:** Strict ACID for payments & subscriptions; Eventual consistency for feed & views

---

## 2. Recommended Technology Stack (End-to-End)

```
[ Client Layer ]
Next.js 14+ (App Router with TypeScript) | Tailwind CSS | HLS.js / Video.js
                  │
                  ▼ HTTPS
[ Edge / CDN & Security Layer ]
Cloudflare CDN / AWS CloudFront (WAF, DDoS, Signed URLs, Edge Caching)
                  │
                  ▼
[ Application / Gateway Layer ]
FastAPI (Python 3.11+) + Uvicorn ASGI (Modular Monolith / Microservices-ready)
                  ├── AI & LLM Engine (Gemini / OpenAI / Anthropic + pgvector)
                  │
       ┌──────────┴──────────┬────────────────────────┐
       ▼                     ▼                        ▼
[ Primary Database ]    [ In-Memory Cache ]    [ Background Worker Queue ]
PostgreSQL 16           Redis 7.x              Celery + Redis Broker
• Relational ACID       • Hot Metadata Cache   • FFmpeg Transcoding (HLS)
• JSONB Curricula       • Rate Limiting        • AWS SES Bulk Email
• pgvector Embeddings   • Write Buffer         • Certificates & Invoicing
                             │
                             ▼
                 [ Object Media Storage ]
                 Cloudflare R2 / AWS S3
                 (Direct Presigned Multipart Uploads)
```

| Layer | Chosen Technology | Why This Tech is Best for RKSkillTube |
| :--- | :--- | :--- |
| **Frontend / Web Client** | **Next.js 14+ (App Router with TypeScript)** | Blazing fast Server Components (SSR), optimal SEO for public courses, dynamic HLS video player state, and type safety across network contracts. |
| **Backend & API Layer** | **Python FastAPI + Uvicorn** | Native async non-blocking concurrency (ASGI), strict Pydantic v2 schemas, automated OpenAPI/Swagger docs, and direct native execution of AI/ML pipelines without microservice overhead. |
| **Primary Relational DB** | **PostgreSQL 16 + pgvector** | Strict ACID guarantees for subscriptions, course purchases, and creator payouts. Supports `JSONB` for flexible course playlists and `pgvector` for unified semantic search without extra vector DB costs. |
| **Caching & Rate Limiting** | **Redis 7.x** | Sub-5ms response for hot video metadata; distributed sliding-window rate limiting; atomic write buffering for viral video views/likes. |
| **Task Queue & Workers** | **Celery + Redis / SQS** | Offloads resource-heavy video transcoding, automated thumbnail generation, and bulk subscriber emails from the API thread. |
| **Media Storage & CDN** | **Cloudflare R2 + CloudFront/CDN** | Zero-egress fee storage with R2; adaptive bitrate streaming via **HLS (.m3u8)**; signed URLs for protecting paid course videos. |
| **AI Chatbot & Search** | **Gemini / OpenAI API + pgvector** | Hybrid search (relational + semantic vector) with conversational AI fallback when direct video lessons are not available. |
| **DevOps & Cloud** | **Docker + GCP Cloud Run / AWS ECS** | Stateless serverless containers auto-scaling from 0 to N instances, easily absorbing 2,000 RPS traffic spikes. |

---

## 3. Detailed Architectural Justifications

### 3.1 Backend: Why FastAPI Outperforms Alternatives
1. **Unified AI Integration:** RKSkillTube features an interactive AI Chatbot (scientific and spiritual learning guidance) and an AI Search Engine. In Node.js or Go, this requires external microservices. FastAPI enables running Python AI SDKs (LangChain, OpenAI, Gemini) directly within the application.
2. **High Concurrency (ASGI):** Built on Starlette and uvloop, FastAPI handles thousands of concurrent long-lived connections with minimal memory footprint.
3. **Rust-Powered Pydantic v2:** End-to-end data validation and serialization at C/Rust speeds with guaranteed type safety.

### 3.2 Database Strategy: PostgreSQL vs MongoDB
* **The Reality:** While early prototyping in MongoDB offers schema flexibility, RKSkillTube is a commercial EdTech platform involving paid channels, recurring memberships, course purchases, and role-based access.
* **The Recommendation:** **PostgreSQL** is the primary source of truth.
  * Relational integrity prevents orphan enrollments and duplicate payment credits.
  * `JSONB` gives MongoDB-style schema flexibility for nested lesson trees.
  * `pgvector` eliminates having to operate a separate vector database cluster.
* *(Note: If continuing with the existing MongoDB setup from prototype, implement MongoDB replica set multi-document transactions for payments and isolate relational entities).*

### 3.3 Media Pipeline: Zero-Server-Choke Architecture
* Videos average 500 MB. Passing video streams through backend API servers would crash memory and saturate network bandwidth.
* **Solution:**
  1. Frontend requests a presigned PUT URL from FastAPI (`POST /api/v1/videos/upload-url`).
  2. Browser uploads directly to AWS S3 / Cloudflare R2 bucket.
  3. S3 upload completion triggers an event to Celery workers.
  4. Celery worker runs FFmpeg to transcode video into **HLS (360p, 480p, 720p, 1080p)** segments.
  5. Content is served globally through CDN edge nodes.

---

## 4. Documentation Artifacts
* Detailed Word Document: [`RKSkillTube - Recommended Tech Stack Analysis.docx`](./RKSkillTube%20-%20Recommended%20Tech%20Stack%20Analysis.docx)
* System Architecture Diagram: [`RKSkillTube - High Level Design.drawio`](./RKSkillTube%20-%20High%20Level%20Design.drawio)
