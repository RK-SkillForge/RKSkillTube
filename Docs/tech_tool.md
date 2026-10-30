# 🛠️ RKSkillTube — Comprehensive Tech & Tool Guide (`tech_tool.md`)

> **Document Purpose:** This document provides a complete, granular breakdown of **every single technology, library, runtime, and tool** used across the **RKSkillTube** platform. It explains **what each tool does**, **why it was chosen over alternatives**, and **how it fits into the overall architecture**.

---

## 🧭 Architectural Philosophy

RKSkillTube is a hybrid video streaming and e-learning platform designed to handle **10 million users, 2,000 peak requests per second (RPS), and 100–120 TB of video storage**.

To achieve this scale while keeping local development smooth and beginner-friendly, the architecture follows four core principles:
1. **Decoupled Frontend & Backend:** Next.js handles user interaction and SEO; FastAPI handles data validation, business logic, and high-speed API delivery.
2. **Asynchronous Heavy Processing:** CPU-heavy tasks (video transcoding, thumbnail generation, AI summaries) are offloaded to background workers (Celery + Redis) so the API remains instantaneous.
3. **Strict Type Safety:** TypeScript on the frontend and Pydantic v2 on the backend ensure contract consistency and eliminate runtime errors.
4. **Zero-Egress Cost Efficiency:** Cloudflare R2 and PostgreSQL with `pgvector` provide high-performance media delivery and AI search without expensive external managed services.

---

## 📋 Technology Stack Overview

| Category | Technology / Tool | Primary Purpose in RKSkillTube |
| :--- | :--- | :--- |
| **Frontend Framework** | Next.js 14+ (App Router) | Server-side rendering (SSR), SEO, client routing, API proxy |
| **Frontend Language** | TypeScript | Compile-time type safety, developer autocomplete, bug prevention |
| **UI Library** | React 18+ | Declarative component-based user interface |
| **Styling** | Tailwind CSS | Utility-first, zero-bloat styling with dark mode support |
| **UI Primitives** | Radix UI / shadcn/ui | Headless, accessible (a11y) interactive components |
| **Icons** | Lucide React | Lightweight, tree-shakeable SVG icon suite |
| **Video Playback** | HLS.js & Video.js | Adaptive Bitrate (ABR) video streaming via MediaSource Extensions |
| **Captions & Scrubbing** | WebVTT & Sprite Sheets | Closed captions and timeline hover thumbnail previews |
| **Client State** | Zustand | Ultra-lightweight global state (auth, player, theater mode) |
| **API Client** | Axios & TanStack Query | HTTP requests, JWT token interceptors, and server-cache |
| **Backend Framework** | FastAPI (ASGI) | Ultra-high performance asynchronous REST APIs and OpenAPI docs |
| **Backend Language** | Python 3.11 / 3.12 | Core backend runtime, AI model integration, and media scripts |
| **ASGI Web Server** | Uvicorn | Lightning-fast asynchronous server implementation for Python |
| **Data Validation** | Pydantic v2 | Strict schema validation and serialization powered by Rust |
| **Configuration** | Pydantic-Settings | 12-factor typed environment variable management |
| **Primary Database** | PostgreSQL 16 | Relational persistence with ACID transactions and JSONB support |
| **Vector Search** | pgvector | Storing and querying vector embeddings for AI recommendations |
| **ORM** | SQLAlchemy 2.0 (Async) | Enterprise Python Object Relational Mapper with asyncpg pool |
| **Database Migrations** | Alembic | Version-controlled database schema changes and rollbacks |
| **Cache & Broker** | Redis 7 | In-memory caching, rate limiting, and Celery task broker |
| **Task Queue** | Celery | Distributed background processing for video and media jobs |
| **Task Monitoring** | Flower | Real-time visual monitoring dashboard for Celery workers |
| **Media Processing** | FFmpeg | Multi-bitrate HLS transcoding, thumbnail and sprite extraction |
| **Object Storage** | Google Cloud Storage (GCS) / Cloudflare R2 | Scalable media bucket for raw uploads, HLS playlists, and sprites |
| **Content Delivery** | Google Cloud CDN / Cloudflare | Global low-latency edge caching with HTTP/3 and DDoS protection |
| **Cloud Infrastructure (IaC)** | Google Cloud Platform (GCP) & Terraform | Cloud SQL (PostgreSQL 16), Cloud Storage, and automated IaC |
| **Authentication** | NextAuth / JWT / Bcrypt | Secure Google OAuth 2.0 and salted password hashing |
| **API Rate Limiting** | SlowAPI | Sliding-window DDoS and brute-force protection |
| **AI & LLM** | Google Gemini API & LangChain | AI course roadmap generation, chatbot, and semantic search |
| **Containerization** | Docker & Docker Compose | Containerized local databases and multi-stage production builds |
| **Version Control** | Git & GitHub Actions | Source control, CI/CD automated linting, testing, and deployment |

