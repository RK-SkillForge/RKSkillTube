# 🎨 RKSkillTube — Frontend Web Client

<div align="center">

[![Next.js](https://img.shields.io/badge/Next.js-14%2B%20App%20Router-black?style=for-the-badge&logo=next.js)](https://nextjs.org/)
[![React](https://img.shields.io/badge/React-18%2F19-61DAFB?style=for-the-badge&logo=react)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/Language-TypeScript-blue?style=for-the-badge&logo=typescript)](https://www.typescriptlang.org/)
[![Tailwind CSS](https://img.shields.io/badge/Styling-Tailwind%20CSS-38B2AC?style=for-the-badge&logo=tailwind-css)](https://tailwindcss.com/)

**The user-facing web application and creator studio for RKSkillTube, built with Next.js App Router, TypeScript, and HLS.js streaming.**

</div>

---

## 📖 Overview

The **RKSkillTube Frontend** delivers a responsive, zero-buffering educational experience. It integrates custom adaptive bitrate (ABR) video playback with YouTube-style scrubber previews, modular course curriculums, creator analytics dashboards, and an AI chat assistant.

### 🌟 Key Features
* **Adaptive Video Streaming:** HLS.js video player supporting dynamic resolution switching (`360p` to `1080p`), speed control, and WebVTT storyboard hover scrubbers.
* **Route Groups & Architecture:** Modular Next.js App Router architecture separating public browsing, student learning, creator studio, and administrative portals.
* **Global State Management:** Lightweight reactive stores using Zustand for authentication session state and playback tracking.
* **Type Safety:** Full compile-time safety with TypeScript interfaces mirroring backend Pydantic models.

---

## 📁 Directory Architecture

```
frontend/src/
├── app/                           # Next.js App Router routes & layouts
│   ├── (auth)/                    # Authentication routes (/login, /register)
│   ├── (creator)/                 # Creator Studio portal (/studio, /studio/upload)
│   ├── (admin)/                   # Admin moderation & platform analytics
│   ├── (main)/                    # Public routes (feed, watch, courses, channel)
│   │   ├── watch/[videoId]/       # Video player page with comments & sidebar
│   │   ├── courses/               # Course catalog & syllabus viewer
│   │   ├── channel/[handle]/      # Creator channel pages
│   │   └── search/                # Semantic AI video & course search
│   ├── layout.tsx                 # Root application shell
│   └── globals.css                # Global Tailwind CSS definitions
├── components/                    # Reusable React UI components
│   ├── HlsPlayer.tsx              # Adaptive HLS video player engine
│   ├── VideoCard.tsx              # Video thumbnail card with preview
│   ├── ScrubberPreview.tsx        # Timeline thumbnail hover scrub previews
│   ├── ChatWindow.tsx             # Floating AI Chatbot Assistant widget
│   └── Navbar.tsx                 # Global responsive navigation header
├── hooks/                         # Custom React hooks
│   ├── useHlsPlayer.ts            # HLS streaming lifecycle and quality levels
│   ├── useWatchProgress.ts        # Syncs user video watch time to backend
│   └── useDebounce.ts             # Input debouncing for search
├── services/                      # API service clients (Axios / Fetch)
│   ├── api.ts                     # Central HTTP client with JWT interceptors
│   ├── videoService.ts            # Video endpoints & presigned upload calls
│   └── courseService.ts           # Course catalog & lesson progress API
├── store/                         # Zustand global state stores
│   ├── useAuthStore.ts            # User identity, JWT tokens, and login state
│   └── usePlayerStore.ts          # Playback position, volume, and playback speed
└── types/                         # TypeScript interfaces and DTO definitions
    ├── video.ts                   # Video metadata and HLS manifest contracts
    ├── course.ts                  # Course, chapter, and lesson models
    └── user.ts                    # User profile and role permissions
```

---

## ⚡ Getting Started Locally

### 1. Prerequisites
- **Node.js (v20+ LTS recommended)**
- **npm** (or **pnpm** / **yarn**)

### 2. Configure Environment Variables
Create a `.env.local` file in `frontend/`:
```bash
cp .env.example .env.local
```

Populate the variables:
```env
# Backend API Base URL
NEXT_PUBLIC_API_URL=http://localhost:8000/api/v1

# CDN / Object Storage URL for HLS streams and thumbnails
NEXT_PUBLIC_CDN_URL=http://localhost:9000

# Optional: Google OAuth Client ID for social sign-in
NEXT_PUBLIC_GOOGLE_CLIENT_ID=your-google-client-id
```

### 3. Install Dependencies
```bash
npm install
```

### 4. Run Development Server
```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## 🛠️ Available Scripts

| Command | Purpose |
| :--- | :--- |
| `npm run dev` | Starts local Next.js development server with hot module reload on port `3000`. |
| `npm run build` | Compiles and builds production-optimized standalone bundle. |
| `npm run start` | Starts the production server using the compiled `.next` build. |
| `npm run lint` | Runs ESLint to check for code quality and style errors. |

---

## 🤝 Contributing Guidelines for Frontend

1. **Server vs. Client Components:** Prefer React Server Components (`RSC`) for SEO and instant initial page load. Only mark components with `'use client'` when using React hooks (`useState`, `useEffect`), event handlers, or browser APIs.
2. **Type Safety:** Always import or define types from `src/types/`. Avoid `any`.
3. **Styling:** Follow Tailwind CSS utility patterns. Keep components responsive across mobile, tablet, and desktop viewports.
4. **Video Playback:** When modifying `HlsPlayer.tsx`, verify buffer handling and memory teardown (`hls.destroy()`) on unmount.
