# 🏗️ RKSkillTube — Infrastructure, Containers & Cloud

<div align="center">

[![Docker](https://img.shields.io/badge/Container-Docker%20Compose-2496ED?style=for-the-badge&logo=docker)](https://www.docker.com/)
[![Terraform](https://img.shields.io/badge/IaC-Terraform-844FBA?style=for-the-badge&logo=terraform)](https://www.terraform.io/)
[![GCP](https://img.shields.io/badge/Cloud-Google%20Cloud-4285F4?style=for-the-badge&logo=google-cloud)](https://cloud.google.com/)
[![Nginx](https://img.shields.io/badge/Reverse%20Proxy-Nginx-009639?style=for-the-badge&logo=nginx)](https://nginx.org/)

**Container orchestration, reverse proxy configuration, local database seeding, and Google Cloud Platform (GCP) Infrastructure as Code.**

</div>

---

## 📁 Directory Overview

```
infrastructure/
├── docker/                        # Multi-stage production container definitions
│   ├── backend.Dockerfile         # FastAPI production image
│   ├── frontend.Dockerfile        # Next.js 14 standalone production image
│   └── worker.Dockerfile          # Celery worker image with bundled FFmpeg
├── nginx/                         # Reverse proxy & gateway
│   └── nginx.conf                 # Routing (/api -> backend, / -> frontend) with SSL & buffers
├── scripts/                       # Automation and seeding utilities
│   ├── setup_local_db.sh          # PostgreSQL & pgvector initialization script
│   └── seed_test_data.py          # Seeds sample users, channels, courses, and videos
├── terraform/                     # Google Cloud Platform (GCP) IaC
│   ├── main.tf                    # GCP provider & remote state configuration
│   ├── gcs.tf                     # Google Cloud Storage buckets for raw & HLS videos
│   ├── cloud_sql.tf               # Cloud SQL PostgreSQL 16 instance with pgvector
│   └── cloud_cdn.tf               # Cloud CDN & Cloud Armor edge caching configuration
└── explain.md                     # Deep-dive GCP migration & architectural rationale
```

---

## ⚡ Quick Start: Running Local Infrastructure

The root `docker-compose.yml` orchestrates the core data stores required for local development:

```bash
# Launch PostgreSQL 16, Redis 7, and Celery Flower dashboard
docker compose up -d

# View status:
docker compose ps

# View logs:
docker compose logs -f
```

### Initializing Database & Seed Data
```bash
# Initialize local database extensions (pgvector):
bash infrastructure/scripts/setup_local_db.sh

# Seed mock development data (users, courses, videos):
python infrastructure/scripts/seed_test_data.py
```

---

## ☁️ Cloud Architecture & Deployment

For a comprehensive explanation of how RKSkillTube is deployed on **Google Cloud Platform (GCP)** and the mapping from standard multi-cloud patterns to GCP services, please read:

👉 [**`infrastructure/explain.md`**](./explain.md)