---

## 1. Frontend Technologies & Tools

### 1.1 Next.js 14+ (App Router)
* **What it is:** A production-grade React framework created by Vercel that offers Server Components, nested layouts, and edge rendering.
* **Why we use it:**
  * **SEO & Fast First Paint:** Public pages (Course catalog, creator channels, video landing pages) are server-rendered with instant HTML for search engines and social previews.
  * **App Router Architecture:** Enables granular caching, streaming server rendering with React Suspense, and nested layouts that prevent re-rendering identical headers/sidebars.
  * **Built-in Optimizations:** Automatically optimizes images (`next/image`), scripts, and fonts (`next/font`) without extra build tooling.
* **Alternative Considered:** *Vite + React SPA* (Rejected because SPAs lack built-in SSR, resulting in poor SEO and slower first-contentful paint on low-bandwidth mobile connections).

### 1.2 TypeScript
* **What it is:** A strongly typed superset of JavaScript that compiles down to plain JavaScript.
* **Why we use it:**
  * **Eliminates `undefined` Errors:** Catches missing object keys, typoed API properties, and null-pointer bugs during development.
  * **Type Sharing with Backend:** Next.js TypeScript interfaces (`interface Video { ... }`) match FastAPI Pydantic schemas, creating an end-to-end type-safe contract across the network.
  * **Superior Developer Experience:** Immediate autocomplete in VS Code/IDE for props, functions, and imported modules.
* **Alternative Considered:** *Plain JavaScript* (Rejected because large video and payment codebases become error-prone and hard to refactor without types).

### 1.3 Tailwind CSS
* **What it is:** A utility-first CSS framework packed with classes like `flex`, `pt-4`, `text-center`, and `rotate-90`.
* **Why we use it:**
  * **Zero CSS Bloat:** Purges unused styles in production, keeping the final CSS bundle under 15 KB.
  * **Speed of Prototyping:** Style components directly inside `.tsx` files without switching between stylesheets.
  * **Native Dark Mode:** Seamless dark mode toggling using `dark:` variants, essential for video viewing platforms.
* **Alternative Considered:** *Styled-Components / CSS Modules* (Rejected due to larger runtime JavaScript overhead and slower build times).

### 1.4 Radix UI & shadcn/ui
* **What it is:** Headless, unstyled, accessible UI primitives (dialogs, dropdowns, tooltips, sliders, tabs).
* **Why we use it:**
  * **Full Accessibility (a11y):** Provides ARIA roles, focus traps, and keyboard navigation (e.g. closing modals with `Escape` and navigating menus with arrow keys) out of the box.
  * **Unstyled Freedom:** Allows full styling customization using Tailwind CSS without fighting default library CSS overrides.

### 1.5 Lucide React
* **What it is:** An open-source, beautifully balanced icon suite with over 1,000+ vector icons.
* **Why we use it:** Tree-shakeable SVG icons that load instantly with zero layout shift (play, pause, volume, captions, settings, user avatars).

---

## 2. Video Streaming & Playback Subsystem

### 2.1 HLS.js
* **What it is:** A JavaScript library that implements an HTTP Live Streaming (HLS) client using HTML5 Video and Media Source Extensions (MSE).
* **Why we use it:**
  * **Adaptive Bitrate Streaming (ABR):** Automatically detects user network bandwidth. If a user is on 4G/WiFi, it seamlessly streams 1080p; if bandwidth drops, it switches dynamically to 480p or 360p without buffering or interrupting playback.
  * **Universal Browser Support:** Works on Google Chrome, Firefox, Edge, Safari, and mobile browsers without requiring Flash or proprietary plugins.
* **Alternative Considered:** *Raw MP4 `<video src="video.mp4">`* (Rejected because raw MP4 forces downloading the entire file upfront, wastes server bandwidth, and cannot adapt to fluctuating user internet speeds).

