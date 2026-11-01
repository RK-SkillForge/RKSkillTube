# 🔐 Feature 01: Authentication & User Management

> **Bounded Context:** Authentication, Token Rotation, Profile Management & Role-Based Access Control (RBAC).  
> **Status:** Foundational (Tier 1 - Easy)  
> **Backend Module:** `backend/app/api/v1/endpoints/auth.py`, `backend/app/api/v1/endpoints/users.py`  
> **Frontend Routes:** `/login`, `/register`, `/profile`, `/settings`  

---

## 🎯 1. Overview & Objectives

The **Authentication & User Management** module is the foundational gatekeeper for RKSkillTube. It provides secure identity verification, session persistence, role permissions, and user profile customization.

### Core Capabilities:
- **Email & Password Authentication:** Passwords hashed with `bcrypt` (12 salt rounds) with constant-time verification.
- **Google OAuth 2.0:** One-click social sign-in for seamless student onboarding.
- **JWT Token Lifecycle:** Short-lived Access Tokens (15–60 minutes) coupled with securely stored Refresh Tokens (7–30 days) and atomic rotation.
- **Role-Based Access Control (RBAC):** Three platform roles:
  - `USER / STUDENT`: Default role, allows watching videos, purchasing/enrolling in courses, taking notes, and commenting.
  - `CREATOR`: Can upload videos, create free/paid courses, customize channel profiles, and view monetization analytics.
  - `ADMIN`: Global system oversight, content moderation, abuse flags, and financial auditing.

---

## 🏛️ 2. Clean Architecture Breakdown

```
┌────────────────────────────────────────────────────────┐
│ 1. PRESENTATION LAYER (API Controllers)                │
│    • auth.py: /login, /register, /refresh, /logout     │
│    • users.py: /me, /me/profile, /me/settings          │
├────────────────────────────────────────────────────────┤
│ 2. BUSINESS LOGIC LAYER (Services)                     │
│    • auth_service.py: verify credentials, issue tokens │
│    • user_service.py: update profiles, change password │
├────────────────────────────────────────────────────────┤
│ 3. DATA ACCESS LAYER (Repositories)                    │
│    • user_repository.py: get_by_email, create_user     │
├────────────────────────────────────────────────────────┤
│ 4. PERSISTENCE LAYER (SQLAlchemy Models)               │
│    • models/user.py: User, RefreshToken entities       │
└────────────────────────────────────────────────────────┘
```

---

## 🗄️ 3. Database Schema

### `users` Table
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `id` | `UUID` | Primary Key, default `uuid4` | Unique user identifier |
| `email` | `VARCHAR(255)` | Unique, Indexed, Not Null | Normalized lowercase email |
| `hashed_password` | `VARCHAR(255)` | Nullable (for OAuth) | Bcrypt hashed password |
| `full_name` | `VARCHAR(100)` | Not Null | Display name |
| `role` | `VARCHAR(20)` | Default `'USER'`, Not Null | `'USER'`, `'CREATOR'`, or `'ADMIN'` |
| `avatar_url` | `VARCHAR(500)` | Nullable | Profile avatar image link |
| `is_active` | `BOOLEAN` | Default `TRUE` | Soft account deactivation flag |
| `is_verified` | `BOOLEAN` | Default `FALSE` | Email verification status |
| `created_at` | `TIMESTAMPTZ` | Default `NOW()` | Registration timestamp |
| `updated_at` | `TIMESTAMPTZ` | Auto-update | Profile modification timestamp |

### `refresh_tokens` Table
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `id` | `UUID` | Primary Key | Unique token record ID |
| `user_id` | `UUID` | Foreign Key (`users.id`) | Owning user reference |
| `token_hash` | `VARCHAR(255)` | Unique, Indexed | SHA256 hashed refresh token |
| `expires_at` | `TIMESTAMPTZ` | Not Null | Token expiration timestamp |
| `revoked` | `BOOLEAN` | Default `FALSE` | Revocation status on logout |

---

## 🔌 4. API Endpoints Contract

### Public Endpoints:
* `POST /api/v1/auth/register` — Registers a new student account.
* `POST /api/v1/auth/login` — Verifies credentials and returns access & refresh tokens.
* `POST /api/v1/auth/refresh` — Rotates and returns a fresh access token.
* `POST /api/v1/auth/google` — Authenticates Google OAuth ID token.

