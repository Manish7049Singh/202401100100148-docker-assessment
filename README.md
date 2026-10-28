# FestPass — Docker & Container Security Assessment

## Project

FestPass is a three-tier registration application: an Nginx frontend, a Flask API, and a PostgreSQL database. Docker Compose connects the frontend to the backend and the backend to the database on separate networks. PostgreSQL data is stored in a named volume.

| Detail | Value |
|---|---|
| Name | Manish |
| Registration number (REGNO) | `202401100100148` |
| Frontend port | `8048` |
| Docker Hub username | `manish00008` |
| GitHub repository | [Manish7049Singh/202401100100148-docker-assessment](https://github.com/Manish7049Singh/202401100100148-docker-assessment) |
| Docker Hub repository | [manish00008/fest-backend](https://hub.docker.com/r/manish00008/fest-backend) |

The backend exposes `GET /api/health`, `GET /api/registrations`, and `POST /api/registrations`. The frontend is served at `http://localhost:8048/`; API requests are reverse-proxied by Nginx.


## Run with Docker Compose

Docker Desktop and Docker Compose must be running. Create a local environment file from the template, replace its placeholder with a strong local PostgreSQL password, and do not commit `.env`:

```powershell
Copy-Item .env.example .env
notepad .env
docker compose config
docker compose up -d --build
docker compose ps
curl.exe http://localhost:8048/api/health
curl.exe http://localhost:8048/api/registrations
```

The health response should include registration number `202401100100148` and `"database": true` once PostgreSQL is healthy. To create a registration:

```powershell
curl.exe -X POST http://localhost:8048/api/registrations `
  -H "Content-Type: application/json" `
  -d '{"name":"Alice","event":"Hackathon"}'
```

Stop the containers while preserving database data with:

```powershell
docker compose down
```

The Compose stack uses the named `festpass-db-data` volume. Do not use `docker compose down -v` if the database contents must be retained.

## Stage 3: Trivy scan and image hardening

The table compares the Trivy findings for the saved baseline scan (`fest-backend:0.1`) and hardened scan (`fest-backend:0.2`). The HIGH counts combine the OS and Python-package targets shown separately in the baseline report. Image sizes were checked from the corresponding local Docker image tags; Trivy's text report itself does not include image sizes.

| Image | HIGH findings | CRITICAL findings | Docker image size |
|---|---:|---:|---:|
| Before — `fest-backend:0.1` | 2,249 (2,243 OS + 6 Python) | 234 | 1.61 GB (412,105,350 bytes) |
| After — `fest-backend:0.2` | 44 | 0 | 197 MB (48,390,832 bytes) |

The `fest-backend:0.2` image uses a digest-pinned Python 3.12 slim base, pinned application dependencies, and a non-root runtime user. The saved final scan still reports HIGH and other non-critical OS package findings; the image is **not vulnerability-free**.

Reports: [`reports/stage3-before.txt`](reports/stage3-before.txt), [`reports/stage3-after.txt`](reports/stage3-after.txt), and [`reports/stage3-summary.txt`](reports/stage3-summary.txt).

The assessment's fixed-CRITICAL gate was run against `fest-backend:0.2` and exited with code `0`:

```powershell
trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 fest-backend:0.2
```

## Docker Hub publication and digest

The Docker Hub tag `manish00008/fest-backend:0.3` was published and its registry digest was verified:

```text
sha256:5653a183a84dfb3336d661eb4be06f3ffe09b4a41271221df42289c9881a2d00
```

The assessment-required `manish00008/fest-backend:1.0.0` tag was checked on Docker Hub and **was not published** as of 2026-10-28. Consequently, there is no verified `1.0.0` digest to report here. The `0.3` digest above is for a different version and is not a substitute.

## Signing and verification status

Cosign 3.1.4 signed the published `0.3` image by its immutable digest using an encrypted local key. Verification of that same digest with the matching public key succeeded. The private key and DPAPI-protected passphrase are stored outside this repository; `.gitignore` excludes `*.key`.

The assessment's release-specific Stage 4/5 requirements remain incomplete: `1.0.0` and `1.0` have not been published, `compose.yaml` still builds the backend instead of pulling `1.0.0`, the public signing key is not included in the repository, and the required unsigned `1.0.1` rejection test has not been performed. The keyless signing attempt expired before authorization; the successful `0.3` signature used the local-key fallback.

## Assessment stage status

- **Stage 1 — Containerise the backend:** completed and committed.
- **Stage 2 — Compose the stack:** completed; database health and persisted registrations were verified through the frontend port.
- **Stage 3 — Scan and harden:** completed; the saved Trivy scan has zero CRITICAL findings, while other OS findings remain.
- **Stage 4 — Version and publish:** partially completed; `0.3` is published, but the required `1.0.0` and `1.0` releases and registry-only Compose test remain outstanding.
- **Stage 5 — Sign and verify:** partially completed; the `0.3` digest is signed and verified, but the `1.0.0` signature, committed public key, and unsigned `1.0.1` rejection test remain outstanding.

## AI Tools Used

AI tools were used as development assistants to understand concepts, troubleshoot errors, and improve the implementation. All commands were executed and tested in my own environment.

| AI Tool | Purpose |
|---|---|
| **ChatGPT (OpenAI)** | Used to understand Docker and Docker Compose concepts, troubleshoot PowerShell and container errors, interpret Trivy vulnerability scan results, and get guidance on Cosign image signing and verification. |
| **GitHub Copilot** | Used to assist with code changes, Dockerfile hardening, Stage 4 and Stage 5 workflow automation, generating assessment documentation, and reviewing the project for missing requirements. |

All AI-assisted changes and commands were reviewed against the assessment requirements. The final implementation, test execution, and evidence collection were performed in my local environment.

## Evidence

Stage 1–3 screenshots and later Stage 4–5 command reports are retained in [`screenshots/`](screenshots/) and [`reports/`](reports/). Relevant reports include:

- [`reports/stage4-push.txt`](reports/stage4-push.txt) and [`reports/stage4-publish.txt`](reports/stage4-publish.txt)
- [`reports/stage4-container-test.txt`](reports/stage4-container-test.txt)
- [`reports/stage4-5-image-scan.txt`](reports/stage4-5-image-scan.txt)
- [`reports/stage5-sign.txt`](reports/stage5-sign.txt) and [`reports/stage5-verify.txt`](reports/stage5-verify.txt)
- [`reports/stage4-5-summary.txt`](reports/stage4-5-summary.txt)

No new screenshots are claimed by this README.
