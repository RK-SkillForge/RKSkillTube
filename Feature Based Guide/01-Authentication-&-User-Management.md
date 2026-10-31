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
