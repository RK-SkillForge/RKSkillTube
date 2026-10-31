# 🤝 Contributing to RKSkillTube

Thank you for your interest in contributing to **RKSkillTube**! 🚀

RKSkillTube is a modern, high-performance video streaming and EdTech platform built to empower engineering students, self-taught learners, and educators—particularly bridging the resource gap for Tier-2 & Tier-3 college communities. The platform integrates adaptive HLS video streaming, modular courses, creator channels, community memberships, and AI-assisted learning.

Whether you are fixing a typo in documentation, squashing a bug, improving test coverage, or implementing a new feature, **your contributions are deeply appreciated!**

---

## 📜 Table of Contents
1. [Code of Conduct](#-code-of-conduct)
2. [How Can I Contribute?](#-how-can-i-contribute)
3. [Finding Your First Issue](#-finding-your-first-issue)
4. [Local Development Setup](#-local-development-setup)
5. [Git Workflow & Branching Strategy](#-git-workflow--branching-strategy)
6. [Commit Message Guidelines](#-commit-message-guidelines)
7. [Coding Standards & Architecture](#-coding-standards--architecture)
8. [Testing & Quality Assurance](#-testing--quality-assurance)
9. [Submitting a Pull Request](#-submitting-a-pull-request)
10. [Community & Getting Help](#-community--getting-help)

---

## 🛡️ Code of Conduct

All contributors and maintainers are expected to adhere to our [Code of Conduct](./CODE_OF_CONDUCT.md). Please treat all community members with kindness, empathy, and respect. Harassment or unprofessional behavior will not be tolerated.

---

## 💡 How Can I Contribute?

You can contribute to RKSkillTube in multiple ways:

* 🐛 **Reporting Bugs:** File detailed bug reports using our [Bug Report Template](.github/ISSUE_TEMPLATE/bug_report.md).
* ✨ **Suggesting Features:** Propose ideas using our [Feature Request Template](.github/ISSUE_TEMPLATE/feature_request.md).
* 📖 **Improving Documentation:** Fix typos, add explanations, or write guides in `Docs/` or READMEs.
* 💻 **Writing Code:** Implement feature requests, fix reported bugs, or optimize database queries and media pipelines.
* 🧪 **Writing Tests:** Enhance unit and integration tests across frontend (`tests/`) and backend (`backend/tests/`).
* 🎨 **UI / UX Design:** Enhance user interfaces, accessibility (a11y), and responsiveness.

---

## 🎯 Finding Your First Issue

If you are new to the project or open source in general:
1. Browse issues tagged with [**`good first issue`**](https://github.com/orgs/RKSkillTube/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22) for small, beginner-friendly tasks.
2. Check [**`help wanted`**](https://github.com/orgs/RKSkillTube/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22) for prioritized community tasks.
3. **Claiming an Issue:** Before starting work, please leave a comment on the issue saying: *"I would like to work on this issue."* Wait for a maintainer to assign it to you to prevent duplicate efforts.

---

## ⚡ Local Development Setup

For an exhaustive guide across Windows, macOS, and Linux, refer to the [Installation Guide](./Docs/INSTALLATION_GUIDE.md). Here is the quick setup:

### Prerequisites
- **Git**
- **Docker Desktop** (for PostgreSQL, Redis, Flower)
- **Node.js (v20+ LTS)** & **npm**
- **Python (3.11+)**
- **FFmpeg** (installed and added to your system PATH)

### 1. Fork & Clone Repository
```bash
# 1. Fork the repo on GitHub, then clone your fork:
git clone https://github.com/<your-username>/RKSkillTube.git
cd RKSkillTube

# 2. Add the upstream remote:
git remote add upstream https://github.com/rkskillforge/RKSkillTube.git
```

### 2. Boot Local Infrastructure (Docker)
In your first terminal:
```bash
docker compose up -d
```
This spins up:
- **PostgreSQL 16** with `pgvector` on port `5432`
- **Redis 7** on port `6379`
- **Flower Dashboard** (Celery monitor) on `http://localhost:5555`

### 3. Setup Backend (FastAPI)
In a second terminal:
```bash
cd backend
python -m venv .venv

# Activate virtual environment:
# Windows (PowerShell):
.\.venv\Scripts\Activate.ps1
# macOS / Linux:
source .venv/bin/activate

# Install dependencies:
pip install -r requirements.txt

# Copy environment variables:
cp .env.example .env

# Run FastAPI dev server:
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```
- Swagger API Docs: `http://localhost:8000/docs`
- ReDoc Docs: `http://localhost:8000/redoc`

### 4. Setup Celery Worker (Transcoding & Tasks)
In a third terminal:
```bash
cd backend
# Activate your .venv first
celery -A app.celery_worker.celery_app worker --loglevel=info --pool=solo
```

### 5. Setup Frontend (Next.js 14)
In a fourth terminal:
```bash
cd frontend
npm install

# Copy environment variables:
cp .env.example .env.local

# Run Next.js dev server:
npm run dev
```
- Frontend Web App: `http://localhost:3000`

---

## 🌿 Git Workflow & Branching Strategy

Always create a new branch from an up-to-date `main` branch:

```bash
# 1. Fetch latest changes from upstream:
git checkout main
git pull upstream main

# 2. Create a descriptive feature/fix branch:
git checkout -b feat/user-authentication-jwt
# Or for a bugfix:
git checkout -b fix/hls-player-buffering
```

### Branch Naming Conventions
- `feat/<short-description>`: New features or capabilities
- `fix/<short-description>`: Bug fixes and regressions
- `docs/<short-description>`: Documentation additions or updates
- `refactor/<short-description>`: Code restructuring without functional changes
- `perf/<short-description>`: Performance and latency optimizations
- `test/<short-description>`: Adding or improving test suites

---

## 📝 Commit Message Guidelines

We follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```text
<type>(<scope>): <short imperative description>

[optional body describing the motivation and context]

[optional footer referencing issue numbers: Closes #123]
```

### Common Types:
- `feat`: A new feature (e.g. `feat(auth): implement Google OAuth 2.0 flow`)
- `fix`: A bug fix (e.g. `fix(player): resolve video scrubber timeline offset`)
- `docs`: Documentation only (e.g. `docs(readme): update backend installation steps`)
- `refactor`: Code change that neither fixes a bug nor adds a feature (e.g. `refactor(db): abstract video queries into VideoRepository`)
- `perf`: A code change that improves performance (e.g. `perf(cache): add Redis caching for popular course metadata`)
- `test`: Adding or correcting tests (e.g. `test(auth): add unit tests for JWT token generation`)
- `chore`: Maintenance tasks, dependencies, or configuration (e.g. `chore(deps): update Next.js to 14.2`)

---

## 🏗️ Coding Standards & Architecture

### Backend (FastAPI / Python)
- **Clean Architecture:** Strictly follow the 4-layer design:
  - `app/api/v1/endpoints/`: Routing and HTTP request handling only.
  - `app/services/`: Business logic, domain orchestration.
  - `app/repositories/`: Direct database queries and ORM operations.
  - `app/models/`: SQLAlchemy ORM database models.
- **Type Annotations:** Always annotate function parameters and return types.
- **Pydantic Schemas:** Use Pydantic v2 schemas in `app/schemas/` for all request validation and response serialization.
- **Asynchronous Code:** Use `async def` for I/O-bound operations and dependency injections (`Depends()`).

### Frontend (Next.js 14 / TypeScript / React)
- **App Router:** Follow Next.js 14 App Router patterns (`app/`).
- **Server Components First:** Keep components as React Server Components (RSC) by default. Add `'use client'` only when state, event handlers, or browser APIs (`HLS.js`, `localStorage`) are needed.
- **Type Safety:** Define TypeScript interfaces/types in `frontend/src/types/`. Avoid `any`.
- **Styling:** Use Tailwind CSS utility classes. Avoid inline style objects.
- **State Management:** Use Zustand for global client-side state.

### Security Best Practices
- Never commit secrets, passwords, database credentials, or private API keys.
- Always use `.env` files (which are ignored by `.gitignore`).
- Sanitize and validate all user inputs.

---

## 🧪 Testing & Quality Assurance

Before submitting a pull request, verify your changes pass all linters and tests:

### Frontend Checks
```bash
cd frontend
npm run lint       # ESLint check
npm run build      # Verify TypeScript builds cleanly
```

### Backend Checks
```bash
cd backend
# Activate virtual environment
pytest tests/unit/           # Run unit tests
pytest tests/integration/    # Run integration tests
```

---

## 🚀 Submitting a Pull Request

1. **Push your branch to your GitHub fork:**
   ```bash
   git push origin feat/user-authentication-jwt
   ```
2. **Open a Pull Request:**
   - Go to your fork on GitHub and click **Compare & pull request**.
   - Select the base repository `rkskillforge/RKSkillTube` and base branch `main`.
3. **Fill out the Pull Request Template:**
   - Link the relevant issue (e.g., `Closes #42`).
   - Describe the changes clearly and concisely.
   - Attach screenshots or screen recordings for UI changes.
   - Check all items in the Contributor Checklist.
4. **Participate in Code Review:**
   - Maintainers will review your PR. Be open to feedback and suggestions.
   - Push additional commits to your branch to address review comments; the PR will update automatically.

---

## 💬 Community & Getting Help

Need help or want to discuss architecture?
- 💬 **GitHub Discussions:** Join the conversations in [Discussions](https://github.com/orgs/RKSkillTube/discussions).
- 📧 **Maintainer Contact:** Reach out at `rkskillforge@gmail.com`.
- 📚 **Documentation Hub:** Consult [`Docs/README.md`](./Docs/README.md).

Thank you for helping make **RKSkillTube** an accessible, high-quality learning platform for learners worldwide! 🌟
