# Document Pipeline

## SECTION 13 — DOCUMENT PIPELINE — COMPLETE SLA STATE MACHINE

### 13.1 Overview

This section defines the complete, unambiguous state machine that governs every document from Teacher submission to final Teacher receipt. Every stage, transition condition, timeout rule, and recovery behavior is specified here. The developer must implement this exactly.

### 13.2 Pipeline Participants

| Participant | Role in Pipeline                                                                                                         |
| ----------- | ------------------------------------------------------------------------------------------------------------------------ |
| **Teacher** | Initiates pipeline by submitting. Views document read-only during review. Receives final PDF. Can flag revision.         |
| **Editor**  | Reviews raw document. Approves or edits. Reviews generated PDF. Approves or allows auto-continuation. Resolves failures. |
| **System**  | Manages SLA clocks. Triggers auto-continuations. Executes WeasyPrint. Sends notifications. Updates document status.      |

### 13.3 Stage-by-Stage Specification

#### STAGE 0 — Pre-Submission (Teacher's Domain)

* Teacher creates document in canvas editor.
* Document `doc-status = draft`.
* Auto-save active (see Section 24 for storage architecture).
* Teacher finalises document (named version saved with datetime metadata).
* Teacher clicks "Send to Editor."
* **Confirmation dialog fires**: _"Once submitted, this document will be locked for editing while under review. Do you want to proceed?"_
* Teacher confirms.

**Transition to Stage 1:**

* `doc-status` → `review`
* `is-locked` → `true`
* `locked-at` → current timestamp
* `submitted-to-editor-at` → current timestamp
* `stage1-sla-deadline` → `submitted-to-editor-at + 2 hours`
* In-app notification sent to Editor: _"New document submitted by \[Teacher Name]: \[Document Title]"_
* Email notification sent to Editor (low priority — per notification settings)
* Document appears in Editor's Kanban: `Queued` column with SLA countdown

***

#### STAGE 1 — Editor Raw Document Review (2-Hour SLA)

**Editor Actions (within 2-hour window):**

* Opens the document in their review queue.
* Can read and edit all document content (heading, news items, images, descriptions).
* Editor's edits are visible to the Teacher in real-time (read-only for Teacher — see Section 13.4).
* Editor can approve when satisfied.

**On Editor Approval (within 2 hours):**

* `stage1-approved-by-editor-at` → current timestamp
* Document status remains `review`
* WeasyPrint PDF generation is triggered immediately (Stage 2 begins)

**On SLA Timeout (2 hours elapse, no Editor action):**

* System auto-triggers WeasyPrint PDF generation.
* System logs: `"Stage 1 SLA elapsed. Auto-continuing to PDF generation."`
* Editor receives in-app notification: _"SLA elapsed for \[Document Title]. PDF generation has been auto-triggered."_
* Stage 2 begins.

**SLA Pause Condition:**

* If WeasyPrint fails during any auto-triggered generation and all retries are exhausted, `sla-paused = true`.
* SLA deadline timers freeze — they do not continue counting.
* See Section 13.5 for WeasyPrint failure recovery.

***

#### STAGE 2 — PDF Generation & Editor PDF Review (2-Hour SLA)

**On PDF Generation Success:**

* WeasyPrint completes PDF generation.
* PDF page count is recorded → `total-pages-generated` and `billable-page-count` populated.
* PDF file is saved to the output store.
* `pdf-generation-triggered-at` → timestamp
* `stage2-sla-deadline` → `pdf-generation-triggered-at + 2 hours`
* Editor receives in-app notification: _"PDF for \[Document Title] has been generated and is ready for your review."_
* Document appears in Editor's Kanban: `PDF Generated` column with SLA countdown.

**Editor PDF Review (within 2-hour window):**

* Editor opens the generated PDF for review.
* Editor can approve delivery to Teacher.

**On Editor PDF Approval (within 2 hours):**

* `stage2-approved-by-editor-at` → current timestamp
* PDF is delivered to Teacher (Stage 3).

**On Stage 2 SLA Timeout (2 hours elapse, no Editor action):**

* System auto-delivers PDF to Teacher.
* System logs: `"Stage 2 SLA elapsed. PDF auto-delivered to Teacher."`
* Stage 3 begins.

***

#### STAGE 3 — Delivery to Teacher

