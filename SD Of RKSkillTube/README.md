# 📐 RKSkillTube — System Design Master Repository

> **Directory Purpose:** This directory contains the complete **High-Level Design (HLD)** and **Low-Level Design (LLD)** specifications for **RKSkillTube**, engineered to support 10 million users, 2,000 peak requests per second, and 100–120 TB of video assets.

---

## 🗂️ Navigation & Structure

| Section | Location | Key Contents |
| :--- | :--- | :--- |
| **00 - High-Level Design (HLD)** | [`./00-High-Level-Design/`](./00-High-Level-Design/README.md) | System capacity estimations, latency and availability SLOs, cloud infrastructure topology, database trade-offs, and DrawIO architecture diagrams. |
| **01 - Low-Level Design (LLD)** | [`./01-Low-Level-Design/`](./01-Low-Level-Design/README.md) | Object-oriented design patterns, database schema entities, clean architecture layers (Controller-Service-Repository), and API request/response DTO contracts. |

---

## 🎯 Target System Scale

```
┌────────────────────────────────────────────────────────┐
│               SYSTEM CAPACITY SPECIFICATIONS           │
├────────────────────────────────┬───────────────────────┤
│ Metric                         │ Target Value          │
├────────────────────────────────┼───────────────────────┤
│ Total Registered Users         │ 10,000,000 Users      │
│ Daily Active Users (DAU)       │ 1,000,000 DAU         │
│ Peak Traffic Load              │ 2,000 Requests/sec    │
│ Read / Write Ratio             │ 100 : 1               │
│ Storage Footprint (Videos)     │ 100 – 120 TB          │
│ Target p95 API Latency         │ < 200 ms              │
│ Target Video Playback Startup  │ < 1.5 seconds         │
│ System Availability Target     │ 99.9% Uptime          │
└────────────────────────────────┴───────────────────────┘
```

---

## 🧭 Architectural Mapping

```
┌────────────────────────────────────────────────────────────────────────┐
│                   HIGH-LEVEL DESIGN (HLD) PERSPECTIVE                  │
│  • Edge / CDN ──► Reverse Proxy / API Gateway ──► Backend Services     │
│  • Primary Database: PostgreSQL 16 (ACID transactions + pgvector)     │
│  • Distributed In-Memory Layer: Redis 7 (Caching, write buffers, locks)│
│  • Asynchronous Processing: Celery + FFmpeg Transcoding Pipeline       │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Realized through
┌───────────────────────────────────▼────────────────────────────────────┐
│                    LOW-LEVEL DESIGN (LLD) PERSPECTIVE                  │
│  • Clean Architecture: Presentation (API) ──► Domain (Services)        │
│                       ──► Persistence (Repositories) ──► ORM Entities  │
│  • Design Patterns: Strategy (Video Storage), Factory, Dependency Inj. │
│  • Data Contracts: Pydantic v2 DTOs & TypeScript Interfaces            │
└────────────────────────────────────────────────────────────────────────┘
```

---

### 📖 Related References
* For local setup instructions, see [`../Docs/INSTALLATION_GUIDE.md`](../Docs/INSTALLATION_GUIDE.md).
* For detailed technical rationale, see [`../Docs/tech_tool.md`](../Docs/tech_tool.md).
* For folder layout mapping, see [`../Docs/folder.md`](../Docs/folder.md).