### Protected Endpoints (Bearer Token required):
* `GET /api/v1/users/me` — Fetches active user profile and permissions.
* `PUT /api/v1/users/me` — Updates profile info (name, bio, interests).
* `POST /api/v1/auth/logout` — Revokes the current refresh token.
* `PUT /api/v1/users/me/password` — Updates password with existing password verification.

---

## 🔒 5. Security Checklist
- [x] Passwords never stored in plaintext; salt rounds >= 12.
- [x] JWT secret key loaded exclusively from environment variables (`SECRET_KEY`).
- [x] Refresh tokens stored as one-way SHA-256 hashes in database.
- [x] Rate limiting applied to `/auth/login` and `/auth/register` to mitigate brute-force attacks.
- [x] Secure CORS configuration prohibiting arbitrary origins in production.

---

## 🛡️ 6. Security Approaches: Less Secure to Most Secure

| Security Tier | Approach | Mechanism | Pros | Risks & Tradeoffs |
| :--- | :--- | :--- | :--- | :--- |
| **Tier 1 (Less Secure)** | **Single Long-Lived JWT in LocalStorage** | Frontend sends credentials; server returns one JWT valid for 30 days stored in browser `localStorage`. | • Easiest to implement and debug.<br>• Fully stateless backend with no DB session overhead. | ⚠️ **Critical XSS Risk**: Any malicious JavaScript or script injection can steal the token.<br>⚠️ **No Revocation**: Token cannot be revoked if stolen. |
| **Tier 2 (Moderate)** | **Stateful Server-Side Sessions (HttpOnly Cookies)** | Server writes session ID in an encrypted `HttpOnly`, `SameSite=Lax` cookie; session stored in DB or Redis. | • JavaScript cannot read the session cookie (immune to XSS token theft).<br>• Super-admin can instantly kill sessions. | ⚠️ **CSRF Vulnerability**: Requires CSRF protection tokens for state-changing requests.<br>⚠️ Database or Redis lookups on every single incoming API request. |
| **Tier 3 (Production Standard - Target for RKSkillTube)** | **Dual-Token with Rotation (Access + Refresh)** | Short-lived Access Token (15–60 min) in memory/header; Refresh Token (7–30 days) stored as a SHA-256 hash in DB and rotated on each call. | • Limits exposure window if access token is leaked.<br>• Replay attacks trigger immediate invalidation of the entire token family.<br>• Highly scalable stateless auth with persistent revocability. | Requires frontend silent token refresh interceptor (handling 401 retries automatically). |
| **Tier 4 (Enterprise Grade)** | **OAuth 2.0 / OIDC + HttpOnly Cookie Jar + MFA** | Proof Key for Code Exchange (PKCE) flow, `SameSite=Strict` HttpOnly cookies, TOTP 2FA, and granular RBAC + ABAC. | • Defense-in-depth against XSS and CSRF.<br>• Maximum identity compliance and governance. | Increased initial development overhead and strict cross-domain reverse proxy setup. |

---

## 👥 7. Dual-Side Management Architecture (User vs. Super Admin)

### A. User Side (Self-Service)
- **Role Assignment:** Users register with default role `USER`.
- **Authentication:** Standard email/password or Google OAuth 2.0.
- **Profile Management:**
  - View own profile data (`GET /api/v1/users/me`).
  - Edit display name, bio, and avatar (`PUT /api/v1/users/me`).
  - Self-service password changes requiring existing password verification.

### B. Admin Side (Super Admin Access Granting & Governance)
- **Role Elevation:** Super Admin can promote users to `CREATOR` (course creation permissions) or `ADMIN` (platform management).
- **User Governance:** Super Admin can inspect user directory, filter by role/verification status, and soft-deactivate/ban abusive accounts (`is_active = False`).
- **Instant Revocation:** When an admin suspends a user, all active refresh tokens for that `user_id` are marked `revoked = True` in the database.

### Admin Endpoints Contract:
* `GET /api/v1/admin/users` — Paginated directory of all platform users (filters: `role`, `is_active`, `search`).
* `PUT /api/v1/admin/users/{user_id}/role` — Super-admin updates target user role (`USER`, `CREATOR`, `ADMIN`).
* `PATCH /api/v1/admin/users/{user_id}/status` — Super-admin activates or suspends a user account.

---

## 📁 8. Implementation File Map