### 2.2 WebVTT & Storyboard Sprite Sheets
* **What it is:** The W3C standard format for timed text tracks (captions, subtitles, chapters, and thumbnail coordinate maps).
* **Why we use it:**
  * **Scrubber Hover Preview:** When a user hovers their cursor over the timeline progress bar, a mini thumbnail shows the exact frame at that second (generated as a single tiled sprite sheet via FFmpeg to minimize network requests).
  * **Accessibility & Global Reach:** Supports closed captions in multiple languages.

---

## 3. Client State & Data Fetching

### 3.1 Zustand
* **What it is:** A small, fast, and scalable state-management solution using simplified flux principles.
* **Why we use it:**
  * **Minimal Boilerplate:** Does not require wrapping your entire application in nested `<Provider>` trees like Redux.
  * **High Performance:** Components only re-render when the exact state slice they subscribe to changes (e.g. only the player updates when playback time advances, preventing the whole page from re-rendering).
  * **Size:** Under 1 KB minified.
* **Alternative Considered:** *Redux Toolkit* (Rejected because Redux introduces excessive boilerplate for UI-level player and auth state).

### 3.2 Axios & TanStack Query (React Query)
* **What it is:** Axios handles low-level HTTP requests and interceptors; TanStack Query manages asynchronous server state.
* **Why we use them:**
  * **JWT Auto-Refresh:** Axios interceptors catch `401 Unauthorized` responses and automatically exchange expired access tokens for fresh ones using refresh tokens without logging the user out.
  * **Client Caching & Deduplication:** TanStack Query prevents duplicate network requests and instantly displays cached course/video feeds while updating in the background.

---

## 4. Backend Technologies & Tools

### 4.1 Python 3.11 / 3.12
* **What it is:** The latest stable releases of Python featuring the Faster CPython performance upgrades (10–60% speed increase over Python 3.10).
* **Why we use it:**
  * Unmatched ecosystem for **AI/LLM pipelines (Gemini, LangChain)** and **video automation (FFmpeg, OpenCV)**.
  * Native asynchronous event loops (`asyncio`) enabling high-concurrency non-blocking I/O.

### 4.2 FastAPI
* **What it is:** A modern, high-performance web framework for building APIs with Python based on standard Python type hints.
* **Why we use it:**
  * **Speed:** One of the fastest Python frameworks available, matching NodeJS and Go benchmarks thanks to Starlette and Pydantic.
  * **Automatic Documentation:** Automatically generates interactive Swagger UI (`/docs`) and ReDoc (`/redoc`) directly from Python type hints with zero manual documentation effort.
  * **Dependency Injection:** Built-in DI system (`Depends()`) makes managing database sessions, authentication guards, and role permissions clean and testable.
* **Alternative Considered:** *Django REST Framework (DRF)* (Rejected because Django is synchronous by default, heavier, and less suited for asynchronous streaming and microservice decomposition).

### 4.3 Pydantic v2
* **What it is:** The most widely used data validation library for Python, rewritten in Rust in version 2.
* **Why we use it:**
  * **5x to 15x Performance Boost:** Validates and serializes incoming JSON payloads at Rust speed.
  * **Strict Data Contracts:** Rejects invalid client inputs (e.g. invalid email formats, missing video IDs, negative prices) before code reaches database queries.

### 4.4 Uvicorn
* **What it is:** An ASGI (Asynchronous Server Gateway Interface) web server implementation for Python.
* **Why we use it:** Powers the FastAPI runtime using `uvloop` (an ultra-fast Drop-in replacement for the standard asyncio event loop written in Cython/libuv).

---

## 5. Database & Persistence Layer

### 5.1 PostgreSQL 16
* **What it is:** The world's most advanced open-source object-relational database system.
* **Why we use it:**
  * **ACID Transactions:** Crucial for student payments, course enrollments, and user balances where partial commits or data corruption cannot be tolerated.
  * **JSONB Capabilities:** Allows storing structured metadata (e.g. course lesson trees, quiz questions) without complex multi-table joins.
  * **Concurrency:** Multi-Version Concurrency Control (MVCC) handles thousands of simultaneous readers and writers.

### 5.2 pgvector Extension
* **What it is:** An open-source vector similarity search extension for PostgreSQL.
* **Why we use it:**
  * **Unified Infrastructure:** Stores vector embeddings (768-dimensional or 1536-dimensional float arrays from Gemini/OpenAI) directly inside PostgreSQL alongside user and video rows.
  * **Eliminates Extra Databases:** You do not need to pay for or maintain an external vector database like Pinecone, Milvus, or Qdrant.
  * **Indexing:** Supports HNSW (Hierarchical Navigable Small World) and IVFFlat indexes for millisecond similarity matching across millions of courses.

