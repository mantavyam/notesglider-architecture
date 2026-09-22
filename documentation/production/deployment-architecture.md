---
icon: server
---

# Deployment Architecture

## SECTION 30 — DEPLOYMENT ARCHITECTURE

### 30.1 Infrastructure Stack

| Service                                | Technology                                            | Notes                                                    |
| -------------------------------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Frontend**                           | Next.js (App Router) containerised → Google Cloud Run | Serverless, auto-scaling                                 |
| **Backend**                            | FastAPI containerised → Google Cloud Run              | Separate Cloud Run service                               |
| **Database**                           | Neon (managed PostgreSQL)                             | External to GCP, accessed via connection string          |
| **Cron Jobs**                          | Google Cloud Scheduler                                | Triggers Cloud Run backend endpoints for scheduled tasks |
| **Image CDN**                          | Cloudinary                                            | External SaaS                                            |
| **Drive / Docs / Translate / YouTube** | Google APIs                                           | External — OAuth per-user                                |
| **Payments**                           | Razorpay                                              | External SaaS — Super Admin's master account             |
| **Real-time**                          | WebSocket via FastAPI/Starlette OR Supabase Realtime  | For collaborative document editing                       |
| **Email**                              | Resend                                                | Generous free tier; event-triggered email service        |

### 30.2 Containerisation

* Both the Next.js frontend and the FastAPI backend are containerised via Docker.
* Each service has its own `Dockerfile` and `docker-compose.yml` for local development.
* Google Cloud Run receives the Docker images via a CI/CD pipeline (e.g. GitHub Actions → Google Artifact Registry → Cloud Run deploy).
* Developer is responsible for defining the CI/CD pipeline based on this architecture.

### 30.3 Scheduled Jobs (Google Cloud Scheduler)

| Job                               | Schedule                                                                         | Endpoint Called                                                                                                         |
| --------------------------------- | -------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Compilation auto-trigger          | 7th, 14th, 21st, last day of each month at configured time (default 6:00 PM IST) | `POST /api/documents/compilation/auto-trigger`                                                                          |
| Magazine auto-trigger             | Last day of each month at configured time (default 6:00 PM IST)                  | `POST /api/documents/magazine/auto-trigger`                                                                             |
| Quarterly Collection auto-trigger | 7th day of the month following each quarter end (default 6:00 PM IST)            | `POST /api/documents/quarterly/auto-trigger`                                                                            |
| Bi-Annual Compendium auto-trigger | 7th day of the month following each half-year end (default 6:10 PM IST)          | `POST /api/documents/biannual/auto-trigger`                                                                             |
| Annual Yearbook auto-trigger      | 7th day of the 1st month of subsequent academic year (default 6:20 PM IST)       | `POST /api/documents/annual/auto-trigger`                                                                               |
| Billing cycle close (POSTPAID)    | 5th of each month at 6:00 PM IST                                                 | `POST /api/billing/cycle/close`                                                                                         |
| Cloudinary year-end archival      | Per-Teacher: end of month following academic year end (dynamic, evaluated daily) | `POST /api/images/archive/check`                                                                                        |
| Retention policy enforcement      | Daily                                                                            | `POST /api/data/retention/enforce`                                                                                      |
| SLA monitor                       | Every 5 minutes                                                                  | `POST /api/pipeline/sla/check` — checks all documents with running SLA clocks and triggers auto-continuations as needed |
| Audit log auto-cleanup            | Daily (if enabled by Super Admin)                                                | `POST /api/audit/cleanup/check` — archives and deletes logs per Super Admin's configured retention                      |

### 30.4 Real-Time Architecture Requirement

The real-time document co-editing sync (Teacher read-only view during Editor review) requires persistent connection support. Options:

**Option A — WebSocket via Starlette (built into FastAPI)**

* Cloud Run supports WebSocket via HTTP/2.
* Ensure Cloud Run is configured with `--session-affinity` for WebSocket connections.
* Implement reconnection logic on the client.

**Option B — Supabase Realtime (managed)**

* Use Supabase Realtime as a managed WebSocket layer.
* Backend publishes document state changes to Supabase channels.
* Frontend subscribes to channels and receives real-time updates.

Developer selects the approach based on expertise and project constraints. Both are acceptable. WebSocket is preferred to avoid adding another third-party dependency.

### 30.5 Environment Configuration

* Three environments: `development`, `staging`, `production`.
* Environment is reflected in `metadata.environment` field on all documents.
* Each environment has its own Neon DB branch, its own Cloud Run services, and its own Cloudinary environment.
* Never share production credentials with development or staging environments.
