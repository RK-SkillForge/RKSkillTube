# 🎓 RKSkillTube — Next-Generation Video & Course Learning Platform

<div align="center">

[![Next.js](https://img.shields.io/badge/Frontend-Next.js%2014%20App%20Router-black?style=for-the-badge&logo=next.js)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/Language-TypeScript-blue?style=for-the-badge&logo=typescript)](https://www.typescriptlang.org/)
[![FastAPI](https://img.shields.io/badge/Backend-FastAPI%20ASGI-009688?style=for-the-badge&logo=fastapi)](https://fastapi.tiangolo.com/)
[![Python](https://img.shields.io/badge/Python-3.11%20%7C%203.12-3776AB?style=for-the-badge&logo=python)](https://www.python.org/)
[![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL%2016%20%2B%20pgvector-336791?style=for-the-badge&logo=postgresql)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Cache-Redis%207-DC382D?style=for-the-badge&logo=redis)](https://redis.io/)
[![Docker](https://img.shields.io/badge/Containers-Docker%20Compose-2496ED?style=for-the-badge&logo=docker)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](./LICENSE)

**A high-performance, hybrid video streaming and EdTech platform built for engineering students, creators, and educators.**

[Explore Documentation](./Docs/README.md) • [Installation Guide](./Docs/INSTALLATION_GUIDE.md) • [Tech Rationale](./Docs/tech_tool.md) • [Resource Hub](./Docs/resource.md)

</div>

---

## 📖 Overview

**RKSkillTube** combines the engagement and dynamic content of **YouTube** with the structured, syllabus-driven curriculum of **Coursera/Udemy**. Built to serve Tier-2 & Tier-3 engineering students, it provides high-quality technical education, AI-assisted interactive learning, and seamless monetization for creators.

### 🌟 Key Highlights
* **Adaptive Bitrate (ABR) Video Streaming:** HLS.js streaming supporting `1080p`, `720p`, `480p`, and `360p` with zero client buffering.
* **Timeline Hover Scrubber:** YouTube-style frame previews using automated FFmpeg VTT storyboard sprite sheets.
* **Curriculum & Lesson Builder:** Free and paid structured courses with modular chapters, video lessons, and completion certificates.
* **AI-Powered Semantic Assistant:** Integrated Google Gemini API and `pgvector` similarity search to generate roadmaps and answer student technical queries.
* **Creator Studio Dashboard:** Video analytics, view counts, engagement graphs, and automated payout reporting.
* **Secure Payment Processing:** Stripe and Razorpay integrations with idempotent webhook transaction logging.

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    CLIENT BROWSER / MOBILE                  │
└──────────────────────────────┬──────────────────────────────┘
                               │
            ┌──────────────────┴──────────────────┐
            ▼                                     ▼
┌──────────────────────────────┐    ┌──────────────────────────────┐
│  FRONTEND: Next.js 14+ (TS)  │    │      GLOBAL EDGE CDN         │
│  • App Router (SSR & SEO)    │    │  • Cloud CDN / Cloudflare    │
│  • HLS.js Custom Video Player│    │  • Cached .m3u8 & .ts chunks │
│  • Zustand Client State      │    │  • Sub-30ms global latency   │
└──────────────┬───────────────┘    └──────────────▲───────────────┘
               │ HTTP REST (JSON)                  │
               ▼                                   │ Media Chunks
┌──────────────────────────────────────────────┐   │
│         BACKEND: FastAPI (Python 3.11+)      │   │
│  • Async Starlette + Uvicorn ASGI Server     │   │
│  • Rust-powered Pydantic v2 Type Validation  │   │
│  • Interactive OpenAPI Docs (/docs, /redoc)  │   │
└──────────────┬───────────────────────┬───────┘   │
               │                       │           │
         ┌─────┴─────┐           ┌─────┴─────┐     │
         ▼           ▼           ▼           ▼     │
┌─────────────────┐ ┌─────────┐ ┌─────────┐ ┌──────┴──────┐
│  PostgreSQL 16  │ │ Redis 7 │ │ Celery  │ │Google Cloud │
│  • Relational DB│ │ • Cache │ │ Worker  │ │Storage (GCS)│
│  • pgvector     │ │ • Broker│ │ • FFmpeg│ │ / R2 / S3   │
│  • Cloud SQL    │ │ • Limits│ │ • Emails│ │ (Media Store│
└─────────────────┘ └─────────┘ └─────────┘ └─────────────┘
```

---

## 🛠️ Technology Toolstack

| Component | Selected Technology | Why It Was Chosen |
| :--- | :--- | :--- |
| **Frontend Framework** | **Next.js 14+ (App Router)** | Server Components for instant initial paint, SEO, and route handling. |
| **Language (Frontend)** | **TypeScript** | Strict compile-time safety preventing `undefined` errors at scale. |
| **Styling** | **Tailwind CSS + Radix UI** | Utility-first styling with zero CSS bloat and accessible UI primitives. |
| **Video Playback** | **HLS.js & WebVTT** | Adaptive streaming dynamically adjusting to client bandwidth. |
| **Backend API** | **FastAPI (ASGI)** | High-concurrency async Python with automatic Swagger UI documentation. |
| **Data Validation** | **Pydantic v2** | Rust-powered schema validation executing up to 15x faster than v1. |
| **Database** | **PostgreSQL 16 + pgvector** | Strict ACID financial guarantees and native AI vector similarity search. |
| **Cache & Task Queue**| **Redis 7 + Celery** | Sub-millisecond write buffering and decoupled FFmpeg video transcoding. |
| **Media Processing** | **FFmpeg** | Transcodes raw creator uploads into multi-bitrate HLS streams & sprites. |
| **Object Storage** | **Google Cloud Storage / R2** | Scalable cloud storage with presigned URLs for media & HLS streams. |

*👉 For an in-depth explanation of every tool, read [`Docs/tech_tool.md`](./Docs/tech_tool.md).*

---

## ⚡ Quick Start: Running Locally

### 1. Prerequisites
Make sure you have installed:
* [Node.js (v20+ LTS)](https://nodejs.org/)
* [Python (3.11+)](https://www.python.org/)
* [Docker Desktop](https://www.docker.com/)
* [FFmpeg](https://ffmpeg.org/)

### 2. Boot Infrastructure (Terminal 1)
```powershell
docker compose up -d
```
* PostgreSQL: `localhost:5432` | Redis: `localhost:6379` | Flower: `http://localhost:5555`

### 3. Launch Backend API (Terminal 2)
```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1    # (On macOS/Linux: source .venv/bin/activate)
pip install -r requirements.txt
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```
* **API Documentation (Swagger):** `http://localhost:8000/docs`

### 4. Launch Celery Worker (Terminal 3)
```powershell
cd backend
.\.venv\Scripts\Activate.ps1
celery -A app.celery_worker.celery_app worker --loglevel=info --pool=solo
```

### 5. Launch Frontend (Terminal 4)
```powershell
cd frontend
npm install
npm run dev
```
* **Web Application:** `http://localhost:3000`

---

## 📚 Documentation Suite

All in-depth guides are organized in the [`Docs/`](./Docs/README.md) directory:

* 🛠️ [**`Docs/INSTALLATION_GUIDE.md`**](./Docs/INSTALLATION_GUIDE.md) — Complete sequential installation guide for Windows, macOS, and Linux.
* 📋 [**`Docs/tech_tool.md`**](./Docs/tech_tool.md) — Comprehensive technical breakdown of which technology is used and why.
* 🌐 [**`Docs/resource.md`**](./Docs/resource.md) — Centralized learning hub with official tutorials, interactive courses, and blogs.
* 📁 [**`Docs/folder.md`**](./Docs/folder.md) — High-level and low-level directory architecture, video player subsystem, and workflows.

---

## 🗂️ Repository Structure

```
RKSkillTube/
├── .github/                        # CI/CD Workflows (GitHub Actions)
├── frontend/                       # Next.js 14+ (App Router with TypeScript)
├── backend/                        # Python FastAPI (ASGI) Core Application
├── infrastructure/                 # Dockerfiles, Nginx configs, and cloud scripts
├── Docs/                           # Central Project Documentation Hub
├── SD Of RKSkillTube/              # High-Level & Low-Level System Design Specs
├── Feature Based Guide/            # Step-by-step modular feature specifications
├── docker-compose.yml              # Local orchestration (PostgreSQL, Redis, Flower)
├── README.md                       # Main repository overview (this file)
└── LICENSE                         # MIT License
```

---

## 🤝 Contributing

We welcome contributions from engineers, educators, and open-source enthusiasts!  
Please review [`CONTRIBUTING.md`](./CONTRIBUTING.md) and [`CODE_OF_CONDUCT.md`](./CODE_OF_CONDUCT.md) before submitting pull requests.

---

## 📄 License

This project is licensed under the [MIT License](./LICENSE).
