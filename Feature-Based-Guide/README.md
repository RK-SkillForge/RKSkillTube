# 🧩 RKSkillTube — Feature-Based Architecture Guide

> **Directory Purpose:** This directory breaks down the **8 core bounded feature contexts** of **RKSkillTube**. Each feature is designed with clear architectural boundaries spanning database models, backend API controllers, business services, and frontend Next.js views.

---

## 📑 Feature Modules Overview

| # | Feature Domain | Backend Module | Frontend Route | Key Capabilities |
| :-: | :--- | :--- | :--- | :--- |
| **01** | **Authentication & User Management** | `app/api/v1/endpoints/auth.py`<br>`app/api/v1/endpoints/users.py` | `/login`<br>`/register`<br>`/profile` | Email/password authentication, Google OAuth 2.0, JWT token rotation, user profile and role-based permissions (Student, Creator, Admin). |
| **02** | **Free & Paid Channels** | `app/api/v1/endpoints/channels.py` | `/channel/[handle]`<br>`/studio` | Channel customization, creator banner/avatar, subscriber counts, membership tiers, and channel community feed. |
| **03** | **Video Streaming & Transcoding** | `app/api/v1/endpoints/videos.py`<br>`app/tasks/video_tasks.py` | `/watch/[videoId]`<br>`/studio/upload` | Presigned direct multipart upload, FFmpeg multi-bitrate HLS packaging (`360p`–`1080p`), VTT scrubber hover previews, and watch progress heartbeats. |
| **04** | **Free & Paid Courses** | `app/api/v1/endpoints/courses.py`<br>`app/tasks/certificate_tasks.py` | `/courses`<br>`/courses/[courseId]` | Nested syllabus builder (Chapters & Lessons), student enrollment tracking, course completion certificates (PDF), and promotional coupons. |
| **05** | **AI Chatbot Assistant** | `app/api/v1/endpoints/ai.py` | Floating Widget (`ChatWindow.tsx`) | Scientific & spiritual learning roadmap generator, dynamic query resolution using Google Gemini API and contextual prompt templates. |
| **06** | **Semantic AI Search** | `app/api/v1/endpoints/ai.py` | `/search?q=...` | Hybrid search combining PostgreSQL full-text search with `pgvector` vector similarity embeddings (Cosine distance). |
| **07** | **Admin System & Moderation** | `app/api/v1/endpoints/admin.py` | `/admin/dashboard`<br>`/admin/moderation` | Automated & manual video review, platform health telemetry, channel suspension, and creator payout auditing. |
| **08** | **Payments & Monetization** | `app/api/v1/endpoints/payments.py`<br>`app/services/payment_service.py` | `/checkout/[id]`<br>`/studio/analytics` | Stripe & Razorpay checkout integration, idempotent webhook processing, creator revenue split calculation, and invoices. |

---

## 🗺️ Feature Architecture Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    STUDENT / CREATOR / ADMIN                │
└──────────────────────────────┬──────────────────────────────┘
                               │
            ┌──────────────────┴──────────────────┐
            ▼                                     ▼
┌──────────────────────────────┐    ┌──────────────────────────────┐
│       LEARNER ACTIONS        │    │       CREATOR ACTIONS        │
│  • Watch Video (HLS Stream)  │    │  • Direct Multipart Upload   │
│  • Enroll in Free/Paid Course│    │  • Build Modular Curriculum  │
│  • Ask AI Learning Assistant │    │  • Review Studio Analytics   │
│  • Comment & Community Like  │    │  • Withdraw Channel Earnings │
└──────────────┬───────────────┘    └──────────────┬───────────────┘
               │                                   │
               └─────────────────┬─────────────────┘
                                 │ HTTP REST (JSON)
                                 ▼
┌─────────────────────────────────────────────────────────────┐
│                   FASTAPI ROUTER DISPATCHER                 │
│  /auth  •  /users  •  /channels  •  /videos  •  /courses    │
│            /payments  •  /ai  •  /admin  •  /health         │
└──────────────────────────────┬──────────────────────────────┘
                               │
         ┌─────────────────────┼─────────────────────┐
         ▼                     ▼                     ▼
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│   PostgreSQL    │   │  Redis 7 Cache  │   │  Celery Worker  │
│  ACID Models    │   │  Views & Broker │   │  FFmpeg & Email │
└─────────────────┘   └─────────────────┘   └─────────────────┘
```

---

## 📂 Related Documentation
* List of feature specifications: [`00-Feature-Name-List.md`](./00-Feature-Name-List.md)
* Technical rationale for each feature: [`../Docs/tech_tool.md`](../Docs/tech_tool.md)
* Low-level design & folder mapping: [`../Docs/folder.md`](../Docs/folder.md)
