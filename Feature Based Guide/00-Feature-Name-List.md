# 📋 RKSkillTube — Complete Feature Roadmap & Implementation Priority

> **Overview:** This document maps all functional capabilities and services from the High-Level System Design (`RKSkillTube.docx`) into structured implementation tiers: **🟢 Easy**, **🟡 Medium**, and **🔴 High (Complex)**.
> Each feature is prioritized based on architectural dependencies, technical complexity, external integrations, and distributed processing requirements.

---

## 🟢 Tier 1: Easy Features (Foundational CRUD & Core Entities)

*Target: Phase 1 — Build first to establish database models, basic APIs, and core platform relationships without complex background jobs or external paid APIs.*

### 1. Authentication & User Management

* **Complexity:** Easy
* **Scope & Capabilities:**
  * User registration and login (Email + Password with bcrypt hashing).
  * JWT access token generation and refresh token rotation (`/auth/login`, `/auth/refresh`, `/auth/logout`).
  * User profile management: name, username, bio, avatar, interests, settings (`/users/me`, `/users/me/settings`).
  * Role-based access control flags: Student (`USER`), Creator (`CREATOR`), and Administrator (`ADMIN`).

### 2. Free Channel Management (Base Channel System)

* **Complexity:** Easy
* **Scope & Capabilities:**
  * One-to-one channel creation for authenticated users (`POST /channels`).
  * Channel profile metadata: channel name, handle (`@handle`), default random URL slug, bio, avatar, and banner image.
  * Channel external social links management: GitHub, LinkedIn, YouTube, Instagram (`/channels/{id}/social-links`).
  * Public channel page retrieval by handle or slug.

### 3. Playlists & "Watch Later" Queue 

* **Complexity:** Easy
* **Scope & Capabilities:**
  * Custom user playlists: create, rename, set public/private visibility (`/playlists`).
  * Playlist items management: add video, remove video, and reorder video positions (`/playlists/{id}/videos`).
  * Built-in dedicated system playlist: "Watch Later" queue with quick toggle (`POST/DELETE /users/me/watch-later/{video_id}`).

### 4. Social Interactions & Engagement 

* **Complexity:** Easy
* **Scope & Capabilities:**
  * Video Reactions: Like and Dislike toggle with idempotent upsert (`PUT/DELETE /videos/{id}/reaction`).
  * Channel Subscriptions: Free channel subscribe/unsubscribe toggle (`POST/DELETE /channels/{id}/subscribe`).
  * Commenting System: Top-level comments on videos (`POST /videos/{id}/comments`).
  * Threaded Replies: 1-level nested comment replies using `parent_comment_id` self-referential foreign key.

---

## 🟡 Tier 2: Medium Features (Business Workflows, File Pipelines & Lifecycle States)

*Target: Phase 2 — Multi-entity workflows, state machines, file upload handshakes, and operational portals.*

### 5. Video Management & Cloud Ingestion Handshake

* **Complexity:** Medium
* **Scope & Capabilities:**
  * Pre-signed cloud storage URL generation for direct client uploads (`POST /videos/upload-url`).
  * Video metadata registration: Title, description, tags, thumbnail URL, duration, visibility (`PUBLIC`, `UNLISTED`, `PRIVATE`).
  * Basic video playback endpoint resolving stream manifest or storage key (`GET /videos/{id}/playback`).
  * Video view count recording with rate limiting (`POST /videos/{id}/view`).

### 6. Course Curriculum Builder & Enrollment System

* **Complexity:** Medium
* **Scope & Capabilities:**
  * Course creation: Title, description, cover image, category, access type (`FREE` / `PAID`), and source (`PLATFORM` / `YOUTUBE`).
  * Modular syllabus builder: attaching platform videos or external YouTube video links in ordered chapters/lessons (`/courses/{id}/videos`).
  * Student enrollment lifecycle: free one-click enrollment or post-purchase enrollment entitlement (`POST /courses/{id}/enroll`).
  * Student enrolled courses library (`GET /users/me/courses`).

### 7. Course Progress Tracking & Certificate System

