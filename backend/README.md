# ⚙️ RKSkillTube — Backend API & Worker Engine

<div align="center">

[![FastAPI](https://img.shields.io/badge/Framework-FastAPI%20ASGI-009688?style=for-the-badge&logo=fastapi)](https://fastapi.tiangolo.com/)
[![Python](https://img.shields.io/badge/Python-3.11%20%7C%203.12-3776AB?style=for-the-badge&logo=python)](https://www.python.org/)
[![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL%2016-336791?style=for-the-badge&logo=postgresql)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Cache%20%26%20Broker-Redis%207-DC382D?style=for-the-badge&logo=redis)](https://redis.io/)
[![Celery](https://img.shields.io/badge/Tasks-Celery%20Workers-37814A?style=for-the-badge&logo=celery)](https://docs.celeryq.dev/)

**The asynchronous core engine of RKSkillTube, providing RESTful APIs, video transcoding pipelines, semantic vector search, and authentication.**

</div>

---

## 📖 Architecture & Design

The backend is built following **Clean Architecture (Layered Architecture)** principles to ensure high maintainability, testability, and separation of concerns:

```
backend/app/
├── api/                           # Presentation Layer (HTTP controllers & routing)
│   ├── deps.py                    # FastAPI Dependency Injection (DB sessions, auth guards)
│   └── v1/
│       ├── router.py              # Main API v1 routing registry
│       └── endpoints/             # Subsystem controllers (auth, videos, courses, ai, etc.)
├── core/                          # Security, JWT utilities, database session engine
├── models/                        # Persistence Layer (SQLAlchemy ORM declarative models)
├── repositories/                  # Data Access Layer (clean database queries & mutations)
├── schemas/                       # Pydantic v2 schemas for request validation & DTOs
├── services/                      # Domain / Business Logic Layer
├── tasks/                         # Celery background workers (FFmpeg transcoding, email)
├── utils/                         # Helper utilities (FFmpeg wrappers, text formatters)
├── alembic/                       # Alembic database schema migrations
├── celery_worker.py               # Celery application entrypoint
├── config.py                      # Pydantic Settings application configuration
└── main.py                        # FastAPI application entrypoint and middleware setup
```

---

## ⚡ Local Setup & Execution

### 1. Prerequisites
- **Python 3.11+**
- **Docker Desktop** (or running instances of PostgreSQL 16 & Redis 7)
- **FFmpeg** (installed on host system for video transcoding)

### 2. Create Virtual Environment
```bash
cd backend
python -m venv .venv

# Activate:
# Windows (PowerShell):
.\.venv\Scripts\Activate.ps1
# macOS / Linux:
source .venv/bin/activate
```

### 3. Install Python Dependencies
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### 4. Configure Environment Variables
Copy `.env.example` to `.env`:
```bash
cp .env.example .env
```
Ensure your database, Redis, and secret keys are configured:
```env
# Application Settings
PROJECT_NAME="RKSkillTube API"
ENVIRONMENT="development"
SECRET_KEY="replace-with-a-secure-random-secret-key"
ACCESS_TOKEN_EXPIRE_MINUTES=60

# Database & Cache
DATABASE_URL=postgresql+asyncpg://postgres:postgres@localhost:5432/rkskilltube
REDIS_URL=redis://localhost:6379/0

# Object Storage / CDN
STORAGE_PROVIDER="gcs" # or "r2" / "s3"
GCS_BUCKET_NAME="rkskilltube-media-dev"

# Celery Task Queue
CELERY_BROKER_URL=redis://localhost:6379/1
CELERY_RESULT_BACKEND=redis://localhost:6379/2
```

### 5. Run Database Migrations (Alembic)
```bash
alembic upgrade head
```

### 6. Start FastAPI Development Server
```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```
- Interactive Swagger UI: [http://localhost:8000/docs](http://localhost:8000/docs)
- Interactive ReDoc: [http://localhost:8000/redoc](http://localhost:8000/redoc)

### 7. Run Celery Background Worker
Open another terminal (with `.venv` activated):
```bash
# Windows:
celery -A app.celery_worker.celery_app worker --loglevel=info --pool=solo

# Linux / macOS:
celery -A app.celery_worker.celery_app worker --loglevel=info -c 4
```

---

## 🧪 Running Automated Tests

All tests reside under `backend/tests/`:

```bash
# Run all unit tests:
pytest tests/unit

# Run integration tests:
pytest tests/integration

# Run tests with coverage report:
pytest --cov=app tests/
```

---

## 🤝 Backend Contributing Rules

1. **Layer Separation:** Never write raw SQL or complex database joins inside `api/endpoints/`. Always encapsulate database queries within `repositories/` and business validation within `services/`.
2. **Schema Validation:** Always accept and return typed Pydantic models from `schemas/`. Do not pass arbitrary dictionaries to or from endpoints.
3. **Async / Await:** All endpoints and database operations must use non-blocking `async def` and `await`.
4. **Task Offloading:** Any CPU-heavy task (transcoding, PDF certificate generation, email dispatch) must be delegated to Celery via `.delay()` and never executed on the main API request thread.
