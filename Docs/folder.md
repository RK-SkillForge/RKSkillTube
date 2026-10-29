# 📁 RKSkillTube — High-Level & Low-Level Folder Structure Architecture

> **Purpose:** This document provides a production-grade folder structure for **RKSkillTube** designed through both **High-Level Design (HLD)** and **Low-Level Design (LLD)** lenses. It maps the system specifications directly to the **Next.js 14+** (App Router with TypeScript) frontend, **FastAPI** backend, **Celery** workers, and **PostgreSQL/Redis** persistence.
>
> 🛠️ *For the complete sequential step-by-step setup (Docker, FastAPI, Next.js TS, FFmpeg, and package lifecycle), see [`Docs/INSTALLATION_GUIDE.md`](./INSTALLATION_GUIDE.md).*
> 🌐 *For official documentation, interactive courses, and curated blogs across all technologies, see [`Docs/resource.md`](./resource.md).*

---

## 🧭 1. Bridging High-Level Design (HLD) and Low-Level Design (LLD)

To master software engineering and system design, this repository maps HLD architectural boundaries directly to LLD design patterns:

```
┌────────────────────────────────────────────────────────────────────────┐
│                   HIGH-LEVEL DESIGN (HLD) PERSPECTIVE                  │
│  • Bounded Contexts: Auth, Video Streaming, Courses, Payments, AI     │
│  • Scalability: 10M Users, 2,000 Peak RPS, 100-120 TB Storage         │
│  • Topology: Client (Next.js) ──► Edge/CDN ──► API ──► DB/Cache/Workers │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Maps directly into
┌───────────────────────────────────▼────────────────────────────────────┐
│                    LOW-LEVEL DESIGN (LLD) PERSPECTIVE                  │
│  • Clean Architecture: Controller ──► Service ──► Repository ──► Model │
│  • Design Patterns: Strategy (Video/Payment), Factory, Dependency Inj. │
│  • Type Contracts: Pydantic v2 DTOs (Backend) & TypeScript Interfaces  │
│  • Next.js App Router: Server Components, Client Islands, Route Groups │
│  • Video Viewer Subsystem: Adaptive Bitrate (ABR) HLS, VTT Scrubbing   │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 🗂️ 2. Complete Root Repository Layout

```
RKSkillTube/
├── .github/                        # CI/CD Workflows (GitHub Actions)
│   ├── workflows/
│   │   ├── backend-ci.yml          # Pytest, linting, Docker build
│   │   └── frontend-ci.yml         # Next.js build, ESLint, TypeScript check
│   ├── ISSUE_TEMPLATE/
│   └── pull_request_template.md
│
├── frontend/                       # Next.js 14+ (App Router with TypeScript) Frontend
├── backend/                        # Python FastAPI (ASGI) Core Backend
├── infrastructure/                 # Cloud & Deployment Configs (Terraform, Docker, Nginx)
│
├── Docs/                           # Central Project Documentation Hub
│   ├── README.md                   # Documentation index & navigation
│   ├── tech_tool.md                # Comprehensive breakdown of all tech used and why
│   ├── resource.md                 # Master technology learning & official resource hub
│   ├── INSTALLATION_GUIDE.md       # Complete sequential setup guide (Docker, FastAPI, Next.js TS)
│   └── folder.md                   # Architectural folder walkthrough (this document)
│
├── SD Of RKSkillTube/              # System Design Documentation
│   ├── 00-High-Level-Design/       # HLD Specs, DrawIO diagrams, Word documents
│   └── 01-Low-Level-Design/        # LLD Class diagrams, Sequence flows, DB schemas
│
├── Feature Based Guide/            # Step-by-step feature breakdown guides
├── docker-compose.yml              # Local orchestration (PostgreSQL, Redis, Flower)
├── docker-compose.prod.yml         # Production multi-container setup
├── README.md                       # Main repository overview
└── .gitignore                      # Ignored files (node_modules, venv, .env)
```

---

## ⚛️ 3. Frontend Architecture — `frontend/` (Next.js 14+ App Router & Video Viewer)

The frontend is structured using **Next.js 14+ with App Router and TypeScript**, cleanly separating Server Components (for instant SSR and SEO) from Client Components (interactive video viewer, player controls, chat widgets, and creator studio).

```
frontend/
├── public/                         # Static assets (favicons, logos, placeholders, test media)
├── src/
│   ├── app/                        # Next.js 14 App Router (Routing & Layouts)
│   │   ├── (auth)/                 # Route Group: Authentication (clean layout without sidebar)
│   │   │   ├── login/page.tsx      # Email/Password + Google OAuth login
│   │   │   └── register/page.tsx   # Sign-up page with interest selection
│   │   │
│   │   ├── (main)/                 # Route Group: Main Student & Learner Experience
│   │   │   ├── layout.tsx          # Main layout: Header, Collapsible Sidebar, Mini Player
│   │   │   ├── page.tsx            # Home page (Personalized recommendation feed)
│   │   │   ├── feed/
│   │   │   │   ├── subscriptions/  # Content from subscribed creators
│   │   │   │   └── trending/       # Top trending videos and courses
│   │   │   ├── watch/
│   │   │   │   └── [videoId]/      # Video Watch Page (HLS Video Player, comments, likes)
│   │   │   ├── courses/
│   │   │   │   ├── page.tsx        # Course catalog (search, filter by free/paid/level)
│   │   │   │   └── [courseId]/     # Course curriculum, lesson syllabus, enrollment
│   │   │   ├── channel/
│   │   │   │   └── [handle]/       # Creator channel page (home, videos, playlists, about)
│   │   │   └── search/page.tsx     # Full-text & semantic video search results
│   │   │
│   │   ├── (creator)/              # Route Group: Creator Studio (Role-guarded)
│   │   │   ├── layout.tsx          # Studio layout (analytics navigation, upload toolbar)
│   │   │   └── studio/
│   │   │       ├── dashboard/      # Views, subscribers, earnings metrics overview
│   │   │       ├── videos/         # Video manager (edit metadata, view visibility)
│   │   │       ├── upload/         # Direct multipart upload to R2/S3 with progress bar
│   │   │       ├── courses/        # Course creator (builder for chapters & lessons)
│   │   │       └── analytics/      # Deep retention graphs and traffic sources
│   │   │
│   │   ├── (admin)/                # Route Group: Admin Operations (Admin role only)
│   │   │   └── admin/
│   │   │       ├── dashboard/      # Platform health, active users, RPS monitor
│   │   │       ├── moderation/     # Flagged video review & policy enforcement
│   │   │       └── payments/       # Transaction ledger & creator payouts audit
│   │   │
│   │   ├── api/                    # Route Handlers (BFF - Backend for Frontend proxy)
│   │   │   └── auth/[...nextauth]/ # NextAuth or OAuth token exchange handler
│   │   │
│   │   ├── globals.css             # Tailwind CSS & theme definitions
│   │   ├── layout.tsx              # Root HTML layout & font definitions
│   │   ├── not-found.tsx           # Custom 404 page
│   │   └── error.tsx               # Global error boundary
│   │
│   ├── components/                 # Reusable UI Component Hierarchy
│   │   ├── ui/                     # Primitives (Button, Modal, Input, Badge, Dropdown, Slider)
│   │   ├── video/                  # 🎬 COMPLETE VIDEO VIEWER & PLAYER SUBSYSTEM
│   │   │   ├── HlsPlayer.tsx       # Adaptive Bitrate (ABR) HLS.js video player core
│   │   │   ├── VideoControls.tsx   # Custom player controls (Play/Pause, Seek bar, Time display)
│   │   │   ├── QualitySelector.tsx # Resolution switcher (Auto, 1080p, 720p, 480p, 360p)
│   │   │   ├── PlaybackSpeed.tsx   # Speed menu (0.25x, 0.5x, 0.75x, 1x, 1.25x, 1.5x, 2x)
│   │   │   ├── TimelineThumbnail.tsx # Hover scrubber thumbnail preview (VTT storyboard sprite)
│   │   │   ├── CaptionsMenu.tsx    # Subtitles & closed captions switcher (.vtt)
│   │   │   ├── MiniPlayer.tsx      # Persistent floating picture-in-picture viewer
│   │   │   ├── TheaterMode.tsx     # Full-width theater layout toggle
│   │   │   ├── VolumeSlider.tsx    # Smooth volume controller with mute toggle
│   │   │   ├── KeyboardShortcutsModal.tsx # Shortcut cheat sheet (Space, K, J, L, F, M)
│   │   │   ├── VideoCard.tsx       # Video preview card with animated thumbnail & duration
│   │   │   ├── VideoActions.tsx    # Like, Dislike, Share, Save to Playlist buttons
│   │   │   └── VideoUploadDrop.tsx # Direct multipart video upload component
│   │   │
│   │   ├── course/                 # Course-specific components
│   │   │   ├── CourseCard.tsx      # Free/Paid course preview card
│   │   │   ├── CurriculumTree.tsx  # Nested chapter & lesson accordion
│   │   │   └── CertificateView.tsx # Downloadable completion certificate view
│   │   ├── chatbot/                # AI Chatbot Widget
│   │   │   ├── ChatFloatingBtn.tsx # Collapsible launcher button
│   │   │   ├── ChatWindow.tsx      # Message history and prompt input
│   │   │   └── ChatMessage.tsx     # Markdown-rendered response with course roadmaps
│   │   ├── comment/                # Nested comment tree & reply box
│   │   └── layout/                 # Header, Sidebar, UserMenu, NotificationBell
│   │
│   ├── hooks/                      # Custom React Hooks (Client Logic & Viewer Resources)
│   │   ├── useAuth.ts              # Authentication state, login, logout, user session
│   │   ├── useHlsPlayer.ts         # Player state, buffering events, quality level switcher
│   │   ├── useWatchProgress.ts     # Syncs playback position to backend (resume watching)
│   │   ├── useVideoAnalytics.ts    # Tracks buffer stalls, watch duration, completion rates
│   │   ├── useKeyboardShortcuts.ts # YouTube-style shortcuts (Space/K play, J/L seek, M mute, F full)
│   │   ├── useDebounce.ts          # Search input debouncing to reduce API traffic
│   │   └── useUpload.ts            # Presigned URL fetching and direct S3/R2 upload
│   │
│   ├── services/                   # API Integration Layer (Axios / Fetch)
│   │   ├── api.ts                  # Axios base client with JWT token interceptors
│   │   ├── videoService.ts         # Video streaming URLs, metadata, view count, watch history
│   │   ├── courseService.ts        # Course syllabus, enrollment, certificates
│   │   ├── channelService.ts       # Channel creation, subscriptions
│   │   ├── paymentService.ts       # Stripe/Razorpay checkout sessions
│   │   └── aiService.ts            # Chatbot question submission & search fallback
│   │
│   ├── store/                      # Global State Management (Zustand)
│   │   ├── useAuthStore.ts         # Logged-in user profile and permissions
│   │   └── usePlayerStore.ts       # Active video, playback time, mini-player, theater mode
│   │
│   ├── types/                      # TypeScript Interfaces (DTOs & Domain Models)
│   │   ├── user.ts                 # User profile, role (student, creator, admin)
│   │   ├── video.ts                # Video entity, resolution URLs, statistics
│   │   ├── player.ts               # Video viewer quality levels, subtitle tracks, player state
│   │   ├── course.ts               # Course entity, lesson items, enrollment status
│   │   └── api.ts                  # Standard API Response wrapper `{ data, error, meta }`
│   │
│   └── lib/                        # Pure Utilities & Helpers
│       ├── formatters.ts           # Format view counts (e.g. 1.2M), time elapsed ("2 days ago")
│       └── constants.ts            # Application constants, route links, CDN endpoints
│
├── .env.local                      # Local environment variables (API URL, Google Client ID)
├── .env.example                    # Template environment variables
├── package.json                    # Dependencies & build scripts (Next.js, React, Hls.js, Lucide)
├── tailwind.config.ts              # Tailwind CSS theme styling
├── tsconfig.json                   # TypeScript configuration
└── next.config.mjs                 # Next.js image domains, CSP headers, rewrites
```

---

## 🐍 4. Backend Architecture — `backend/` (FastAPI Modular Monolith)

The backend follows **Clean Architecture & Domain-Driven Design (DDD)**. The system is decoupled into **Controllers (API routes)**, **Services (Business rules)**, and **Repositories (Database access)**.

```
backend/
├── app/
│   ├── main.py                     # Application Factory: mounts routes, middleware, lifespan
│   ├── config.py                   # Pydantic BaseSettings: loads .env with strict type validation
│   │
│   ├── api/                        # CONTROLLER LAYER (Presentation / REST Endpoints)
│   │   ├── deps.py                 # Dependency Injection: get_db(), get_current_user(), require_role()
│   │   └── v1/
│   │       ├── router.py           # Master router registering all feature routers
│   │       └── endpoints/
│   │           ├── auth.py         # POST /login, /register, /google, /refresh
│   │           ├── users.py        # GET/PATCH /users/me, preferences, profile
│   │           ├── channels.py     # Channel CRUD, subscribe, studio settings
│   │           ├── videos.py       # Presigned upload URLs, video metadata, likes, watch history
│   │           ├── courses.py      # Course curriculum, pricing, enrollment
│   │           ├── comments.py     # Nested comment threads, reactions
│   │           ├── payments.py     # Checkout creation, webhook listener, invoicing
│   │           ├── ai.py           # POST /ai/chat, /ai/recommendations, semantic search
│   │           ├── admin.py        # Moderation, channel suspension, platform audits
│   │           └── health.py       # Health check: verifies DB, Redis, and Celery status
│   │
│   ├── core/                       # CROSS-CUTTING CONCERNS (Security & Infrastructure)
│   │   ├── database.py             # PostgreSQL async SQLAlchemy engine & session maker
│   │   ├── redis.py                # Redis client pool connection & cache helpers
│   │   ├── security.py             # Password hashing (bcrypt) & JWT token encode/decode
│   │   ├── oauth.py                # Google OAuth 2.0 verification with Authlib
│   │   ├── rate_limit.py           # SlowAPI sliding window rate limiter
│   │   └── exceptions.py           # Custom Domain Exceptions (NotFound, Forbidden, PaymentError)
│   │
│   ├── models/                     # DATA LAYER: SQLAlchemy ORM Entities (PostgreSQL)
│   │   ├── base.py                 # Declarative Base with id, created_at, updated_at
│   │   ├── user.py                 # User & UserRole tables
│   │   ├── channel.py              # Channel & ChannelSubscriber tables
│   │   ├── video.py                # Video, VideoReaction, VideoView, WatchHistory tables
│   │   ├── course.py               # Course, CourseLesson, CourseEnrollment tables
│   │   ├── comment.py              # Comment table (self-referencing parent_id)
│   │   ├── payment.py              # PaymentTransaction, SubscriptionPlan tables
│   │   └── notification.py         # Notification table
│   │
│   ├── schemas/                    # DTO LAYER: Pydantic v2 Request & Response Contracts
│   │   ├── user.py                 # UserCreate, UserLogin, UserResponse
│   │   ├── video.py                # VideoUploadRequest, VideoResponse, PresignedUrlResponse
│   │   ├── course.py               # CourseCreate, CourseDetailResponse, LessonSchema
│   │   ├── payment.py              # CheckoutSessionRequest, PaymentStatusResponse
│   │   └── ai.py                   # ChatRequest, ChatResponse, RecommendationResponse
│   │
│   ├── services/                   # BUSINESS LOGIC LAYER (Service Pattern)
│   │   ├── auth_service.py         # User registration, password verification, token issuance
│   │   ├── video_service.py        # Presigned upload URL generation, metadata persistence, progress
│   │   ├── course_service.py       # Course publishing, enrollment validation, certificate issue
│   │   ├── payment_service.py      # Idempotent Stripe webhook handling, entitlement grants
│   │   ├── ai_service.py           # LangChain/Gemini prompt templates, pgvector search
│   │   ├── notification_service.py # In-app notification creation & worker queuing
│   │   └── storage_service.py      # S3/Cloudflare R2 client wrapper (presigned URLs)
│   │
│   ├── repositories/               # DATA ACCESS LAYER (Repository Pattern)
│   │   ├── base_repository.py      # Generic CRUD repository (get, create, update, delete)
│   │   ├── user_repository.py      # User queries by email, username, google_id
│   │   ├── video_repository.py     # Video queries with pagination, channel filtering, watch history
│   │   ├── course_repository.py    # Course queries with enrolled students join
│   │   └── payment_repository.py   # Transaction logging with strict row-locking
│   │
│   ├── tasks/                      # ASYNCHRONOUS WORKER JOBS (Celery & Media Pipeline)
│   │   ├── video_tasks.py          # FFmpeg video transcoding to multi-bitrate HLS (.m3u8)
│   │   ├── storyboard_tasks.py     # Generates VTT storyboard sprite sheet for hover scrubbing
│   │   ├── subtitle_tasks.py       # Speech-to-text automated caption generation (.vtt)
│   │   ├── thumbnail_tasks.py      # Video frame extraction & thumbnail generation
│   │   ├── email_tasks.py          # Batch sending subscriber notification emails via SES
│   │   └── certificate_tasks.py    # PDF certificate generation upon course completion
│   │
│   ├── utils/                      # Helper Functions
│   │   ├── ffmpeg_wrapper.py       # Shell wrapper for FFmpeg commands
│   │   └── text_helpers.py         # Slugify titles, parse video duration
│   │
│   ├── celery_worker.py            # Celery entrypoint: initializes task app & broker
│   └── alembic/                    # Database Migrations (PostgreSQL)
│       ├── env.py                  # Migration environment configuration
│       └── versions/               # Auto-generated SQL migration scripts
│
├── tests/                          # Automated Testing Suite
│   ├── unit/                       # Tests for services, schemas, and helper logic
│   ├── integration/                # Tests for API endpoints using test PostgreSQL database
│   └── e2e/                        # End-to-end user flows (register -> upload -> watch)
│
├── .env.example                    # Template environment variables
├── requirements.txt                # Python dependencies
└── Dockerfile                      # Container definition for FastAPI backend
```

---

## 🏗️ 5. Infrastructure & Operations — `infrastructure/`

The `infrastructure/` directory contains all deployment, containerization, and cloud automation configurations:

```
infrastructure/
├── docker/
│   ├── backend.Dockerfile          # Optimized multi-stage build for FastAPI
│   ├── worker.Dockerfile           # Celery worker container with FFmpeg pre-installed
│   └── frontend.Dockerfile         # Next.js standalone output production container
│
├── terraform/                      # Infrastructure as Code (IaC)
│   ├── main.tf                     # Cloud provider definitions (AWS / GCP / Cloudflare)
│   ├── s3_r2.tf                    # Media storage buckets & CORS policies
│   ├── cloudfront.tf               # CDN edge distribution & signed URL key groups
│   └── rds.tf                      # Managed PostgreSQL database cluster
│
├── nginx/                          # Reverse Proxy (if self-hosting)
│   └── nginx.conf                  # SSL termination, rate limit buffer, gzip compression
│
└── scripts/                        # Automation & Deployment Scripts
    ├── setup_local_db.sh           # Local DB initialization & pgvector extension setup
    └── seed_test_data.py           # Inserts mock creators, courses, and sample videos