* **Complexity:** Medium
* **Scope & Capabilities:**
  * Granular playback progress heartbeat recording per lesson (`PUT /courses/{id}/lessons/{videoId}/progress`).
  * Course completion calculation (evaluating 100% completed lessons).
  * Certificate eligibility verification endpoint (`GET /courses/{id}/certificate/eligibility`).
  * Dynamic certificate generation with unique certificate serial number (`POST /courses/{id}/certificate`).
  * Public tamper-proof certificate verification route (`GET /certificates/verify/{certificate_number}`).

### 8. Channel Memberships & Paid Perks *(Separated from Base Channels)*

* **Complexity:** Medium
* **Scope & Capabilities:**
  * Creator-defined membership plans and tier pricing (`POST /channels/{id}/membership-plans`).
  * Member status tracking: active billing period, expiration date, renewal state.
  * Vanity URL slug and custom domain assignment for verified/paid channels (`/channels/slug/{slug}`).
  * Creator broadcast permissions: member-only video visibility and exclusive perks.

### 9. In-App Notification Center & Notification Settings

* **Complexity:** Medium
* **Scope & Capabilities:**
  * User in-app notification inbox with pagination (`GET /users/me/notifications`).
  * Unread notification count badge (`GET /users/me/notifications/unread-count`).
  * Batch and single notification state updates (`PATCH /notifications/{id}/read`, `POST /users/me/notifications/read-all`).
  * User notification preferences configuration (toggle email vs. in-app for comments, uploads, milestone alerts).

### 10. Admin Governance & Platform Operations Dashboard

* **Complexity:** Medium
* **Scope & Capabilities:**
  * System overview metrics: total users, active channels, course counts, platform revenue.
  * User & Channel moderation: view user list, ban/suspend accounts, update channel verification badge (`PATCH /admin/users/{id}/status`).
  * Content review queue: manual review of flagged videos and course materials (`GET /admin/videos/review`).
  * Financial transaction ledger audit view (`GET /admin/payments`).

---

## 🔴 Tier 3: High / Complex Features (Distributed Systems, Real-Time Workers, AI & Financials)

*Target: Phase 3 — Heavy background processing, external payment gateways, distributed queues, vector similarity, and LLM orchestration.*

### 11. Payments, Checkout & Idempotent Webhook Engine

* **Complexity:** High / Complex
* **Scope & Capabilities:**
  * Multi-gateway checkout session creation (Stripe & Razorpay) for course purchases, membership tiers, and promotion campaigns (`POST /memberships/checkout`, `POST /promotions/checkout`).
  * Secure webhook receiver handling asynchronous payment capture, payment intent confirmation, and failure events.
  * Idempotent payment processing with transactional ledger (`payments` table) ensuring zero double-charging.
  * Automated revenue-split calculation between platform fee and creator payout ledger.
  * Refund processing and membership subscription recurring billing lifecycle.

### 12. Video Transcoding, Adaptive Bitrate (ABR) & CDN Streaming Worker *(Separated from Base Video)*

* **Complexity:** High / Complex
* **Scope & Capabilities:**
  * Asynchronous background job queue (Celery + Redis) triggered upon upload completion.
  * FFmpeg transcoding pipeline: generating multi-bitrate HLS streams (`360p`, `480p`, `720p`, `1080p`) and `.m3u8` master playlists.
  * Video scrubber hover preview generation (WebVTT + tiled JPEG sprite sheet).
  * Storage optimization and edge CDN delivery with pre-signed video token authorization.

### 13. Course Promotion Engine & Sponsored Discovery *(Separated from Course Builder)*

* **Complexity:** High / Complex
* **Scope & Capabilities:**
  * Creator promotion campaign setup: budget, promotion duration, target audience (`POST /courses/{id}/promotions`).
  * Promotion checkout and payment lock.
  * Admin approval/rejection moderation pipeline (`POST /admin/promotions/{id}/approve|reject`).
  * Featured course slot scheduler and sponsored card injection into search results and feeds (`GET /promotions/featured`).

### 14. Feed & Algorithmic Recommendation Engine 

* **Complexity:** High / Complex
* **Scope & Capabilities:**
  * Home Discovery Feed: Hybrid aggregation combining trending videos, enrolled courses, and popular channels.
  * Subscriptions Feed: Time-decay chronological stream of uploads from subscribed creator channels.
  * Recommendation Engine: Collaborative filtering and engagement-weighted ranking (watch duration, reaction ratio, user interest tags).