### 5.3 SQLAlchemy 2.0 (Async) & Alembic
* **What it is:** SQLAlchemy is the Python SQL toolkit and ORM; Alembic is its database migration tool.
* **Why we use them:**
  * **Async I/O:** Uses `asyncpg` to execute queries without blocking the Python event loop.
  * **Migration History:** Alembic tracks every database schema change in version-controlled Python files, allowing safe zero-downtime database upgrades and rollbacks.

---

## 6. Caching, Tasks & Media Processing

### 6.1 Redis 7
* **What it is:** An open-source, in-memory data structure store used as a database, cache, message broker, and streaming engine.
* **Why we use it:**
  * **View Counter Buffering:** Writing video view increments directly to PostgreSQL under high load would lock database rows. Redis buffers view increments (`INCR video:123:views`) in memory and flushes them to Postgres periodically.
  * **Session & Cache Storage:** Caches user profiles, trending video lists, and course outlines with short TTLs (Time-to-Live).
  * **Message Broker:** Acts as the high-throughput transport broker between FastAPI and Celery workers.

### 6.2 Celery & Flower
* **What it is:** Celery is a distributed asynchronous task queue; Flower is its web-based monitoring tool.
* **Why we use them:**
  * **Decoupled Heavy Computation:** When a user uploads a 2 GB video, FastAPI returns an immediate response (`202 Accepted`). Celery processes the transcoding in the background without tying up API threads.
  * **Retry Logic & Error Handling:** Automatically retries failed encoding tasks with exponential backoff.
  * **Flower Dashboard:** Provides real-time visibility into active workers, task success rates, and queue bottlenecks.

### 6.3 FFmpeg
* **What it is:** The industry standard cross-platform command-line tool for transcoding, packaging, and filtering audio/video.
* **Why we use it:**
  * **Adaptive HLS Encoding:** Converts raw creator uploads into multiple resolution tiers:
    * `1080p` (Full HD - High bitrate)
    * `720p` (HD - Medium bitrate)
    * `480p` (SD - Standard mobile bitrate)
    * `360p` (Low - Data saver bitrate)
  * **Segment Generation:** Slices videos into 4-second `.ts` chunks accompanied by master and variant `.m3u8` playlist files.
  * **Thumbnail & Sprite Generation:** Extracts high-quality thumbnail posters and creates sprite grids for scrubber hover previews.

---

## 7. Cloud Infrastructure, Storage & CDN Delivery

### 7.1 Google Cloud Platform (GCP) — Primary Cloud Provider
* **What it is:** Enterprise cloud suite hosting RKSkillTube's production database, storage, and edge caching.
* **Why we use it:**
  * **Unified Ecosystem:** Provides seamless IAM authentication, Cloud SQL for managed PostgreSQL with `pgvector`, and tight integration with Google Gemini AI APIs.
  * **Automated Infrastructure:** Managed with Terraform using the official Google provider (`hashicorp/google`).

### 7.2 Google Cloud Storage (GCS)
* **What it is:** Scalable, highly durable object storage on Google Cloud.
* **Why we use it:**
  * **V4 Signed URLs:** Generates secure, short-lived signed PUT URLs from FastAPI using the `google-cloud-storage` Python SDK so client browsers upload directly to GCS without passing video files through backend servers.
  * **Origin for CDN:** Seamlessly integrates as an origin backend bucket for Google Cloud CDN.

### 7.3 Google Cloud CDN & Global Load Balancing
* **What it is:** Anycast-routed edge caching network connected directly to Google's private global fiber backbone.
* **Why we use it:**
  * **Sub-30ms Video Delivery:** Immutable HLS `.m3u8` playlists, `.ts` chunks, and WebVTT storyboard images are cached at Google Edge Points of Presence (PoPs) worldwide.
  * **Cloud Armor Protection:** Built-in DDoS mitigation and web application firewall (WAF) policies.

### 7.4 Multi-Cloud Alternatives (Cloudflare R2 / AWS S3)
* **What it is:** S3-compatible cloud object storage alternative.
* **Why we support it:** Provides optional zero-egress fee storage configuration for flexible multi-cloud deployments.

---

## 8. Authentication & Security Tools

### 8.1 NextAuth.js (Auth.js) & JWT
* **What it is:** Complete authentication suite for Next.js with JSON Web Token session management.
* **Why we use it:**
  * **Google OAuth 2.0:** Secure one-click social authentication for students and creators.
  * **Stateless Tokens:** Decoupled JWT access tokens signed with HMAC-SHA256 allow FastAPI to authenticate requests without performing database lookups on every single API hit.