### Backend (`backend/`)
| Module / Layer | File Path | Purpose & Responsibilities |
| :--- | :--- | :--- |
| **Data Models** | `backend/app/models/user.py` | SQLAlchemy models for `User` and `RefreshToken` tables. |
| **Security Core** | `backend/app/core/security.py` | Bcrypt password hashing, JWT generation, and token hash helpers. |
| **Dependencies & RBAC** | `backend/app/api/deps.py` | `get_current_user`, `get_current_active_user`, and role checkers (`RoleChecker(["ADMIN"])`). |
| **Schemas (DTOs)** | `backend/app/schemas/user.py`<br>`backend/app/schemas/auth_service.py` | Pydantic validation schemas for user input, responses, and token contracts. |
| **Repositories** | `backend/app/repositories/user_repository.py` | DB operations: querying users by email/id, updating roles, managing refresh tokens. |
| **Services** | `backend/app/services/auth_service.py` | Business logic for credentials verification, token generation, and rotation checks. |
| **API Endpoints** | `backend/app/api/v1/endpoints/auth.py`<br>`backend/app/api/v1/endpoints/users.py`<br>`backend/app/api/v1/endpoints/admin.py` | Controllers for authentication, profile editing, and super-admin user management. |

### Frontend (`frontend/`)
| Component / Module | File Path | Purpose & Responsibilities |
| :--- | :--- | :--- |
| **State Store** | `frontend/src/store/useAuthStore.ts` | Zustand store managing current session user, tokens, and role. |
| **Custom Auth Hook** | `frontend/src/hooks/useAuth.ts` | Reusable hooks for `login`, `logout`, `useCurrentUser`, `isAdmin`, `isCreator`. |
| **API Client** | `frontend/src/services/api.ts` | Axios/fetch client with request auth header injection and 401 response refresh interceptor. |
| **User Pages** | `frontend/src/app/(auth)/login/page.tsx`<br>`frontend/src/app/(auth)/register/page.tsx`<br>`frontend/src/app/(main)/profile/page.tsx`<br>`frontend/src/app/(main)/settings/page.tsx` | User authentication screens, public profile view, and personal account settings. |
| **Admin Pages** | `frontend/src/app/(admin)/admin/dashboard/page.tsx`<br>`frontend/src/app/(admin)/admin/users/page.tsx` | Super admin dashboard and interactive user management directory table. |
| **Route Protection** | `frontend/src/middleware.ts` | Next.js Edge middleware to protect `/admin/*` and `/studio/*` routes against unauthorized roles. |

---

## 🧩 9. Future-Proof Frontend Component Checklist

To ensure scalability, modularity, and easy maintenance:

1. **Security & Route Wrappers:**
   - `RoleGuard`: Reusable UI wrapper checking user roles before rendering sensitive views.
   - `AuthGuard`: Redirects unauthenticated users to `/login` with an appended redirect query.
2. **Authentication Elements:**
   - `LoginForm` & `RegisterForm`: Validated with React Hook Form + Zod.
   - `SocialAuthButton`: Pre-styled Google Sign-In button handling OAuth redirects.
3. **User Profile Components:**
   - `ProfileCard`: User avatar, display name, handle, role badge, and joined date.
   - `AvatarUploader`: Image selector with client-side preview and crop before upload.
   - `SecurityForm`: Change password interface with current password verification.
4. **Admin Management Console:**
   - `UserManagementTable`: Data grid with search, role filters, sorting, and pagination.
   - `RoleSelectorModal`: Dialog for Super Admins to promote or demote user permissions.
   - `UserStatusToggle`: Action button to toggle user activation status (`Active` / `Suspended`).

---

## 📚 10. Curated Learning Resources

- **Security & Threat Modeling:**
  - [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
  - [Auth0 Refresh Token Rotation Guide](https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation)
  - [Pragmatic Web Security: LocalStorage vs. Cookies](https://pragmaticwebsecurity.com/articles/apisecurity/localstorage-xss)
- **FastAPI Backend Authentication:**
  - [FastAPI Security Tutorial (OAuth2 with Password & Bearer with JWT)](https://fastapi.tiangolo.com/tutorial/security/)
  - [FastAPI Advanced Dependencies & Security](https://fastapi.tiangolo.com/tutorial/dependencies/)
- **Next.js & Frontend State:**
  - [Next.js App Router Authentication Documentation](https://nextjs.org/docs/app/building-your-application/authentication)
  - [Zustand State Management Documentation](https://zustand.docs.pmnd.rs/)
