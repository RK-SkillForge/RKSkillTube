# 🏗️ RKSkillTube — Infrastructure Architecture & GCP Migration Guide

## 📌 Executive Summary
This document provides a comprehensive explanation of every folder and file created within the `infrastructure/` directory for the **RKSkillTube** project. 

Because **Google Cloud Platform (GCP)** is confirmed as our primary cloud provider, this guide also highlights the exact changes required to transition the Infrastructure as Code (Terraform) setup from the default AWS template to **GCP-native services**.

---

## 🧭 Architectural Mapping: AWS vs. Google Cloud (GCP)

The initial template in `Docs/folder.md` referenced standard AWS services. For deployment on **Google Cloud (GCP)**, the architecture maps as follows:

| Component | Default / AWS Target | Google Cloud (GCP) Target | Why It's Needed in RKSkillTube |
| :--- | :--- | :--- | :--- |
| **Object Storage** | AWS S3 / Cloudflare R2 (`s3_r2.tf`) | **Google Cloud Storage (GCS)** (`gcs.tf`) | Stores raw user video uploads, processed HLS `.m3u8` playlists, `.ts` chunks, and storyboard spritesheets. |
| **Relational Database** | AWS RDS PostgreSQL (`rds.tf`) | **Cloud SQL for PostgreSQL** (`cloud_sql.tf`) | High-availability PostgreSQL 16 instance with the `pgvector` extension enabled for AI recommendations. |
| **Content Delivery (CDN)** | AWS CloudFront (`cloudfront.tf`) | **Cloud CDN + Cloud Armor** (`cloud_cdn.tf`) | Global low-latency edge caching for video streaming chunks and DDoS protection. |
| **Core IaC Provider** | AWS Provider (`main.tf`) | **GCP Provider (`hashicorp/google`)** (`main.tf`) | Configures GCP project, region, service account authentication, and remote state. |
| **Container Registry** | AWS ECR | **Artifact Registry** (`artifact_registry.tf`) | Stores Docker images for FastAPI backend, Celery worker, and Next.js frontend. |
| **Compute / Hosting** | AWS ECS / EKS | **Cloud Run** or **GKE (Google Kubernetes Engine)** | Serverless container execution (Cloud Run) or orchestrated cluster (GKE) for backend, worker, and frontend. |

---

## 📁 Detailed Folder & File Breakdown

### 1. `infrastructure/docker/`
Contains container definitions for containerizing each application tier.

* **`backend.Dockerfile`**
  * **Role**: Container definition for the FastAPI ASGI backend application.
  * **Why created**: Multi-stage Python 3.11/3.12 build that installs production dependencies without build bloat, runs Uvicorn ASGI server with non-root security principles, and serves API endpoints with high concurrency.
* **`frontend.Dockerfile`**
  * **Role**: Container definition for the Next.js 14+ (TypeScript) frontend.
  * **Why created**: Multi-stage Node.js build utilizing Next.js `output: 'standalone'` to produce a lean image (<150MB) for production deployment.
* **`worker.Dockerfile`**
  * **Role**: Container definition for the background Celery worker tasks.
  * **Why created**: Specifically packages Python along with pre-installed **FFmpeg** and necessary codecs to perform adaptive bitrate (ABR) HLS transcoding and VTT storyboard sprite generation.

---

### 2. `infrastructure/nginx/`
Handles reverse proxying, SSL termination, and rate-limiting.

* **`nginx.conf`**
  * **Role**: Production Nginx reverse proxy configuration.
  * **Why created**: 
    - Routes incoming traffic: `/api/*` to the FastAPI backend, `/` to Next.js frontend, and `/media/*` or `/static/*` directly or to CDN.
    - Enforces SSL/TLS encryption, HTTP/2 or HTTP/3 termination, Gzip/Brotli compression, client upload buffers (`client_max_body_size 500M` for video uploads), and rate-limiting against abusive requests.

---

### 3. `infrastructure/scripts/`
Automation utility scripts for local development and database provisioning.

* **`setup_local_db.sh`**
  * **Role**: Shell script to initialize the PostgreSQL database locally or in Docker.
  * **Why created**: Creates the `rkskilltube` database, installs required extensions (notably `CREATE EXTENSION IF NOT EXISTS vector;` for `pgvector`), and runs initial Alembic database migrations.
* **`seed_test_data.py`**
  * **Role**: Python script to seed mock development data.
  * **Why created**: Inserts realistic sample users, creator channels, courses, modular chapters, and sample video records so developers can test the platform locally without manual data entry.

---

### 4. `infrastructure/terraform/`
Infrastructure as Code (IaC) defining managed cloud infrastructure.

#### Current (Initial) Files vs. Required GCP Changes:
1. **`main.tf`**:
   * **Purpose**: Cloud provider configuration and backend state setup.
   * **Change Required**: Configure the `google` provider with `project`, `region`, and GCS remote state backend instead of AWS S3 remote backend.
2. **`s3_r2.tf` ➔ Recommended: `gcs.tf`**:
   * **Purpose**: Object storage buckets with CORS configuration for direct browser uploads.
   * **Change Required**: Replace AWS S3 bucket definitions with `google_storage_bucket` (e.g., `rkskilltube-raw-videos` and `rkskilltube-hls-media`) and configure CORS rules for web player access.
3. **`rds.tf` ➔ Recommended: `cloud_sql.tf`**:
   * **Purpose**: Managed relational database.
   * **Change Required**: Replace AWS RDS with `google_sql_database_instance` (PostgreSQL 16) with private VPC networking and database flags enabling vector operations.
4. **`cloudfront.tf` ➔ Recommended: `cloud_cdn.tf`**:
   * **Purpose**: Content delivery and caching.
   * **Change Required**: Configure Google Cloud HTTPS Load Balancer with Cloud CDN backend buckets and Cloud Armor security policies.

---

## 📋 Recommended Next Steps

1. **Approve File Renaming & Cleanup**:
   - Safely replace/rename:
     - `infrastructure/terraform/s3_r2.tf` ➔ `infrastructure/terraform/gcs.tf`
     - `infrastructure/terraform/rds.tf` ➔ `infrastructure/terraform/cloud_sql.tf`
     - `infrastructure/terraform/cloudfront.tf` ➔ `infrastructure/terraform/cloud_cdn.tf`
2. **Populate Configurations**:
   - Write GCP Terraform manifests with best practices.
   - Fill in Dockerfiles (`backend.Dockerfile`, `frontend.Dockerfile`, `worker.Dockerfile`).
   - Fill in `nginx.conf` and database initialization scripts.