### 8.2 Passlib & Bcrypt
* **What it is:** Password hashing library implementing the adaptive Blowfish-based bcrypt algorithm.
* **Why we use it:** Uses automatically salted, computationally expensive hashes to protect student and creator credentials against rainbow table and GPU dictionary attacks.

### 8.3 SlowAPI
* **What it is:** A Python rate-limiting library for Starlette and FastAPI based on Redis.
* **Why we use it:** Implements sliding-window rate limiting (e.g. maximum 5 login attempts per minute, 60 search queries per minute) to prevent brute-force attacks and API abuse.

---

## 9. AI & Semantic Search Tools

### 9.1 Google Gemini API
* **What it is:** Google's state-of-the-art multimodal large language model family.
* **Why we use it:**
  * **Massive Context Window & Cost-Efficiency:** Analyzes entire course transcripts and curriculum outlines in a single prompt.
  * **RKSkillTube AI Assistant:** Generates personalized learning paths, answers student questions about course content, and extracts chapter summaries from video transcripts.

### 9.2 LangChain
* **What it is:** A framework for developing applications powered by language models.
* **Why we use it:** Standardizes prompt templating, output parsing, and Retrieval-Augmented Generation (RAG) pipelines connecting student questions to course video embeddings stored in `pgvector`.

---

## 10. DevOps, Containerization & CI/CD

### 10.1 Docker & Docker Compose
* **What it is:** Platform for building, running, and orchestrating containerized applications.
* **Why we use it:**
  * **One-Click Local Environment:** Developers can spin up PostgreSQL 16 (with `pgvector`), Redis 7, and Flower with a single `docker compose up -d` command without installing database engines directly on Windows/macOS.
  * **Production Parity:** Eliminates "works on my machine" bugs by guaranteeing identical software versions and runtime dependencies in development, staging, and production.

### 10.2 GitHub Actions
* **What it is:** Automated CI/CD workflow platform integrated directly into GitHub.
* **Why we use it:**
  * Automatically runs ESLint, TypeScript compiler checks, and Next.js builds on the frontend.
  * Automatically runs Pytest, Flake8 linting, and Docker container builds on the backend on every Pull Request before code can be merged.

---

## 🎯 Summary Matrix: Why This Stack Fits RKSkillTube

```
┌────────────────────────────────────────────────────────────────────────┐
│                          RKSKILLTUBE FULL STACK                        │
├───────────────────────┬──────────────────────┬─────────────────────────┤
│ LAYER                 │ TECHNOLOGY           │ CORE ADVANTAGE          │
├───────────────────────┼──────────────────────┼─────────────────────────┤
│ User Interface        │ Next.js 14 + TS      │ Fast SSR, SEO, Type-Safe│
│ Styling & a11y        │ Tailwind + Radix     │ Zero-bloat, Accessible  │
│ Video Streaming       │ HLS.js + WebVTT      │ Adaptive Bandwidth (ABR)│
│ State Management      │ Zustand + Axios      │ Ultra-light, JWT Interc.│
│ API Engine            │ FastAPI + Uvicorn    │ Async Speed, Auto Docs  │
│ Data Validation       │ Pydantic v2 (Rust)   │ 10x Speed, Strict Types │
│ Relational Data       │ PostgreSQL 16        │ ACID, JSONB, Stability │
│ Vector Embeddings     │ pgvector             │ Native AI Search in DB  │
│ Background Processing │ Celery + FFmpeg      │ Non-blocking Transcoding│
│ Caching & Broker      │ Redis 7              │ Sub-ms Counters & Queues│
│ Cloud Platform & IaC  │ Google Cloud (Terra) │ Cloud SQL, GCS, CloudCDN│
│ Cloud Storage         │ GCS / Cloudflare R2  │ Signed URLs, Scalability│
│ AI Intelligence       │ Google Gemini API    │ Roadmaps, Chatbot, RAG  │
│ Orchestration         │ Docker Compose       │ One-Command Dev Boot    │
└───────────────────────┴──────────────────────┴─────────────────────────┘
```

---

> [!TIP]
> * For the step-by-step local installation guide, see [`Docs/INSTALLATION_GUIDE.md`](./INSTALLATION_GUIDE.md).
> * For external documentation links, interactive courses, and blogs, see [`Docs/resource.md`](./resource.md).
> * For the codebase layout and directory structure, see [`Docs/folder.md`](./folder.md).