* PDF is delivered: in-app notification + email (high priority, always on). Email links to a signed backend URL — Drive availability is not required.
* Email content: _"Your \[Document Title] PDF is ready for download."_ with download link.
* `doc-status` → `published`
* `delivered-to-teacher-at` → current timestamp
* `event_type = "delivered"` is written to `pipeline_events`. This event fans out to two side-effects:
  1. **Reveal.js auto-persistence** (always): backend headless-browser print of the Reveal.js slide-deck HTML runs, the resulting PDF is saved to the backend store, and the §26.5 compression step runs immediately. This guarantees a slide-deck PDF artefact exists for every delivered document, regardless of whether the Teacher ever invokes the in-app print-to-PDF action. See §17.2 Auto-Persistence.
  2. **Drive sync** (optional, per §25.0): if `organisations.drive_sync_enabled = true`, the WeasyPrint final PDF is uploaded to Teacher's Drive `YYYY/MMMYY/DAILY/FINAL/` and (if the sub-toggle is on) the Editor's Drive mirror. If `drive_sync_enabled = false`, this step is silently skipped — the document is fully accessible via the in-app dashboard and the email link.
* Teacher's Kanban card moves to `Delivered` column.
* Billing: `billable-page-count` is added to Teacher's current billing cycle running total.

**Teacher Options After Delivery:**

* Download PDF (always available via the in-app dashboard, independent of Drive sync state).
* Flag for revision.

***

#### STAGE 4 — Revision Queue (Manual Only — No Auto-Continuation)

**If Teacher flags the delivered PDF:**

* A revision **questionnaire wizard** opens (modal dialog). The wizard enforces the **mandatory screenshot step** — see §13.3.4A. The Teacher cannot submit the flag without at least one screenshot attached.
* Teacher provides a reason/note + at least one annotated screenshot.
* `revision-count` incremented by 1.
* Revision record appended to `revision-history` array: `{ flagged-at, flagged-by, reason, screenshots: [...metadata...], resolved-at: null }`. Screenshot binaries are stored in the `revision_screenshots` table (§29).
* Document re-enters Editor's queue as a **Flagged for Revision** item with screenshot thumbnails surfaced inline.
* `doc-status` → `review` (revision state)

**Critical Difference from Normal Pipeline:**

* **NO auto-continuation applies at any stage of a revision queue.**
* No SLA clocks run. The Editor must manually approve at every step.
* The Editor's dashboard shows flagged revision items with distinct visual treatment (red border, warning icon, priority badge) and an audible/visual alert.
* Editor notification: _"\[Teacher Name] has flagged \[Document Title] for revision. Manual review required."_ (in-app + email — this is elevated to high-priority notification).

**Revision Resolution:**

* Editor reviews, edits, and manually approves the raw document → manually triggers PDF regeneration → manually approves PDF → manually delivers to Teacher.
* On delivery: `revision-history[-1].resolved-at` is populated with current timestamp.
* Teacher can flag again, which starts the process again at `revision-count + 1`.

> **No upper limit on revision cycles is enforced in v1.** A maximum revision cycle count may be added in a future version.

***

#### 13.3.4A Revision Screenshot Wizard (Mandatory)

**Trigger**: Teacher clicks "Flag for Revision" on a delivered PDF.

**Wizard steps**:

1. **PDF viewer iframe** — the delivered PDF opens inside the wizard in an in-app iframe-based PDF viewer (e.g. PDF.js). Navigation controls: page-up/down, zoom, page indicator.
2. **Region annotation tool** — Teacher can:
   * Drag a **custom quadrilateral outline** on the current page to focus the Editor on a specific region (4 freely-placed points; not restricted to rectangles).
   * OR pick a **page or page range** (e.g. "p. 3" or "p. 5–7") for whole-page screenshots.
3. **Add more** — repeat step 2 up to the cap.
4. **Reason / comment** — free-text note (required, min 30 chars).
5. **Submit** — wizard validates and posts.

**Capture mechanism**: Frontend renders the relevant page(s) of the PDF onto an offscreen canvas, draws the quadrilateral overlay, crops to bounding box, exports as PNG.

**Constraints (hard-enforced)**:

| Constraint               | Value                                                              |
| ------------------------ | ------------------------------------------------------------------ |
| Format                   | PNG only                                                           |
| Max size per screenshot  | 1 MB (rejected client-side and server-side if exceeded)            |
| Max screenshots per flag | 5                                                                  |
| Min screenshots per flag | 1 (submission blocked at 0 — wizard "Submit" disabled until ≥1)    |
| Storage                  | Base64 in `revision_screenshots` table, linked to `revision_id`    |
| Retention                | Auto-purged at `revision-history[-1].resolved-at + 30 days` (cron) |

**Editor-side rendering**: In the Editor's "Flagged for Revision" queue, each flag card shows the reason note plus a horizontal strip of screenshot thumbnails. Click a thumbnail → expands to full-size with the original quadrilateral outline preserved as an overlay. Page range labels are surfaced next to each thumbnail (e.g. "Page 3, region").