### 15. Multi-Channel Notification Dispatch Worker Engine **Complexity:** High / Complex

* **Scope & Capabilities:**
  * Event-driven messaging bus triggered on events (e.g., creator uploads new video/course, comment replies, subscriber milestones).
  * Asynchronous worker fan-out for large subscriber lists (e.g., emailing 10,000+ subscribers without blocking the API).
  * Email template rendering (SendGrid / AWS SES) and delivery status tracking (`notification_deliveries` table).
  * Web push notification dispatcher.

### 16. AI Chatbot Learning Assistant

* **Complexity:** High / Complex
* **Scope & Capabilities:**
  * Interactive conversational assistant with Google Gemini LLM integration (`POST /ai/chat`).
  * Domain-specific dual algorithm: combining scientific curriculum roadmaps with spiritual guidance/motivation.
  * Context-aware prompt engineering: ground answers in platform courses, video lessons, and creator tutorials.
  * User learning pathway generator and study tips recommendation.

### 17. Semantic AI Search Engine & Fallback Intelligence

* **Complexity:** High / Complex
* **Scope & Capabilities:**
  * Hybrid search architecture (`GET /search?q=...`):
    * Primary: PostgreSQL full-text search across titles, descriptions, and tags.
    * Fallback & Semantic: `pgvector` vector embeddings and cosine similarity search for concepts without exact keyword matches.
  * AI-Powered Fallback: When content is unavailable on RKSkillTube, invoke AI to formulate curated learning recommendations and indexed YouTube references.
  * Multi-entity search filters: Courses, Videos, Channels, and Playlists.

### 18. Automated Content Moderation & Policy Violation System

* **Complexity:** High / Complex
* **Scope & Capabilities:**
  * Automated content screening pipeline analyzing video titles, descriptions, transcripts, and comment texts for policy violations.
  * Automated strike and penalty engine: warning, video takedown, and automated channel suspension on reaching strike threshold.
  * Admin violation audit log with policy override and restoration capabilities.

### 19. Creator Studio Analytics Engine & Telemetry 

* **Complexity:** High / Complex
* **Scope & Capabilities:**
  * High-throughput time-series metrics collection: views, unique impressions, watch time retention curves.
  * Creator Studio Dashboard (`/creator/studio`, `/creator/analytics/overview`):
    * Overall channel performance (30d/90d/all-time).
    * Video-level and course-level engagement breakdowns.
    * Real-time subscriber acquisition and churn graphs.
    * Monetization and net earnings estimation.
  * Automatic creator milestone notification triggers (e.g., 100 subscribers, 1,000 views).

---

## 🗺️ Recommended Step-by-Step Implementation Roadmap

```
PHASE 1: FOUNDATION (Easy)
  ├── 1. Authentication & User Management (JWT, Roles)
  ├── 2. Free Channel Management & Profiles
  ├── 3. Playlists & "Watch Later" Queue
  └── 4. Social Interactions (Likes, Comments, Subscriptions)
          │
          ▼
PHASE 2: CORE WORKFLOWS (Medium)
  ├── 5. Video Cloud Ingestion & Direct Upload Handshake
  ├── 6. Course Syllabus Builder & Enrollments
  ├── 7. Course Progress Tracking & Public Certificate Verification
  ├── 8. Channel Memberships & Paid Perks (Tiers, Vanity URLs)
  ├── 9. In-App Notification Center & Preferences
  └── 10. Admin Governance & Platform Operations Portal
          │
          ▼
PHASE 3: DISTRIBUTED & ADVANCED SYSTEMS (High / Complex)
  ├── 11. Payments, Checkout & Webhook Ledger (Stripe/Razorpay)
  ├── 12. Video Transcoding (FFmpeg, Multi-bitrate HLS, Celery)
  ├── 13. Course Promotion Engine & Sponsored Discovery
  ├── 14. Feed & Algorithmic Recommendation Engine
  ├── 15. Multi-Channel Notification Dispatch Worker (Email/Push)
  ├── 16. AI Chatbot Assistant (Google Gemini LLM, Spiritual/Scientific)
  ├── 17. Semantic AI Search & Vector Fallback (pgvector)
  ├── 18. Automated Content Moderation & Strike System
  └── 19. Creator Studio Analytics Engine & Telemetry
```
