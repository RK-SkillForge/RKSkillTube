# 📚 RKSkillTube — Master Technology Learning & Resource Hub (`resource.md`)

> **Welcome to the Central Learning & Resource Hub for RKSkillTube!**  
> Whether you are touching these technologies for the first time or looking for deep-dive production references, this document aggregates the **highest-quality official documentation**, **interactive courses**, **curated blogs**, **architecture blueprints**, and **video guides** across the entire project stack.
>
> 💡 *Feel free to bookmark this file and contribute new resources as you discover them!*

---

## 📑 Table of Contents
1. [Frontend Core (Next.js, TypeScript, React)](#1-frontend-core)
2. [UI & Styling (Tailwind CSS, Icons, Primitives)](#2-ui--styling)
3. [Video Streaming & Playback (HLS, Media Players, WebVTT)](#3-video-streaming--playback)
4. [State Management & Data Fetching (Zustand, Axios, TanStack Query)](#4-state-management--data-fetching)
5. [Backend Core (FastAPI, Python, Pydantic)](#5-backend-core)
6. [Databases & Persistence (PostgreSQL, pgvector, SQLAlchemy, Alembic)](#6-databases--persistence)
7. [Caching & Task Queues (Redis, Celery, Flower)](#7-caching--task-queues)
8. [Media Processing Pipeline (FFmpeg)](#8-media-processing-pipeline)
9. [Object Storage & CDN (Cloudflare R2, AWS S3, CloudFront)](#9-object-storage--cdn)
10. [Authentication & Security (NextAuth, JWT, OAuth 2.0)](#10-authentication--security)
11. [DevOps & Containers (Docker, Docker Compose, CI/CD)](#11-devops--containers)
12. [AI & Semantic Search (Gemini API, LangChain, Vector Embeddings)](#12-ai--semantic-search)
13. [How to Add New Resources to this File](#13-how-to-add-new-resources-to-this-file)

---

## 1. Frontend Core

### ⚡ Next.js 14+ (App Router)
* 📘 **[Next.js Official Learn Course (Interactive)](https://nextjs.org/learn)** — **Start here!** Free interactive, zero-setup course teaching App Router, layouts, server components, and data fetching.
* 📖 **[Official Next.js Documentation](https://nextjs.org/docs)** — The definitive reference for App Router routing, caching, server actions, and optimization.
* ✍️ **[Lee Robinson's Blog (VP of Product @ Vercel)](https://leerob.io/blog)** — World-class architectural articles explaining Server Components, caching strategies, and performance patterns.
* 🐙 **[Next.js Official Examples Repository](https://github.com/vercel/next.js/tree/canary/examples)** — Over 100+ production-ready code examples covering auth, caching, and API integration.

### 🔷 TypeScript
* 📘 **[Total TypeScript by Matt Pocock](https://www.totaltypescript.com/tutorials)** — The best hands-on, beginner-friendly interactive TypeScript tutorial available online.
* 📖 **[Official TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)** — Official guide explaining types, interfaces, generics, and type inference.
* 🛠️ **[TypeScript Playground](https://www.typescriptlang.org/play)** — Browser-based scratchpad to test TypeScript code and check compiled JavaScript.
*  cheatsheet **[React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)** — Practical, copy-pasteable guide for typing React props, hooks, and events.

### ⚛️ React 18+
* 📖 **[React Official Documentation (react.dev)](https://react.dev/learn)** — Interactive modern guide covering components, props, `useState`, `useEffect`, and component lifecycles.
* 🎥 **[Web Dev Simplified (YouTube)](https://www.youtube.com/@WebDevSimplified)** — Clear 10-minute visual breakdowns of core React hooks and design patterns.

---

## 2. UI & Styling

### 🎨 Tailwind CSS
* 📖 **[Tailwind CSS Official Documentation](https://tailwindcss.com/docs)** — Fast, searchable reference for every utility class (flexbox, grid, animations, colors).
* 🛠️ **[Tailwind Play](https://play.tailwindcss.com/)** — Interactive browser sandbox to experiment with Tailwind classes in real time.
* 🧩 **[Tailwind Merge (`tailwind-merge`) & Clsx](https://github.com/dcastil/tailwind-merge)** — Standard utilities for merging dynamic Tailwind classes cleanly without conflicts.

### 🎭 UI Primitives & Icons
* 📖 **[Radix UI Primitives](https://www.radix-ui.com/primitives/docs/overview/introduction)** — Unstyled, accessible UI primitives (dialogs, dropdowns, tooltips, tabs) with full keyboard navigation.
* 📖 **[shadcn/ui Documentation](https://ui.shadcn.com/)** — Beautifully designed, accessible components built on Radix and Tailwind that you can copy into your project.
* 🎨 **[Lucide Icons](https://lucide.dev/icons/)** — Clean, modern icon set optimized for React and Next.js.

---

## 3. Video Streaming & Playback

### 📺 HLS Streaming & Video Players
* 📖 **[HLS.js Official Documentation](https://github.com/video-dev/hls.js)** — JavaScript library implementing HTTP Live Streaming (HLS) client using HTML5 Video and MediaSource Extensions (MSE).
* 🎮 **[HLS.js Interactive Demo Player](https://hls-js.netlify.app/demo/)** — Test HLS streams, switch bitrates, view buffer graphs, and inspect live manifests.
* 📖 **[Video.js Documentation](https://videojs.com/)** — Battle-tested open-source HTML5 video player framework.
* 📖 **[MDN Web Docs: Media Source Extensions (MSE)](https://developer.mozilla.org/en-US/docs/Web/API/Media_Source_Extensions_API)** — Understanding how modern browsers stream video chunks dynamically.

### 🎞️ Subtitles & Timeline Scrubbing
* 📖 **[MDN Web Docs: WebVTT Subtitles Format](https://developer.mozilla.org/en-US/docs/Web/API/WebVTT_API)** — Guide to writing and parsing `.vtt` caption files and storyboard image sprites.
* ✍️ **[Mux Guide: Video Scrubbing Thumbnails](https://www.mux.com/blog/how-to-generate-storyboards-for-video-scrubbing)** — Explains how YouTube and Netflix generate thumbnail sprites for scrubber hover previews.

---

## 4. State Management & Data Fetching

### 🐻 Zustand (Global State)
* 📖 **[Zustand Official Documentation](https://zustand.docs.pmnd.rs/getting-started/introduction)** — Small, fast, scalable bearbones state-management solution using simplified flux principles.
* ✍️ **[TkDodo's Blog: Working with Zustand](https://tkdodo.eu/blog/working-with-zustand)** — Best practices for structuring Zustand stores with TypeScript.

### 🔄 Data Fetching & HTTP
* 📖 **[TanStack Query (React Query) Docs](https://tanstack.com/query/latest/docs/framework/react/overview)** — Powerful asynchronous state management for server data, caching, and auto-refetching.
* 📖 **[Axios Official Documentation](https://axios-http.com/docs/intro)** — Promise-based HTTP client for the browser and node.js with request/response interceptors.

---

## 5. Backend Core

### 🚀 FastAPI & ASGI
* 📘 **[FastAPI Official Interactive Tutorial](https://fastapi.tiangolo.com/tutorial/)** — Widely regarded as one of the best interactive documentation guides in open source.
* 📖 **[Starlette Documentation](https://www.starlette.io/)** — The lightweight ASGI framework that powers FastAPI's speed and HTTP routing.
* 📖 **[Uvicorn Documentation](https://www.uvicorn.org/)** — Lightning-fast ASGI web server implementation for Python.
* 🐙 **[Full-Stack FastAPI Template (tiangolo)](https://github.com/fastapi/full-stack-fastapi-template)** — Tiangolo's official production boilerplate with Docker, Celery, and Auth.

### 🛡️ Pydantic v2 (Data Validation)
* 📖 **[Pydantic v2 Documentation](https://docs.pydantic.dev/latest/)** — Data validation and settings management using Python type annotations.
* ✍️ **[FastAPI + Pydantic v2 Migration & Best Practices](https://realpython.com/fastapi-python-web-apis/)** — Real Python guide to writing clean, typed REST endpoints.

---

## 6. Databases & Persistence

### 🐘 PostgreSQL 16 & pgvector
* 📖 **[PostgreSQL 16 Official Manual](https://www.postgresql.org/docs/16/index.html)** — Comprehensive reference for relational data, indexing, and transactions.
* 🐙 **[pgvector Official Repository & Guide](https://github.com/pgvector/pgvector)** — Open-source vector similarity search extension for PostgreSQL (essential for AI course recommendations).
* ✍️ **[Supabase Guide: Vector Embeddings in Postgres](https://supabase.com/docs/guides/ai/vector-columns)** — Great conceptual guide on cosine distance, inner product, and vector indexing (HNSW & IVFFlat).

### 🗄️ ORMs & Migrations
* 📖 **[SQLAlchemy 2.0 Unified Documentation](https://docs.sqlalchemy.org/en/20/)** — Python SQL toolkit and Object Relational Mapper with native async support (`asyncpg`).
* 📖 **[SQLModel Documentation](https://sqlmodel.tiangolo.com/)** — Library designed by Tiangolo to combine SQLAlchemy and Pydantic into a single model definition.
* 📖 **[Alembic Migration Guide](https://alembic.sqlalchemy.org/en/latest/)** — Database schema migration tool for SQLAlchemy.

---

## 7. Caching & Task Queues

### ⚡ Redis 7
* 📖 **[Redis Official Documentation](https://redis.io/docs/)** — In-memory data store used for fast caching, rate limiting, and message brokering.
* 📘 **[Redis University (Free Courses)](https://university.redis.com/)** — Free courses on data structures, caching patterns, and Redis pub/sub.

### 🌾 Celery (Distributed Processing)
* 📖 **[Celery Official User Guide](https://docs.celeryq.dev/en/stable/)** — Asynchronous task queue/job queue based on distributed message passing.
* 📖 **[Flower Monitoring Dashboard](https://flower.readthedocs.io/en/latest/)** — Real-time web dashboard for Celery tasks, workers, and rate limits.
* ✍️ **[TestDriven.io: Asynchronous Tasks with FastAPI and Celery](https://testdriven.io/blog/fastapi-and-celery/)** — Complete walkthrough of connecting FastAPI, Celery, and Redis with Docker.

---

## 8. Media Processing Pipeline

### 🎬 FFmpeg
* 📖 **[FFmpeg Official Documentation](https://ffmpeg.org/documentation.html)** — Complete reference manual for audio and video transcoding.
* ✍️ **[HLS Packaging with FFmpeg Guide](https://ottverse.com/hls-packaging-using-ffmpeg/)** — Step-by-step tutorial on generating multi-bitrate HLS streams (`1080p`, `720p`, `360p`) and master `.m3u8` playlists.
* ✍️ **[Generating Video Storyboard Sprites with FFmpeg](https://ffmpeg.org/ffmpeg-filters.html#tile-1)** — How to extract video frames every $N$ seconds and pack them into a tile sprite image for timeline scrubbing.

---

## 9. Object Storage & CDN

### ☁️ Cloudflare R2 & AWS S3
* 📖 **[Cloudflare R2 Documentation](https://developers.cloudflare.com/r2/)** — S3-compatible zero-egress fee object storage.
* 📖 **[AWS Boto3 (Python SDK) S3 Presigned URLs Guide](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/s3-presigned-urls.html)** — How to safely generate presigned PUT and GET URLs for direct client uploads.
* 📖 **[CloudFront CDN Documentation](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)** — Global Edge CDN caching for low-latency video streaming.

---

## 10. Authentication & Security

### 🔐 Auth & Tokens
* 📖 **[NextAuth.js (Auth.js) Documentation](https://authjs.dev/)** — Complete authentication for Next.js applications (Google OAuth, Credentials, Session management).
* 📖 **[PyJWT Documentation](https://pyjwt.readthedocs.io/en/latest/)** — JSON Web Token implementation in Python for stateless backend authorization.
* 📖 **[Passlib / Bcrypt Guide](https://passlib.readthedocs.io/en/stable/)** — Secure password hashing and salt generation best practices.
* 📖 **[OWASP Top 10 Security Risks](https://owasp.org/www-project-top-ten/)** — Standard security checklist for web APIs (CORS, CSRF, Injection, Broken Object Level Auth).

---

## 11. DevOps & Containers

### 🐳 Docker & Docker Compose
* 📘 **[Docker Getting Started Tutorial](https://docs.docker.com/get-started/)** — Understand containers, images, volumes, and networking.
* 📖 **[Docker Compose Specification](https://docs.docker.com/compose/)** — Multi-container application orchestration reference.
* ✍️ **[Best Practices for Containerizing Next.js & Python Applications](https://docs.docker.com/guides/docker-concepts/building-images/build-best-practices/)** — Multi-stage builds and minimal image sizes.

---

## 12. AI & Semantic Search

### 🤖 Gemini API, LangChain & Vector Embeddings
* 📖 **[Google AI for Developers (Gemini API Docs)](https://ai.google.dev/docs)** — Official guides and SDK reference for multimodal prompts and chat assistants.
* 📖 **[LangChain Python Documentation](https://python.langchain.com/docs/get_started/introduction)** — Framework for developing applications powered by language models and vector retrievers.
* ✍️ **[OpenAI Cookbook: Vector Search with Embeddings](https://cookbook.openai.com/)** — Great conceptual patterns for semantic course search and RAG (Retrieval Augmented Generation).

---

## 13. How to Add New Resources to this File

When you find a great blog post, YouTube video, or documentation page that helped you solve a problem in RKSkillTube, please add it here!

### Guidelines for Adding Resources:
1. **Prefer Official & Updated Material**: Avoid tutorials older than 2 years if they reference outdated syntax (e.g. Next.js Pages Router or Pydantic v1).
2. **Follow the Markdown Format**:
   ```markdown
   * 📘/📖/✍️/🎥 **[Resource Title](URL)** — One-sentence explanation of what makes this resource valuable.
   ```
3. **Place in the Relevant Category**: Keep the document organized under the existing headings.

---

> [!TIP]
> To see how to install the stack locally, visit [`Docs/INSTALLATION_GUIDE.md`](./INSTALLATION_GUIDE.md).  
> To see the folder architecture, visit [`Docs/folder.md`](./folder.md).