### 13.4 Real-Time Document Co-Visibility (Yjs CRDT)

During Stage 1 (Editor is actively editing the raw document):

* The Teacher can see the Editor's changes **live, in real-time**, rendered in the canvas in read-only mode.
* This is functionally equivalent to Google Docs collaborative view — Teacher is a read-only observer.
* **Implementation requirement (v5)**: Live diffs flow over a **Yjs CRDT channel** (with a WebSocket provider — `y-websocket` or equivalent) **separate from** the 240-second Neon DB autosave (§24.3). This separation is mandatory because the 240s persistence cadence is too slow for live co-visibility.
  * Yjs broadcasts keystroke-level operations between connected clients in-memory — no DB write per keystroke.
  * Yjs document state is the source of truth during an active co-edit session; periodic snapshots are flushed to Neon at the 240s autosave cadence.
  * Yjs's local IndexedDB persistence provider (`y-indexeddb`) doubles as the §24.2 browser-cache recovery layer — on reconnect, local ops merge cleanly with server state via CRDT semantics, no manual conflict resolution.
  * Reconnect: if Teacher's connection drops, on resume they resync to the current Yjs doc state.
  * Status label visible in Teacher's view: _"Editor is currently reviewing your document"_.
* This real-time sync is **one-directional during lock** — only the Editor writes; the Teacher observes (Teacher's awareness presence is sent and displayed in the app bar at the top in an avatar icon representing the teacher but write ops are rejected when `is-locked = true`).

### 13.5 Non-Payment Pipeline Block

Separate from the SLA pipeline, the billing system can block the Teacher's ability to enter Stage 0:

| Billing State                               | Effect                                                                                                                                                                                                     |
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Invoice paid, within cycle                  | Full pipeline access                                                                                                                                                                                       |
| Invoice overdue — within 7-day grace period | Full pipeline access. Persistent in-app banner showing outstanding invoice amount and due date.                                                                                                            |
| Invoice overdue — after 7-day grace period  | "Send to Editor" button is disabled. Tooltip: _"Your account has an outstanding balance. Please clear your dues to submit documents for PDF generation."_ Document creation and PPTX export remain active. |
| Payment cleared                             | Full pipeline access restored automatically via Razorpay webhook trigger — no manual Editor intervention required.                                                                                         |

### 13.6 Q\&A Late Addition — PDF Regeneration Flow

When Q\&A is **added after** the document has already reached `doc-status = published` (Stage 3 delivered), the system follows a deterministic regen path. This applies whether the late addition comes from a Sub-member (with Q\&A permission), the Editor, or the Teacher returning to the published document.

**Trigger**:

* Actor opens the published document → "Add Q\&A" action → completes the §10.10 wizard → clicks "Accept Q\&A".

**Backend sequence**:

1. New named version is cut: current version `1.x.y` → `1.(x+1).0`. Original version is preserved in `document_versions`.
2. `qa` node is written to the new version's `content_json`.
3. `metadata.qa-late-added` is set to `true`. `metadata.has-qa` set to `true`.
4. WeasyPrint PDF regeneration is **automatically triggered** — same flow as Stage 2 generation but bypassing Editor SLA approval (the regen is a system action, not a queue item).
5. New `pdf_outputs` record is created. New `billable-page-count` recorded.
6. **Billing impact**: If the new PDF's page count is **greater** than the prior `billable-page-count`, the delta pages are added to the current billing cycle. If equal or lower, no billing change.
7. New PDF is delivered to the Teacher: in-app + email notification — _"Your \[Document Title] has been updated with Q\&A. Updated PDF is ready for download."_
8. Drive sync (only if `organisations.drive_sync_enabled = true`, per §25.0): replaces the FINAL PDF in `YYYY/MMMYY/DAILY/FINAL/` with the new version; previous version moves to `YYYY/MMMYY/DAILY/FINAL/_archive/`. When Drive is disabled, the backend store is the single source of truth and the prior version is moved to a backend `_archive/` prefix instead.

**Constraints**:

* Late-add regen does **not** re-enter the SLA pipeline. No Stage 1/Stage 2 queues. No Editor approval required.
* Late-add regen is **not** allowed once the document's billing cycle has been **finalised in a paid invoice** (`documents.is_billed = true`). UI hides the "Add Q\&A" action; tooltip: _"Document is finalized in invoice \[INV-####]. Q\&A can no longer be added."_
* Translation regen: if the original document had translations (`is-translated = true`), each translated parallel document also regenerates its PDF with the Q\&A node translated. See §18.
