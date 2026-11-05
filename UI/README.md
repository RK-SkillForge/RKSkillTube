# 🎓 RKSkillTube UI Design System & Prototype Architecture

> **Welcome to the RKSkillTube UI Hub!**  
> This directory contains the complete set of high-fidelity, feature-driven UI prototypes, design system tokens, and responsive frontend screens for **RKSkillTube** — a next-generation hybrid video streaming and course learning platform built for engineering students, creators, and platform administrators.

---

## 🔗 Official Project Links & External Resources

| Resource | URL Link | Description |
| :--- | :--- | :--- |
| **Google Stitch Platform** | [stitch.withgoogle.com](https://stitch.withgoogle.com/) | The AI-assisted UI design and rapid prototyping engine used to generate the layouts. |
| **RKSkillTube Stitch Project** | [stitch.withgoogle.com/projects/1557866825418234142](https://stitch.withgoogle.com/projects/1557866825418234142) | The live Google Stitch cloud project containing the complete set of interactive prototypes, artboards, and exports. |
| **Frontend Codebase** | [`frontend/src`](../frontend/src) | The production Next.js 14+ (App Router) TypeScript application where these prototypes are integrated. |
| **Feature Roadmap** | [`Feature-Based-Guide`](../Feature-Based-Guide) | Complete functional breakdown, API specifications, and architectural documentation. |

---

## 🎨 Global Design System: "Obsidian Kinetic"

All screens strictly follow the **Obsidian Kinetic** dark-mode design specification located at [`UI/00-design-system/obsidian_kinetic/DESIGN.md`](./00-design-system/obsidian_kinetic/DESIGN.md).

* **Background Surfaces:** Deep Obsidian & Slate (`#080B10`, `#0B0F17`, `#0F172A`).
* **Card Containers:** Frosted glassmorphism (`#111827 / 75% opacity`) with subtle 1px border strokes (`rgba(255, 255, 255, 0.08)`).
* **Primary Accents:** Electric Indigo (`#4F46E5`) & Hyper Blue (`#3B82F6`) for interactive controls and focus states.
* **Status & Indicator Accents:**
  * **Emerald Green (`#10B981`):** Verified badges, completed course progress, gross revenue, and successful transactions.
  * **Violet (`#8B5CF6`):** Google Gemini AI tutor assistance, study roadmaps, and automated transcript analysis.
  * **Amber / Yellow (`#F59E0B`):** Pending reviews, Celery transcoding queues, and moderation alerts.
  * **Crimson Red (`#EF4444`):** WAF threat mitigations, copyright strikes, and account suspensions.
* **Typography:** Clean sans-serif hierarchy (Geist / Inter) with monospace tabular numbers for analytics and timecodes.

---

## 📁 Directory Structure & Feature Organization

The UI prototypes are organized into **5 modular, feature-based directories**:

```text
UI/
├── 00-design-system/
│   └── obsidian_kinetic/
│       └── DESIGN.md                  # Comprehensive design tokens, palette & glassmorphism specs
│
├── 01-authentication/
│   ├── user-auth/
│   │   ├── rkskilltube_authentication_onboarding/       # Split-screen Student/Creator Login & Registration
│   │   └── rkskilltube_authentication_modal_overlay/    # Centered Glassmorphic Quick Auth Modal
│   └── admin-auth/
│       └── rkskilltube_infrastructure_governance_admin_gateway/  # Invite-only restricted Super Admin Gateway
│
├── 02-learner-platform/
│   ├── rkskilltube_explore_learning_feed/               # Home Discovery Feed (Hybrid videos + courses)
│   ├── rkskilltube_trending_explore_feed/               # Ranked Trending Feed (#1, #2 engineering topics)
│   ├── rkskilltube_subscriptions_feed/                  # Creator Story Reels & Chronological Subscriptions
│   ├── rkskilltube_course_curriculum_syllabus_detail/   # Course Syllabus, Modular Lessons & Enrollment
│   ├── rkskilltube_course_video_watch_room/             # Cinema Video Player, Scrubber & AI Study Buddy
│   ├── rkskilltube_saved_playlists_library_management/  # Custom Playlists & 3D Stacked Thumbnails
│   ├── rkskilltube_watch_later_queue_management/        # Reorderable Watch Later Queue & Runtime Tracker
│   ├── rkskilltube_certificate_of_completion_credential_verification/ # Tamper-Proof Certificate Verification
│   └── rkskilltube_student_profile_account_settings/    # Student Profile, Study Goals & Video Preferences
│
├── 03-creator-studio/
│   ├── rkskilltube_creator_studio_dashboard_1/          # Creator Command Center (v1)
│   ├── rkskilltube_creator_studio_dashboard_2/          # Creator Command Center (v2 with deep metrics)
│   ├── rkskilltube_asynchronous_video_course_upload_studio/ # Non-blocking Background Upload Dock
│   ├── rkskilltube_course_curriculum_builder/           # Drag-and-Drop Course Syllabus Tree Builder
│   ├── rkskilltube_content_videos_management_studio/    # Content & Video Grid, Visibility & Batch Actions
│   ├── rkskilltube_creator_analytics_insights_dashboard/ # Retention Curves, Traffic Sources & Watch Time
│   ├── rkskilltube_creator_monetization_payouts_operations/ # Stripe/Razorpay Payouts & Membership Tiers
│   ├── rkskilltube_comments_student_q_a_management_inbox/ # Student Code Questions & Instructor Q&A
│   ├── rkskilltube_channel_studio_customization/        # Channel Banner, Layout Arranger & Links
│   └── rkskilltube_studio_channel_settings/             # Transcoding Defaults, Moderation & Permissions
│
└── 04-super-admin-governance/
    ├── rkskilltube_super_admin_operations_dashboard/    # Master Super Admin Command Center
    ├── rkskilltube_super_admin_financial_revenue_ledger/ # Platform Fee (20%) / Creator (80%) Ledger
    ├── rkskilltube_legal_dispute_dmca_compliance_console/ # DMCA Takedown Adjudication & Copyright Strikes
    ├── rkskilltube_content_moderation_quality_review_queue/ # Flagged Video Review Queue & AI Safety
    ├── rkskilltube_infrastructure_distributed_engine_dashboard/ # Celery Workers, Redis 7 & PostgreSQL Metrics
    ├── rkskilltube_user_channel_governance_directory/   # 10M User Directory, Bans & Verified Badges
    ├── rkskilltube_security_waf_console/                # Cloudflare WAF Threat Mitigation & IP Blacklist
    ├── rkskilltube_security_audit_logs_console/         # Immutable ISO-Timestamped Admin Audit Trail
    └── rkskilltube_global_platform_settings/            # Global Feature Flags & Emergency Maintenance Mode
```

---

## 📦 What is Inside Each Screen Folder?

Every individual screen folder contains two primary files:
1. **`code.html`**: The complete, standalone, production-ready HTML markup built with Tailwind CSS utility classes and modern SVG icons. You can open this file in any web browser to preview, inspect, or copy styling rules.
2. **`screen.png`**: A high-resolution rendered screenshot of the screen for instant visual reference during development and code review.

---

## 🚀 How to Use & Convert These UIs into Next.js React Code

### Step 1: Previewing Screens Locally
To inspect any screen without running a server, open the folder in your file explorer and double-click `code.html`. It will render in your default browser exactly as designed.

### Step 2: Route Mapping Reference
Each prototype corresponds directly to a Next.js 14 App Router route inside [`frontend/src/app`](../frontend/src/app):

| Prototype Screen | Destination Route in Next.js | Key Components to Build |
| :--- | :--- | :--- |
| `01-authentication/user-auth/*` | `app/(auth)/login/page.tsx`<br>`app/(auth)/register/page.tsx` | Split-screen hero, OAuth buttons, password strength meter |
| `01-authentication/admin-auth/*` | `app/(admin)/admin/gateway/page.tsx` | Hardware key (WebAuthn), TOTP input, restriction banner |
| `02-learner-platform/rkskilltube_explore_learning_feed` | `app/(main)/page.tsx` | Hero AI roadmap input, hybrid video & course cards grid |
| `02-learner-platform/rkskilltube_course_video_watch_room` | `app/(main)/watch/[videoId]/page.tsx` | Custom HLS player, chapter drawer, Gemini study chatbot |
| `02-learner-platform/rkskilltube_course_curriculum_syllabus_detail` | `app/(main)/courses/[courseId]/page.tsx` | Modular syllabus tree, preview trailer, enrollment card |
| `02-learner-platform/rkskilltube_student_profile_account_settings` | `app/(main)/users/me/settings/page.tsx` | Profile bio, engineering tags, video player preferences |
| `03-creator-studio/rkskilltube_creator_studio_dashboard_*` | `app/(creator)/studio/dashboard/page.tsx` | KPI metrics, recent upload tables, revenue breakdown |
| `03-creator-studio/rkskilltube_asynchronous_video_course_upload_studio` | `components/video/VideoUploadDock.tsx` | Non-blocking floating bottom dock, Celery progress monitor |
| `03-creator-studio/rkskilltube_course_curriculum_builder` | `app/(creator)/studio/courses/page.tsx` | Drag-and-drop chapter modules, lesson attachments |
| `04-super-admin-governance/*` | `app/(admin)/admin/*` | Financial ledger, DMCA adjudication, Celery infrastructure |

### Step 3: Migration Workflow into React / TypeScript
1. **Extract Layout Primitives:** Open `code.html` and extract common structural wrappers into reusable components (e.g., `Card`, `Badge`, `Button`, `Modal`).
2. **Translate to JSX:** Replace HTML attributes with React equivalents (`class` ➔ `className`, inline styles ➔ Tailwind classes).
3. **Connect State & Hooks:**
   * Use **Zustand** stores (`frontend/src/store`) for persistent global state (e.g., active floating upload queue, user authentication tokens).
   * Use **TanStack Query** for fetching course curricula and video metadata from the FastAPI backend.
4. **Integrate Real Backend APIs:** Connect actions to FastAPI endpoints defined in `backend/app/api/v1` (e.g., presigned upload URLs, view tracking, course enrollments).

---

## 🔄 Git Workflow: Keeping Your Repository Synced

To commit and push updates to the UI prototypes:

```bash
# 1. View all tracked and untracked changes
git status

# 2. Stage the UI directory and documentation
git add UI/

# 3. Commit with a structured conventional commit message
git commit -m "docs(ui): add comprehensive UI architecture guide, Stitch links, and route mapping"

# 4. Push to GitHub
git push origin main
```