```

---

## 🔄 6. Tracing Key Workflows: High-Level to Low-Level

### Workflow 1: Direct Video Upload & HLS Transcoding

* **High-Level Flow:**
  `Browser ──(1) Get Upload URL──► FastAPI ──(2) Presigned URL──► Browser ──(3) Upload Raw Video──► Cloudflare R2 ──(4) Task──► Celery Worker (FFmpeg) ──(5) HLS .m3u8──► CDN`
* **Low-Level Code Path:**
  1. Frontend calls `videoService.ts:getUploadUrl()`.
  2. Handled by `backend/app/api/v1/endpoints/videos.py`.
  3. Controller delegates to `backend/app/services/storage_service.py` to generate a presigned PUT URL.
  4. Once uploaded, API enqueues a Celery task: `backend/app/tasks/video_tasks.py:transcode_to_hls`.
  5. Celery downloads raw MP4, runs `backend/app/utils/ffmpeg_wrapper.py` generating `360p`, `720p`, `1080p` `.ts` chunks and an `.m3u8` playlist.
  6. Transcoded files are pushed back to R2 and metadata is marked `READY` in `video_repository.py`.

---

### Workflow 2: Seamless Video Viewing, Adaptive Streaming & Progress Syncing

* **High-Level Flow:**
  `Next.js Watch Page ──(1) Fetch Manifest──► Cloudflare R2/CDN (master.m3u8) ──(2) Dynamic Bitrate Chunks──► HlsPlayer.tsx ──(3) Progress Heartbeat──► FastAPI ──► PostgreSQL WatchHistory`
* **Low-Level Code Path:**
  1. User navigates to `/watch/[videoId]`. Next.js Server Component fetches video metadata from `backend/app/api/v1/endpoints/videos.py`.
  2. `HlsPlayer.tsx` attaches `hls.js` to the `<video>` element, binding `master.m3u8`.
  3. Based on client bandwidth, `useHlsPlayer.ts` dynamically switches between `360p`, `720p`, and `1080p` video streams.
  4. Scrubber hover shows frame previews generated by `storyboard_tasks.py`.
  5. Every 10 seconds, `useWatchProgress.ts` posts a lightweight heartbeat to `PATCH /api/v1/videos/{id}/progress`, saving the exact second to `WatchHistory` in PostgreSQL.
  6. When the user returns from any device, playback automatically resumes from that saved timestamp.

---

### Workflow 3: Secure Course Enrollment & Payment

* **High-Level Flow:**
  `Student ──(1) Checkout──► Stripe/Razorpay ──(2) Webhook──► FastAPI ──(3) ACID Transaction──► PostgreSQL ──(4) Cache Invalidate──► Redis`
* **Low-Level Code Path:**
  1. Frontend calls `courseService.ts:enroll()`.
  2. Stripe webhook posts to `backend/app/api/v1/endpoints/payments.py:stripe_webhook`.
  3. Controller passes event payload to `backend/app/services/payment_service.py:process_payment`.
  4. Service initiates an atomic database transaction via `payment_repository.py` creating a payment record and inserting into `CourseEnrollment` in one commit.
  5. Redis cache key `user:{id}:enrolled_courses` is cleared to reflect immediate course access.

---

### Workflow 4: AI Chatbot & Semantic Search

* **High-Level Flow:**
  `Student Query ──► Next.js ──► FastAPI AI Router ──► pgvector (Embedding Match) ──► LLM (Gemini/OpenAI) ──► Course Roadmap Answer`
* **Low-Level Code Path:**
  1. User asks question in `ChatMessage.tsx`.
  2. Call dispatched to `backend/app/api/v1/endpoints/ai.py`.
  3. Endpoint calls `backend/app/services/ai_service.py:ask_ai_assistant`.
  4. Service queries PostgreSQL using `pgvector` similarity search to check if existing courses or video lessons address the prompt.
  5. The relevant course context is passed to the Gemini/OpenAI API with scientific & spiritual prompt instructions to generate a structured roadmap.

---

## 🎓 7. Educational Guide: How to Learn Both HLD & LLD

| To Learn...                                         | Study This Folder                                                  | What to Focus On                                                                                                                                                             |
| :-------------------------------------------------- | :----------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Video Player & Streaming (LLD)**            | `frontend/src/components/video/` & `hooks/`                    | Adaptive Bitrate streaming (ABR), VTT storyboard sprite scrubbing, subtitle tracks, and resume-playback state management.                                                    |
| **Microservices vs. Modular Monolith (HLD)**  | `backend/app/api/v1/endpoints/`                                  | Notice how each domain (auth, videos, courses, payments) is isolated. Any endpoint folder can be detached into its own microservice as traffic scales.                       |
| **Separation of Concerns (LLD)**              | `backend/app/api/` vs `services/` vs `repositories/`         | Endpoints only handle HTTP; Services handle business rules; Repositories handle database queries.                                                                            |
| **Design Patterns (LLD)**                     | `backend/app/services/storage_service.py`                        | Observe the **Strategy Pattern**: The interface allows swapping between AWS S3, Cloudflare R2, or local storage without altering API controllers.                      |
| **Asynchronous Distributed Processing (HLD)** | `backend/app/tasks/`                                             | Understand why heavy workloads (video encoding, storyboard sprites, bulk email) are pushed to Celery queues rather than blocking synchronous HTTP threads.                   |
| **Data Consistency & ACID (HLD/LLD)**         | `backend/app/models/` and `repositories/payment_repository.py` | Study foreign key constraints, indexes, and atomic transaction commits.                                                                                                      |
| **Modern Full-Stack Rendering (HLD/LLD)**     | `frontend/src/app/`                                              | Understand the distinction between Next.js **Server Components** (instant server rendering, zero bundle size) and **Client Components** (interactive UI states). |
