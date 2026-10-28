# FestPass - Docker & Container Security Practical Assessment (CA-2)

## 1. Student & Project Details
- **Student Name:** Adarsh Singh
- **Registration Number (REGNO):** 202401100200017
- **Assigned Port:** 8017 (`80 + 17`)
- **Docker Hub Username:** adarshsingh000
- **Docker Hub Repository:** [https://hub.docker.com/r/adarshsingh000/fest-backend](https://hub.docker.com/r/adarshsingh000/fest-backend)
- **GitHub Repository:** [https://github.com/Adarsh011732/202401100200017-docker-assessment](https://github.com/Adarsh011732/202401100200017-docker-assessment)

---

## 2. AI Tools Usage Declaration
- **Tool:** Google Antigravity (Powered by Gemini 3.8)
- **Usage:**
  - Automated analysis of assessment requirements and security guidelines.
  - Hardening of `backend/Dockerfile` by migrating from bloated `python:3.9` to `python:3.12-alpine` and introducing a dedicated non-root user (`appuser`).
  - Formulating network segmentation in `compose.yaml` (dual isolated bridges: `frontend-backend` and `backend-db`).
  - Diagnosing and remediating Windows/WSL2 port binding conflicts (`wslrelay` collision).
  - Executing Trivy vulnerability analysis and saving before/after audit reports.
  - Orchestrating Sigstore Cosign cryptographic key generation, OCI artifact signing by digest, signature verification, and unsigned rejection validation.

---

## 3. Docker Image Digest (Version 1.0.0)
- **Tagged Image:** `adarshsingh000/fest-backend:1.0.0`
- **Digest (pushed to Docker Hub):**
  ```text
  sha256:70f76e540f1a00ec8556a9cb472ec296fae8e83cbf40ec6d68fd41d93699ec56
  ```

---

## 4. Before-and-After Vulnerability Comparison (Task 3.3)

| Metric | `fest-backend:0.1` (Before) | `fest-backend:0.2` (After) | Improvement |
| :--- | :--- | :--- | :--- |
| **Base Image** | `python:3.9` (Debian bullseye) | `python:3.12-alpine` (Alpine 3.24.3) | Pinned lightweight Alpine base |
| **Execution User** | `root` (UID 0) | `appuser` (non-root UID 100) | Principle of Least Privilege |
| **CRITICAL Vulnerabilities** | **234** | **0** | **100% eliminated** |
| **HIGH Vulnerabilities** | **2,249** | **0** | **100% eliminated** |
| **Total Vulnerabilities** | **13,381** | **6** (0 High/Critical) | **>99.9% reduction** |
| **Trivy Exit Code (`--severity CRITICAL --ignore-unfixed`)** | Failed (`1`) | **Passed (`0`)** | Compliant with security gate |
| **Image Size** | **1.61 GB** | **107 MB** | **93.4% size reduction** |

---

## 5. Incomplete Steps
- **None** — All 5 stages (Stages 1 through 5) and required tasks were fully completed and validated.

---

## 6. Architecture & Security Implementation Summary

### Network Isolation (Stage 2)
The application runs across two isolated user-defined bridge networks:
1. `frontend-backend`: Connects `frontend` (Nginx reverse proxy) and `backend` (Python Flask API).
2. `backend-db`: Connects `backend` and `db` (PostgreSQL 17).
The frontend container has no route or DNS resolution to `db`, preventing lateral movement should the frontend be compromised (`ping -c 1 db` returns `bad address 'db'`).

### Storage Persistence (Stage 2)
Database records are stored in a dedicated named Docker volume (`pgdata` mapped to `/var/lib/postgresql/data`), ensuring zero data loss across container lifecycle events (`docker compose down` and `docker compose up -d`).

### Image Signing & Verification (Stage 5)
Cryptographic signing was performed using **Cosign** (Sigstore):
- Key pair generated locally with `.gitignore` strictly excluding `*.key`.
- Image `1.0.0` signed directly by its manifest digest: `adarshsingh000/fest-backend@sha256:70f76e540f1a00ec8556a9cb472ec296fae8e83cbf40ec6d68fd41d93699ec56`.
- Verification against `cosign.pub` validated successfully.
- Unsigned release `1.0.1` was tested and correctly rejected with `Error: no signatures found`.
