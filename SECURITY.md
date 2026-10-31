# 🛡️ Security Policy for RKSkillTube

At **RKSkillTube**, we take security seriously. We appreciate the efforts of security researchers and community contributors who help keep our platform and users safe.

---

## 📋 Supported Versions

We provide security updates and patches for active versions of RKSkillTube:

| Version | Status | Supported |
| :--- | :--- | :---: |
| `main` branch | Current Development | :white_check_mark: |
| `0.1.x` | Alpha / Early Release | :white_check_mark: |
| `< 0.1.0` | Deprecated / Prototype | :x: |

---

## 🚨 Reporting a Vulnerability

If you discover a security vulnerability or sensitive bug in RKSkillTube, **please DO NOT create a public GitHub issue.** Publicly disclosing vulnerabilities before they can be resolved puts the community at risk.

Instead, please report vulnerabilities through one of the following private channels:

### 1. GitHub Security Advisory (Recommended)
Submit a report privately via GitHub:
1. Navigate to the **Security** tab of the repository on GitHub.
2. Click **Report a vulnerability** to open a draft advisory.
3. Provide details and reproduction steps.

### 2. Direct Security Email
If you prefer email, send a report to:
📧 **`rkskillforge@gmail.com`**

---

## 📝 What to Include in Your Report

To help us triage and resolve the issue quickly, please include:
- A clear description of the vulnerability and its potential impact.
- Step-by-step reproduction instructions or a minimal Proof of Concept (PoC).
- The affected component:
  - Frontend (`Next.js / Video Player / Auth Flow`)
  - Backend API (`FastAPI / Endpoints / JWT Validation / CORS`)
  - Database (`PostgreSQL / SQLAlchemy / SQL Injection / pgvector`)
  - Media Processing (`FFmpeg / Celery Worker / File Upload`)
  - Cloud / Infrastructure (`Docker / GCS Storage Bucket / Nginx`)
- Any suggested mitigations or patches (if available).

---

## ⏱️ Response Timeline

- **Initial Acknowledgment:** Within **48 hours** of receiving your report.
- **Triage & Assessment:** Within **5 business days**, confirming whether the issue is accepted or declined.
- **Remediation & Patching:** High/Critical severity issues will be prioritized for immediate patching.
- **Public Disclosure:** Coordinated public disclosure will take place only after the fix has been deployed and verified.

---

## 🏅 Safe Harbor & Recognition

We consider security research conducted in good faith to be authorized and will not pursue legal action against researchers who:
- Make a good faith effort to avoid privacy violations, data destruction, and service interruption.
- Give us reasonable time to resolve the issue before public disclosure.
- Do not exploit vulnerabilities for extortion or malicious purposes.

With your consent, we will gladly credit your responsible disclosure in our release notes and contributor hall of fame!
