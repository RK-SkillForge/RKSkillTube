# 📚 RKSkillTube Documentation Hub

<div align="center">

[![Next.js](https://img.shields.io/badge/Frontend-Next.js%2014-black?style=for-the-badge&logo=next.js)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/Language-TypeScript-blue?style=for-the-badge&logo=typescript)](https://www.typescriptlang.org/)
[![FastAPI](https://img.shields.io/badge/Backend-FastAPI-009688?style=for-the-badge&logo=fastapi)](https://fastapi.tiangolo.com/)
[![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL%2016-336791?style=for-the-badge&logo=postgresql)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Cache-Redis%207-DC382D?style=for-the-badge&logo=redis)](https://redis.io/)
[![Docker](https://img.shields.io/badge/Orchestration-Docker-2496ED?style=for-the-badge&logo=docker)](https://www.docker.com/)

**The centralized engineering, architecture, and setup guide repository for RKSkillTube.**

</div>

---

## 🧭 Documentation Index

| Document | Purpose | Key Topics Covered |
| :--- | :--- | :--- |
| 🛠️ [**`tech_tool.md`**](./tech_tool.md) | **Technology & Tooling Rationale** | Complete breakdown of every library, runtime, and tool; architectural trade-offs; why Next.js, FastAPI, PostgreSQL, Redis, and FFmpeg were chosen over alternatives. |
| 🚀 [**`INSTALLATION_GUIDE.md`**](./INSTALLATION_GUIDE.md) | **Sequential Setup Guide** | Step-by-step setup from scratch: System prerequisites, Docker Compose, FastAPI backend (.venv), Next.js frontend (TypeScript), FFmpeg, CORS validation, and package lifecycle. |
| 🌐 [**`resource.md`**](./resource.md) | **Master Learning Hub** | Hand-curated official documentation links, interactive tutorials (Next.js Learn, Total TypeScript, FastAPI User Guide), top engineering blogs, and cheatsheets. |
| 📁 [**`folder.md`**](./folder.md) | **Repository & Folder Architecture** | High-level to low-level folder maps, clean architecture boundaries, the video viewer subsystem components, and end-to-end traced workflows. |

---

## 🎯 Which Guide Should I Open?

```
┌────────────────────────────────────────────────────────┐
│  "I want to understand what tech is used and why"      │ ──► Open tech_tool.md
├────────────────────────────────────────────────────────┤
│  "I want to run the project locally on my machine"     │ ──► Open INSTALLATION_GUIDE.md
├────────────────────────────────────────────────────────┤
│  "I'm learning this stack and need official tutorials" │ ──► Open resource.md
├────────────────────────────────────────────────────────┤
│  "I need to know where to put my code and components"  │ ──► Open folder.md
└────────────────────────────────────────────────────────┘
```

---

## ⚡ Quick Start Summary

For the full, detailed instructions, follow [`INSTALLATION_GUIDE.md`](./INSTALLATION_GUIDE.md). Here is the 60-second summary:

```powershell
# 1. Start Database, Cache & Workers (Docker)
docker compose up -d

# 2. Start Backend API (FastAPI)
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000

# 3. Start Frontend (Next.js + TypeScript)
cd ../frontend
npm install
npm run dev
```

* **Frontend:** `http://localhost:3000`
* **Backend API & Swagger Docs:** `http://localhost:8000/docs`
* **Celery Flower Dashboard:** `http://localhost:5555`

---

## 🤝 Contributing to Documentation
If you add new features, endpoints, or discover helpful blogs and tutorials, update the relevant files:
* Add technical justification to [`tech_tool.md`](./tech_tool.md).
* Add learning links to [`resource.md`](./resource.md).
* Add folder structure changes to [`folder.md`](./folder.md).
