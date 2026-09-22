---
icon: triangle-exclamation
---

# Error Handling & Recovery

## SECTION 28 — ERROR HANDLING & RECOVERY LADDERS

### 28.1 Image Upload Failure — Recovery Ladder

**Scenario**: An image upload to Cloudinary fails during Newsletter creation.

<table><thead><tr><th width="239.32421875">Attempt</th><th>Behavior</th></tr></thead><tbody><tr><td>Attempt 1</td><td>Backend silently retries the Cloudinary upload — no user interruption</td></tr><tr><td>Attempt 2</td><td>Second silent auto-retry</td></tr><tr><td>Attempt 3 failure</td><td>Processing halts temporarily. Teacher receives in-app notification: <em>"Image upload failed for [Headline]. Please re-upload or replace the image."</em> Teacher is given a re-upload option.</td></tr><tr><td>Re-upload Attempt 1</td><td>Teacher re-uploads image. Backend attempts again.</td></tr><tr><td>Re-upload Attempt 2</td><td>Another auto-retry.</td></tr><tr><td>Re-upload Attempt 3 failure</td><td>Error is silently logged. Document continues processing <strong>without the image</strong>.</td></tr></tbody></table>

**Fallback Behavior After 3rd Re-upload Failure:**

* A **custom placeholder image is auto-generated** from the news headline text for the affected section. This can be a server-side image with the headline text rendered on a neutral background.
* After PDF generation completes, the Teacher receives an in-app notification: _"The following news items used auto-generated placeholder images due to upload failures: \[list of headlines]. You can replace these images by editing the document."_

**Editor Visibility:**

* All image failure events are logged in `metadata.image-failures` array: `{ news-item-headline, failure-reason, placeholder-generated: true/false }`.
* Editor sees the full failure log during their review queue processing.
* Editor has the ability to manually upload a replacement image for any placeholder-affected news section before triggering PDF generation.

***

### 28.2 Google Drive Sync Failure — Recovery

**Scenario**: A Google Drive sync operation fails (e.g. during finalisation or PDF delivery).

<table><thead><tr><th width="196.9453125">Attempt</th><th>Behavior</th></tr></thead><tbody><tr><td>Attempt 1</td><td>Backend silently retries the Drive API call</td></tr><tr><td>Attempt 2</td><td>Second silent auto-retry</td></tr><tr><td>Both retries exhausted</td><td>Error is logged. Failure surfaces in Super Admin's system dashboard AND Editor's RBAC dashboard with full error response detail. Email notification sent to Super Admin and Editor with error response included.</td></tr></tbody></table>

**Teacher Impact**: None — Drive sync failures are completely abstracted from the Teacher. The Teacher is never notified of Drive sync failures.

**Super Admin Resolution**: Super Admin investigates the error from system-wide logs and can manually trigger a re-sync for the affected document. Super Admin escalates to a software developer if the issue indicates a systematic application problem.

**Editor Visibility**: Editor sees the failure notification and can manually trigger a re-sync for the affected document from their dashboard, but investigation and root cause analysis is the Super Admin's responsibility.

**Logging**: All Drive sync failure events are stored in `metadata.drive-sync-failures` array: `{ attempted-at, error-code, error-message, path-attempted }`.

***

### 28.3 WeasyPrint PDF Generation Failure — Recovery

**Scenario**: WeasyPrint fails during PDF generation (either auto-triggered or manually triggered by Editor).

<table><thead><tr><th width="199.47265625">Attempt</th><th>Behavior</th></tr></thead><tbody><tr><td>Attempts 1–N</td><td>Backend auto-retries WeasyPrint generation N times. N is <strong>configurable by the Super Admin</strong> in Super Admin System Settings → PDF Pipeline Configuration. Developer should implement a sensible default (recommended: 3 retries).</td></tr><tr><td>All N retries exhausted</td><td>Failure escalates to Super Admin and Editor.</td></tr></tbody></table>

**On Escalation:**

* `sla-paused = true` is set in the document metadata.
* The SLA countdown clock **pauses at its current value** — it does not reset, and it does not continue counting while the failure is unresolved.
* Super Admin receives **high-priority** in-app notification AND email: _"PDF generation failed for \[Document Title] after \[N] retries. Investigation required. Error: \[error details]."_
* Editor receives **high-priority** in-app notification AND email: _"PDF generation failed for \[Document Title]. The Super Admin has been notified and is investigating."_
* All organisation members receive a consolation in-app notification: _"A processing issue was detected for \[Document Title]. Our team is investigating."_
* The document is flagged in the Editor's Kanban with an error indicator.

**Super Admin Resolution:**

* Super Admin investigates the error from system-wide logs (template issue, malformed HTML, WeasyPrint config problem, etc.).
* Super Admin corrects the underlying issue or escalates to a software developer.
* Super Admin or Editor manually re-triggers PDF generation from the document detail view.
* On re-trigger: `sla-paused = false`, SLA clock resumes from where it was frozen.

**External Service Downtime Note**: If the error is caused by downtime of external services (Cloudflare, CDN, Google APIs), this does not indicate a systematic application issue. The core PDF generation pipeline (WeasyPrint) operates independently and shall continue to function normally. Super Admin acknowledges the external downtime and waits for service restoration before re-triggering affected operations.

***

### 28.4 Storage Limit Failure — Drive & Neon DB

**Scenario**: A file upload to Google Drive or a data write to Neon DB fails due to storage limits.

<table><thead><tr><th width="126.72265625">System</th><th>Behavior on Storage Limit Hit</th></tr></thead><tbody><tr><td>Google Drive</td><td>Immediate email + in-app notification to the affected user (Teacher or Editor) AND the Super Admin. Message includes: storage used, storage limit, recommended action. User must resolve (delete old files or upgrade storage) before further Drive syncs will succeed. Super Admin monitors storage alerts system-wide.</td></tr><tr><td>Neon DB</td><td>Same notification pattern. Backend suspends non-critical writes. Core document state writes are prioritised. Super Admin investigates and coordinates resolution.</td></tr></tbody></table>

* These failures are **non-silent** — the user must take action.
* The system does not silently skip Drive uploads without notifying the user.

***

### 28.5 Real-Time Connection Loss (WebSocket / Realtime)

**Scenario**: The real-time connection between Teacher and Editor during active document review is lost.

<table><thead><tr><th width="313.51953125">Event</th><th>Behavior</th></tr></thead><tbody><tr><td>Teacher connection drops</td><td>Teacher's view shows: <em>"Connection lost. Reconnecting..."</em> loading state.</td></tr><tr><td>Reconnection success</td><td>Teacher's view resyncs to the current document state automatically. Any changes made by the Editor during the disconnection are reflected immediately upon reconnect.</td></tr><tr><td>Reconnection failure after 3 attempts</td><td>Teacher sees: <em>"Unable to reconnect to live collaboration. Please refresh the page."</em></td></tr></tbody></table>

***

### 28.6 General API Error Handling

* All FastAPI endpoints must return structured error responses: `{ "error": true, "code": "ERROR_CODE", "message": "Human-readable message", "details": {} }`.
* All frontend API calls must handle errors gracefully with user-facing messages. Silent failures are not acceptable for any user-triggered action.
* Rate limiting must be implemented on the FastAPI backend. On rate limit hit: return HTTP 429 with `Retry-After` header. Frontend shows: _"Too many requests. Please wait a moment and try again."_
