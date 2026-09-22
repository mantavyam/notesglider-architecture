---
icon: box-isometric
---

# Document Storage

## SECTION 24 — DOCUMENT STORAGE ARCHITECTURE (4-LAYER)

### 24.1 Overview

Document data is persisted across four distinct storage layers, each serving a different purpose and operating on a different trigger:

| Layer   | System                                            | Trigger                                                          | Purpose                                                                          |
| ------- | ------------------------------------------------- | ---------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| Layer 0 | Yjs CRDT realtime channel (in-memory + IndexedDB) | Every operation (keystroke-level)                                | Live co-visibility (§13.4) + offline/connectivity recovery (replaces v4 Layer 1) |
| Layer 1 | Browser IndexedDB (via Yjs persistence provider)  | Every operation                                                  | Local durability for offline recovery — folded into Layer 0                      |
| Layer 2 | Neon DB                                           | **Every 240 seconds (background)** — v5 raised from 30s for cost | Primary persistent storage, snapshot of Yjs doc state                            |
| Layer 3 | Neon DB (Named Version)                           | On Teacher finalisation                                          | Immutable version checkpoints                                                    |
| Layer 4 | Google Drive                                      | On approval/finalisation events                                  | Long-term archival and backup                                                    |

> **v5 change**: Neon DB autosave cadence raised from **30s → 240s** to reduce read/write cost significantly. Live editing UX is preserved because the realtime experience now flows over a separate Yjs CRDT channel (Layer 0), not the DB write cadence. The 240s tick is purely for crash-recovery/server-side persistence.

### 24.2 Layer 0/1 — Yjs CRDT + Browser IndexedDB (Keystroke-Level, Realtime, Offline)

* **Mechanism**: Lexical document state is bound to a Yjs document (`y-lexical` binding or equivalent). Yjs operations propagate over a WebSocket provider (`y-websocket`) for live co-visibility (§13.4), and are simultaneously persisted to the browser's IndexedDB via `y-indexeddb`.
* **Trigger**: Every editor operation (keystroke-level granularity).
* **Purpose**:
  * **Live co-visibility** — Editor's edits appear on the Teacher's read-only canvas in real-time without any DB round trip.
  * **Offline / connectivity-loss recovery** — local IndexedDB persists every op; on reconnect, CRDT semantics merge local ops with server state automatically (no manual conflict prompt required for the same user's own offline edits).
* **Connectivity Loss Recovery**: If the user's internet connection drops during editing, ops continue accumulating in IndexedDB. On reconnect, Yjs syncs and merges. If a stale tab/session is detected (e.g. Yjs vector clock indicates the local doc is behind server state), the system silently fast-forwards the local view — no prompt.
* **Cache Bloat Prevention**: Configurable retention window for IndexedDB Yjs snapshots (default: last 7 days of local ops per document). Older Yjs updates are pruned after server-side persistence is confirmed for the corresponding range.
* **Session Scope**: IndexedDB is per-device, per-browser. Yjs server doc is global.

### 24.3 Layer 2 — Background Neon DB Sync (Every 240 Seconds)

* **Mechanism**: Server-side Yjs provider (FastAPI worker, or sidecar Node service hosting `y-websocket`) snapshots the canonical Yjs document state and writes it as serialised JSON to the `documents.content_json` column.
* **Trigger**: Every **240 seconds** while the document has unsaved server-side changes. Also flushed immediately on certain events: document lock, named-version creation, document close.
* **Why 240s (raised from 30s in v4)**: Reduces Neon DB write volume by \~8× without harming UX, because Yjs Layer 0 already handles live co-visibility and offline recovery. The 240s cadence is a crash-safety floor — worst-case data loss on a Yjs server crash with no client persistence is 240s of edits, but client IndexedDB (Layer 1) reduces real-world loss to seconds.
* **Non-blocking**: Imperceptible to the user — no spinner, no toast, no pause.
* **Conflict Resolution**: Concurrent edits are resolved by CRDT merge in the Yjs server doc before being snapshot to Neon. There are no row-level write conflicts at the DB layer. Teacher write ops are rejected by the Yjs server-side authorisation check when `is-locked = true`.
* **On sync failure**: Retry silently with exponential backoff. The Yjs doc remains the source of truth; persistence is eventually consistent. Log failures server-side.

### 24.4 Layer 3 — Named Version on Finalisation

* **Trigger**: Teacher clicks "Finalise Document."
* **Behavior**: The current document state is saved as a named, immutable version checkpoint in Neon DB.
* **Version naming**: Semantic versioning. First finalisation = `1.0.0`. Subsequent finalisations after edits = `1.1.0`, `1.2.0`, etc. (minor version increments per finalisation).
* **Version metadata**: Each version record includes: version number, finalised by (user-id), finalised at (timestamp), change summary (auto-generated or user-provided note).
* **Immutability**: Once a named version is created, its content cannot be modified. New edits create new version records.

### 24.5 Layer 4 — Google Drive Sync (Event-Triggered, OPTIONAL)

Drive sync is **entirely optional** — gated by the Org-level toggle `organisations.drive_sync_enabled` (default `false`). See §25.0 for the system-wide policy. When the toggle is `false` this layer is **inactive** and storage is fully served from Layers 1–3 + Cloudinary + the backend object store.

When the toggle is `true`:

* **Default sub-toggle (auto-enabled when Drive is on)**:
  * Final PDF output → synced to Teacher's Drive AND (if mapped) Editor's Drive.
* **Sub-toggles (Configurable by Teacher — disabled by default even when Drive is on)**:
  * RAW document: saved as both a `.md` file AND a Google Docs file (for Google ecosystem-native preview).
  * Images folder: saved following the naming convention.
* **Storage Warning**: When a Teacher enables RAW document sync, a warning dialog fires: _"Enabling raw document sync will store additional files in your Google Drive. Your Google Drive has a 15GB free storage limit. Enabling this may consume significant storage space. Do you want to proceed?"_ Teacher must explicitly confirm by typing the confirmation approval message in the text field.
* **Drive Sync Triggers**:
  * Newsletter: on Teacher finalisation (RAW, if enabled) + on PDF delivery (FINAL, always)
  * Compilation: on PDF generation (FINAL)
  * Magazine: on PDF generation (FINAL)
  * Mindmap: on Mindmap PDF generation (FINAL)

### 24.6 Auto-Deletion & Data Retention Policy

* **Location**: Account Settings → Data Retention (per user: Teacher and Editor independently)
* **Configuration options**:
  * 3 months — delete data older than 3 months
  * 6 months — delete data older than 6 months
  * 12 months — delete data older than 12 months
  * Custom date range — define a specific date from which to delete
* **Minimum retention floor**: **2 months** — the system must reject any deletion configuration that would remove data less than 2 months old. This floor is non-negotiable and enforced server-side.
* **Execution**: Backend cron job evaluates each user's retention policy independently and executes deletions on schedule. Deletions are soft-deletes first (`is-deleted = true`), with hard deletes after a 7-day tombstone period.
* **Storage limit notifications**: If a new Drive upload fails due to storage limits → immediate email + in-app notification to the affected user with a specific CTA to free space. This is a non-silent failure — the user must take action.
