# 🛠️ RKSkillTube — Complete Sequential Installation & Full-Stack Setup Guide

> **Project Architecture:** Next.js 14+ (TypeScript Frontend) + FastAPI (Python Backend) + PostgreSQL 16 (with pgvector) + Redis 7 + Celery + FFmpeg + Cloudflare R2 / AWS S3 + Docker.  
> **Platform Support:** Fully documented for **Windows (PowerShell)**, **macOS**, and **Linux**.

---

## 📋 Table of Contents
1. [Prerequisites & System Toolstack](#1-prerequisites--system-toolstack)
2. [Step 1: Clone Repository & Workspace Layout](#step-1-clone-repository--workspace-layout)
3. [Step 2: Backing Infrastructure Setup via Docker Compose](#step-2-backing-infrastructure-setup-via-docker-compose)
4. [Step 3: Backend Setup (Python, FastAPI & Celery)](#step-3-backend-setup-python-fastapi--celery)
5. [Step 4: Frontend Setup (Next.js 14+ with TypeScript)](#step-4-frontend-setup-nextjs-14-with-typescript)
6. [Step 5: Media Processing Setup (FFmpeg)](#step-5-media-processing-setup-ffmpeg)
7. [Step 6: Full-Stack Connectivity & CORS Verification](#step-6-full-stack-connectivity--cors-verification)
8. [Step 7: Framework & Package Lifecycle Management (Uninstall, Reset, Re-add)](#step-7-framework--package-lifecycle-management)
9. [Step 8: Running the Full Development Environment (4 Terminals)](#step-8-running-the-full-development-environment)
10. [Step 9: Verification Checklist & Health Endpoints](#step-9-verification-checklist--health-endpoints)
11. [Step 10: Troubleshooting Common Issues](#step-10-troubleshooting-common-issues)
12. [Step 11: Top Learning Resources & Official Guides](#step-11-top-learning-resources--official-guides)

---

## 1. Prerequisites & System Toolstack

Install the required foundational runtimes on your operating system before setting up code:

| Tool | Recommended Version | Purpose | Installation Command |
| :--- | :--- | :--- | :--- |
| **Node.js** | v20.x or v22.x LTS | Next.js frontend runtime & package manager | `winget install OpenJS.NodeJS.LTS` *(Win)* <br> `brew install node` *(Mac)* |
| **Python** | 3.11.x or 3.12.x | FastAPI backend & Celery worker runtime | `winget install Python.Python.3.11` *(Win)* <br> `brew install python@3.11` *(Mac)* |
| **Docker Desktop** | Latest (v26+) | Runs PostgreSQL, Redis, and local services in isolated containers | `winget install Docker.DockerDesktop` *(Win)* <br> `brew install --cask docker` *(Mac)* |
| **FFmpeg** | 6.0+ | Video transcoding to HLS (`.m3u8`) and thumbnail generation | `winget install Gyan.FFmpeg` *(Win)* <br> `brew install ffmpeg` *(Mac)* |
| **Git** | Latest | Version control and repository management | `winget install Git.Git` *(Win)* <br> `brew install git` *(Mac)* |

> [!TIP]
> **Windows Users:** After installing tools, close and reopen your PowerShell window to refresh environment `$PATH` variables. Verify all installations:
> ```powershell
> node -v
> python --version
> docker --version
> ffmpeg -version
> git --version
> ```

---

## Step 1: Clone Repository & Workspace Layout

```powershell
# 1. Clone repository
git clone https://github.com/your-org/RKSkillTube.git

# 2. Enter workspace root
cd RKSkillTube
```

Your root workspace contains:
* `frontend/` — Next.js 14+ App Router with TypeScript.
* `backend/` — Python FastAPI core application and Celery workers.
* `infrastructure/` — Dockerfiles, Nginx configurations, and cloud templates.
* `Docs/` — System design and technical documentation hub.

---

## Step 2: Backing Infrastructure Setup via Docker Compose

Instead of manually installing PostgreSQL and Redis onto your host machine, run them inside containerized environments:

### 2.1 Verify or Create `docker-compose.yml` in Root:
```yaml
version: '3.8'

services:
  # Primary Relational & Vector Database
  postgres:
    image: pgvector/pgvector:pg16
    container_name: rkskilltube-postgres
    restart: always
    environment:
      POSTGRES_USER: rkuser
      POSTGRES_PASSWORD: rkpassword123
      POSTGRES_DB: rkskilltube_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U rkuser -d rkskilltube_db"]
      interval: 5s
      timeout: 5s
      retries: 5

  # In-Memory Cache, Rate Limiter & Message Broker
  redis:
    image: redis:7.2-alpine
    container_name: rkskilltube-redis
    restart: always
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: ["redis-server", "--appendonly", "yes"]
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5

  # Celery Monitoring Dashboard (Flower)
  flower:
    image: mher/flower:latest
    container_name: rkskilltube-flower
    environment:
      - CELERY_BROKER_URL=redis://redis:6379/0
      - FLOWER_PORT=5555
    ports:
      - "5555:5555"
    depends_on:
      - redis

volumes:
  postgres_data:
  redis_data:
```

### 2.2 Start Containers:
```powershell
docker compose up -d
```
Verify status:
```powershell
docker compose ps
```

---

## Step 3: Backend Setup (Python, FastAPI & Celery)

### 3.1 Create and Activate Virtual Environment
Navigate to `backend/`:
```powershell
cd backend
python -m venv .venv
```

**Activate the virtual environment:**
- **Windows (PowerShell):**
  ```powershell
  .\.venv\Scripts\Activate.ps1
  ```
  *(If execution is disabled, run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` first).*
- **Linux / macOS:**
  ```bash
  source .venv/bin/activate
  ```

### 3.2 Install Backend Dependencies
Upgrade pip and install requirements:
```powershell
python -m pip install --upgrade pip
```

Ensure `backend/requirements.txt` contains:
```txt
# Core Framework & Server
fastapi==0.111.1
uvicorn[standard]==0.30.1
python-multipart==0.0.9

# Configuration & Validation
pydantic==2.7.4
pydantic-settings==2.3.4
python-dotenv==1.0.1

# Database (PostgreSQL + Async SQLAlchemy + Asyncpg + pgvector)
sqlalchemy[asyncio]==2.0.31
asyncpg==0.29.0
alembic==1.13.2
pgvector==0.3.1

# Authentication & Security
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
authlib==1.7.2
httpx==0.27.0
itsdangerous==2.2.0

# Background Tasks & Cache
celery[redis]==5.4.0
redis==5.0.7
flower==2.0.1

# Video Processing & Cloud Storage
ffmpeg-python==0.2.0
boto3==1.34.144
botocore==1.34.144
Pillow==10.4.0
aiofiles==23.2.1

# AI & Recommendations
google-generativeai==0.7.2
openai==1.35.13

# Utilities & Testing
slowapi==0.1.9
loguru==0.7.2
python-slugify==8.0.4
email-validator==2.3.0
pytest==8.2.2
pytest-asyncio==0.23.7
```

Install:
```powershell
pip install -r requirements.txt
```

### 3.3 Configure Backend Environment (`backend/.env`)
Create `backend/.env`:
```env
ENVIRONMENT=development
PORT=8000
DEBUG=True
API_V1_STR=/api/v1
SECRET_KEY=super_secret_jwt_signing_key_change_in_production_min_32_chars
ACCESS_TOKEN_EXPIRE_MINUTES=1440

# PostgreSQL Connection
DATABASE_URL=postgresql+asyncpg://rkuser:rkpassword123@localhost:5432/rkskilltube_db

# Redis & Celery
REDIS_URL=redis://localhost:6379/0
CELERY_BROKER_URL=redis://localhost:6379/1
CELERY_RESULT_BACKEND=redis://localhost:6379/2

# Cloud Storage (Google Cloud Storage / Cloudflare R2 / S3)
STORAGE_PROVIDER=gcs  # Options: 'gcs' (Google Cloud Storage), 'cloudflare_r2', 'aws_s3'
GCP_PROJECT_ID=your-gcp-project-id
GCS_BUCKET_NAME=rkskilltube-media
GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json

# (Optional Multi-Cloud S3 / R2 Keys)
AWS_ACCESS_KEY_ID=your_key_id
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_REGION=auto
AWS_S3_BUCKET_NAME=rkskilltube-media
CLOUDFLARE_R2_ENDPOINT_URL=https://<account_id>.r2.cloudflarestorage.com
CDN_DOMAIN=https://media.rkskilltube.com

# AI API Keys
GEMINI_API_KEY=your_gemini_api_key_here

# Frontend URL (For CORS)
FRONTEND_URL=http://localhost:3000
```

---

## Step 4: Frontend Setup (Next.js 14+ with TypeScript)

Next.js provides **first-class, built-in, out-of-the-box support for TypeScript**.

### 4.1 Initialize Frontend (If Starting Fresh)
From your repository root:
```powershell
npx create-next-app@latest frontend
```
When prompted by the wizard, choose:
```text
✔ Would you like to use TypeScript? … Yes       <-- First-class TypeScript
✔ Would you like to use ESLint? … Yes
✔ Would you like to use Tailwind CSS? … Yes
✔ Would you like your code inside a `src/` directory? … Yes
✔ Would you like to use App Router? (recommended) … Yes
✔ Would you like to use Turbopack for next dev? … Yes
✔ Would you like to customize the import alias (@/*)? … No (use default @/*)
```

Or navigate to your existing directory:
```powershell
cd frontend
```

### 4.2 Install Essential Frontend Packages
Install the video player, icons, state management, and utilities:
```powershell
# Video streaming player and icons
npm install hls.js lucide-react

# Global state management and API client
npm install zustand axios

# UI and styling utilities
npm install clsx tailwind-merge date-fns

# Development TypeScript types
npm install -D @types/node @types/react @types/react-dom typescript
```

### 4.3 Configure Frontend Environment (`frontend/.env.local`)
Create `frontend/.env.local`:
```env
# Backend API Base URL
NEXT_PUBLIC_API_URL=http://localhost:8000/api/v1

# CDN / Media URL for HLS video streaming
NEXT_PUBLIC_CDN_URL=https://media.rkskilltube.com

# Google OAuth Client ID
NEXT_PUBLIC_GOOGLE_CLIENT_ID=your_google_oauth_client_id.apps.googleusercontent.com

# Frontend Domain
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### 4.4 Example Typed Component (`src/components/video/VideoCard.tsx`)
```tsx
import React from 'react';

export interface VideoCardProps {
  id: string;
  title: string;
  creatorName: string;
  views: number;
  duration: string;
  thumbnailUrl: string;
}

export const VideoCard: React.FC<VideoCardProps> = ({
  id,
  title,
  creatorName,
  views,
  duration,
  thumbnailUrl,
}) => {
  return (
    <div className="rounded-xl overflow-hidden border border-gray-800 bg-gray-900 hover:scale-[1.02] transition">
      <div className="relative aspect-video bg-black">
        <img src={thumbnailUrl} alt={title} className="w-full h-full object-cover" />
        <span className="absolute bottom-2 right-2 bg-black/80 text-white text-xs px-2 py-0.5 rounded">
          {duration}
        </span>
      </div>
      <div className="p-3">
        <h3 className="font-semibold text-white line-clamp-2">{title}</h3>
        <p className="text-sm text-gray-400 mt-1">{creatorName}</p>
        <p className="text-xs text-gray-500">{views.toLocaleString()} views</p>
      </div>
    </div>
  );
};
```

---

## Step 5: Media Processing Setup (FFmpeg)

FFmpeg transcodes uploaded MP4/MOV videos into adaptive multi-bitrate HLS streams (`.m3u8` playlists and `.ts` video chunks) and extracts thumbnail previews.

### Installation by OS:
* **Windows (PowerShell):**
  ```powershell
  winget install Gyan.FFmpeg
  ```
  *(Restart PowerShell after installation).*
* **macOS:**
  ```bash
  brew install ffmpeg
  ```
* **Linux (Ubuntu/Debian):**
  ```bash
  sudo apt update && sudo apt install -y ffmpeg
  ```

### Verify:
```powershell
ffmpeg -version
ffprobe -version
```

---

## Step 6: Full-Stack Connectivity & CORS Verification

Next.js (running on port `3000`) and FastAPI (running on port `8000`) must communicate over HTTP with CORS enabled.

### 6.1 Enable CORS in FastAPI (`backend/app/main.py`):
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI(title="RKSkillTube API", version="1.0.0")

# Allow Next.js frontend origin
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/api/v1/health")
async def health_check():
    return {
        "status": "ok",
        "message": "FastAPI is running and connected successfully!",
        "version": "1.0.0"
    }
```

### 6.2 Fetching from Next.js (`frontend/src/app/page.tsx`):
```tsx
import React from 'react';

async function getBackendHealth() {
  try {
    const res = await fetch('http://localhost:8000/api/v1/health', {
      cache: 'no-store', // Always fresh on each load
    });
    if (!res.ok) throw new Error('Backend responded with error');
    return res.json();
  } catch (err) {
    return { status: 'offline', message: 'Backend is currently unreachable' };
  }
}

export default async function HomePage() {
  const health = await getBackendHealth();

  return (
    <main className="min-h-screen bg-gray-950 text-white p-8">
      <h1 className="text-3xl font-bold">🎓 RKSkillTube Platform</h1>
      <div className="mt-4 p-4 border border-gray-800 rounded-lg bg-gray-900 max-w-lg">
        <p className="text-sm text-gray-400">Frontend: <strong>Next.js 14+ (TypeScript)</strong></p>
        <p className="text-sm text-gray-400 mt-2">Backend Status: <strong className="text-green-400">{health.status}</strong></p>
        <p className="text-xs text-gray-500 mt-1">{health.message}</p>
      </div>
    </main>
  );
}
```

---

## Step 7: Framework & Package Lifecycle Management

A clear guide on how to uninstall packages, reset broken dependencies, and re-add packages in both ecosystems.

### A. Frontend (Node.js & npm)
* `package.json`: Your master list of dependencies.
* `package-lock.json`: Exact snapshot of sub-dependencies.
* `node_modules/`: The downloaded code.

#### 1. Uninstall a single package:
```bash
npm uninstall <package-name>
# Example: npm uninstall bootstrap
```
#### 2. Clean Reset (Safe Reinstall):
If packages conflict or show strange errors, delete the downloaded copies and reinstall fresh from `package.json`:
```powershell
# 1. Remove node_modules and lockfile
# 2. Reinstall cleanly:
npm install
```

---

### B. Backend (Python & pip)
* `.venv/`: The isolated virtual environment folder.
* `requirements.txt`: The pinned list of packages.

#### 1. Uninstall a single Python library:
```bash
pip uninstall <package-name>
# Example: pip uninstall fastapi
```
#### 2. Save your current packages:
```bash
pip freeze > requirements.txt
```
#### 3. Clean Reset (Fresh Environment):
```powershell
# 1. Deactivate current env
deactivate

# 2. Create fresh virtual environment
python -m venv .venv

# 3. Activate
.\.venv\Scripts\Activate.ps1

# 4. Re-install all packages
pip install -r requirements.txt
```

---

## Step 8: Running the Full Development Environment

To run the complete system locally, open **4 separate terminal windows**:

### Terminal 1: Infrastructure (Docker)
```powershell
docker compose up -d
```
* PostgreSQL: `localhost:5432`
* Redis: `localhost:6379`
* Flower: `http://localhost:5555`

### Terminal 2: FastAPI Backend Server
```powershell
cd backend
.\.venv\Scripts\Activate.ps1
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```
* Interactive Swagger Docs: **`http://localhost:8000/docs`**
* Alternative ReDoc: **`http://localhost:8000/redoc`**

### Terminal 3: Celery Video & Task Worker
```powershell
cd backend
.\.venv\Scripts\Activate.ps1
celery -A app.celery_worker.celery_app worker --loglevel=info --pool=solo
```
> [!NOTE]
> On Windows, always include `--pool=solo` because Windows does not support POSIX `fork()`.

### Terminal 4: Next.js Frontend
```powershell
cd frontend
npm run dev
```
* Web Application: **`http://localhost:3000`**

---

## Step 9: Verification Checklist & Health Endpoints

After launching all four terminals, verify every layer:

| Checkpoint | Target URL / Command | Expected Result |
| :--- | :--- | :--- |
| **Frontend UI** | `http://localhost:3000` | RKSkillTube home page loads with Next.js & TypeScript |
| **API Health Check** | `http://localhost:8000/api/v1/health` | `{"status": "ok", "message": "FastAPI is running..."}` |
| **Interactive API Docs** | `http://localhost:8000/docs` | Swagger UI listing all endpoints (Auth, Video, Course, AI) |
| **PostgreSQL Database** | `docker exec -it rkskilltube-postgres psql -U rkuser -d rkskilltube_db -c "\dt"` | Shows tables created by Alembic migrations |
| **Redis Cache** | `docker exec -it rkskilltube-redis redis-cli ping` | Returns `PONG` |
| **Celery Worker Monitor** | `http://localhost:5555` | Flower UI showing active workers ready for video transcoding |

---

## Step 10: Troubleshooting Common Issues

### Issue 1: `Activate.ps1 cannot be loaded because running scripts is disabled`
* **Fix (Windows):** Open PowerShell and run:
  ```powershell
  Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
  ```
  Then reactivate: `.\.venv\Scripts\Activate.ps1`.

### Issue 2: Celery Worker crashes with `ValueError: not enough values to unpack` on Windows
* **Fix:** Always specify `--pool=solo` on Windows:
  ```powershell
  celery -A app.celery_worker.celery_app worker --loglevel=info --pool=solo
  ```

### Issue 3: Next.js CORS Error when calling FastAPI
* **Fix:** Ensure `backend/app/main.py` includes `CORSMiddleware` with `allow_origins=["http://localhost:3000"]`.

### Issue 4: Port 5432 or 6379 Already in Use
* **Fix:** A local PostgreSQL or Redis instance is already running on Windows. Either stop the Windows service via `services.msc` or change the port mapping in `docker-compose.yml` (e.g. `"5433:5432"`).

---

## Step 11: Top Learning Resources & Official Guides

For comprehensive official documentation, interactive courses, and deep-dive blogs across all technologies in this project, refer to our central learning hub:

👉 **[Master Technology Resource Hub (`Docs/resource.md`)](./resource.md)**

* **Next.js Official Course:** [https://nextjs.org/learn](https://nextjs.org/learn)
* **TypeScript Handbook & Playground:** [https://www.typescriptlang.org/docs/](https://www.typescriptlang.org/docs/)
* **FastAPI Interactive Tutorial:** [https://fastapi.tiangolo.com/tutorial/](https://fastapi.tiangolo.com/tutorial/)
* **Pydantic v2 Documentation:** [https://docs.pydantic.dev/latest/](https://docs.pydantic.dev/latest/)
* **HLS.js Documentation & Demo:** [https://github.com/video-dev/hls.js](https://github.com/video-dev/hls.js)
* **Docker Documentation:** [https://docs.docker.com/get-started/](https://docs.docker.com/get-started/)
