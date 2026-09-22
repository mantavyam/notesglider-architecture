---
icon: ellipsis-stroke-vertical
---

# Non-Functional Requirements

## SECTION 31 — NON-FUNCTIONAL REQUIREMENTS

### 31.1 Performance

* **Performance targets**: Best-effort for v1.0. No hard SLA targets are defined at this stage.
* Monitoring (via Google Cloud Monitoring) will be implemented pre-deployment to observe real performance baselines.
* Specific performance SLAs will be defined in v1.1 based on observed production data.
* Developer should follow general best practices: paginated API responses, lazy loading, image optimisation via Cloudinary transformations, database indexing on all foreign keys and frequently queried columns.

### 31.2 Security

<table><thead><tr><th width="237.83203125">Requirement</th><th>Specification</th></tr></thead><tbody><tr><td>Authentication</td><td>JWT tokens with short expiry (e.g. 15 minutes) + refresh token rotation</td></tr><tr><td>Token storage</td><td>HttpOnly cookies — never localStorage</td></tr><tr><td>Password storage</td><td>bcrypt hashing (cost factor ≥ 12) for email/password auth</td></tr><tr><td>Sensitive credential storage</td><td>AES-256 encryption at rest for Razorpay keys (Super Admin) and OAuth tokens</td></tr><tr><td>HTTPS</td><td>Enforced on all environments. No plain HTTP.</td></tr><tr><td>RBAC enforcement</td><td>Server-side on every endpoint — never rely on client-side checks alone</td></tr><tr><td>Multi-tenant isolation</td><td><code>org_id</code> scoping on every query — enforce at ORM layer</td></tr><tr><td>Input validation</td><td>Validate and sanitise all user inputs on the backend. Use Pydantic models (FastAPI)</td></tr><tr><td>Rate limiting</td><td>Implement on all public and authenticated endpoints. Configurable per endpoint.</td></tr><tr><td>CORS</td><td>Strict CORS policy — only allow requests from the approved frontend origin</td></tr><tr><td>Dependency scanning</td><td>Include in CI/CD pipeline</td></tr></tbody></table>

### 31.3 Accessibility

* WCAG 2.1 AA compliance is mandatory across all screens.
* Keyboard navigation must work fully for all interactive elements.
* All images must have `alt` text.
* Sufficient colour contrast ratios (minimum 4.5:1 for normal text, 3:1 for large text).
* Screen reader compatible markup throughout.

### 31.4 Scalability

* Google Cloud Run auto-scales horizontally — no manual scaling configuration required for v1.
* Neon DB scales automatically per plan.
* Cloudinary handles image CDN scaling.
* Background jobs (cron, SLA monitor) must be idempotent — safe to run multiple times without side effects.

### 31.5 Monitoring (Pre-Deployment)

* Application monitoring must be configured using Google Cloud Monitoring before the app goes to production.
* Minimum monitoring coverage:
  * Error rate per endpoint
  * Response time per endpoint (p50, p95, p99)
  * WeasyPrint job success/failure rate
  * Drive sync success/failure rate
  * SLA compliance rate (% of documents that complete Stage 1 and Stage 2 within 2 hours vs. auto-continuation)
* Alerts sent to the Editor (system owner) for error spikes or service degradation.

### 31.6 PDF Compression

* All WeasyPrint-generated PDFs must be compressed before storage and Drive sync.
* Compression must not result in visible quality degradation.
* Recommended libraries: `ghostscript` (via Python subprocess) or `pikepdf`.
* Developer evaluates and selects the best option for quality-vs-size tradeoff.

### 31.7 Browser Support

* Target: Last 2 versions of Chrome, Firefox, Safari, and Edge.
* Internet Explorer is not supported.
* Mobile browsers: Chrome Mobile and Safari Mobile (iOS).
