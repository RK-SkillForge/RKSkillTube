# 🔬 RKSkillTube — Low-Level Design (LLD) Specification

> **Directory Purpose:** This document defines the **Low-Level Design (LLD)** for RKSkillTube. It specifies the clean architecture layers, design patterns, database entity relationships, and type contracts governing the backend and frontend codebases.

---

## 🏛️ 1. Clean Architecture Layers

The backend strictly separates presentation, business logic, data access, and database models:

```
┌────────────────────────────────────────────────────────┐
│ 1. PRESENTATION LAYER (API Controllers / Endpoints)    │
│    • Handles HTTP, routing, query params, auth checks. │
│    • Location: backend/app/api/v1/endpoints/           │
├────────────────────────────────────────────────────────┤
│ 2. DOMAIN / BUSINESS LOGIC LAYER (Services)            │
│    • Encapsulates business rules, validation, logic.   │
│    • Location: backend/app/services/                   │
├────────────────────────────────────────────────────────┤
│ 3. DATA ACCESS LAYER (Repositories)                    │
│    • Executes database queries, joins, and filters.    │
│    • Location: backend/app/repositories/               │
├────────────────────────────────────────────────────────┤
│ 4. PERSISTENCE LAYER (SQLAlchemy Models)               │
│    • Declarative database schemas and foreign keys.    │
│    • Location: backend/app/models/                     │
└────────────────────────────────────────────────────────┘
```

---

## 🎨 2. Software Design Patterns in RKSkillTube

### 2.1 Strategy Pattern (Video & Object Storage)
* **Problem:** We need to support Cloudflare R2 in production ($0 egress) but may use AWS S3 or local storage in testing.
* **Solution:** A unified `StorageService` interface with pluggable strategies (`CloudflareR2Strategy`, `S3Strategy`, `LocalStorageStrategy`). Endpoints only invoke `storage.get_upload_url()` without knowing the cloud provider.

### 2.2 Dependency Injection Pattern
* **Problem:** Tightly coupled database sessions and user authorization make code fragile and untestable.
* **Solution:** FastAPI's `Depends()` injects database sessions (`get_db`), active authenticated users (`get_current_user`), and role requirements (`require_admin`).

### 2.3 Repository Pattern
* **Problem:** Scattering SQL queries inside API routes makes it difficult to change or optimize database schemas.
* **Solution:** Every entity has a repository (`VideoRepository`, `CourseRepository`, `UserRepository`) providing clean CRUD operations.

### 2.4 Observer / Event-Driven Pattern (Task Dispatch)
* **Problem:** Encoding video files or dispatching emails inside HTTP requests blocks the server.
* **Solution:** Endpoints publish tasks to the Celery broker (`video_tasks.transcode_to_hls.delay(video_id)`), returning immediate responses to users while workers process asynchronously.

---

## 🗄️ 3. Core Database Entities & Relationships (PostgreSQL)

```
┌─────────────────┐       1:N       ┌─────────────────┐
│     User        ├────────────────►│     Channel     │
│ (id, email, ...)│                 │ (id, user_id,..)│
└────────┬────────┘                 └────────┬────────┘
         │ 1:N                               │ 1:N
         ▼                                   ▼
┌─────────────────┐       1:N       ┌─────────────────┐
│   Enrollment    │◄────────────────┤     Course      │
│ (user_id, course│                 │ (id, channel_id)│
└─────────────────┘                 └────────┬────────┘
         ▲                                   │ 1:N
         │ N:1                               ▼
┌─────────────────┐       N:1       ┌─────────────────┐
│  WatchHistory   │◄────────────────┤      Video      │
│ (user_id, video)│                 │ (id, hls_url..) │
└─────────────────┘                 └─────────────────┘
```

* **User**: Primary authentication credentials, OAuth IDs, and role assignments (`student`, `creator`, `admin`).
* **Channel**: Creator identity, subscriber counts, and channel metadata.
* **Video**: Title, description, HLS playlist URL (`.m3u8`), duration, thumbnail URL, and view count.
* **Course**: Curriculum container, pricing (free vs paid), level, and published status.
* **CourseLesson**: Individual video lessons nested within chapters.
* **WatchHistory**: Records the exact playback timestamp per user so learning progress can be resumed across devices.
* **PaymentTransaction**: Audit-compliant financial records for course purchases and memberships with row locking.

---

## 🛡️ 4. Data Transfer Objects (DTOs) & Type Contracts

To ensure complete type safety:
* **Backend:** Pydantic v2 schemas (`backend/app/schemas/`) strictly validate incoming request bodies and serialize responses.
* **Frontend:** TypeScript interfaces (`frontend/src/types/`) mirror backend schemas, guaranteeing zero runtime undefined errors.

---

### 📖 Related References
* High-level architectural specifications: [`../00-High-Level-Design/README.md`](../00-High-Level-Design/README.md)
* Codebase directory layout: [`../../Docs/folder.md`](../../Docs/folder.md)
* Local environment setup: [`../../Docs/INSTALLATION_GUIDE.md`](../../Docs/INSTALLATION_GUIDE.md)
