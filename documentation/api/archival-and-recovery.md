---
icon: box-archive
---

# Archival & Recovery

## SECTION 26A — FULL-AY ARCHIVAL & RECOVERY

### 26A.1 Overview & Toggle

In v4, only **images** were archived at the academic-year boundary (§26.4). In v5, the Organisation owner ("Teacher") can opt into a **Full-AY Archive** that bundles **all data** produced during the previous academic year — JSON content, generated PDFs, generated PPTX files, all images, Q\&A nodes, revision history — into a single ZIP archive. The archive destination depends on the §25.0 Drive sync toggle: if Drive is enabled, the ZIP is uploaded to the Teacher's Google Drive; otherwise it lives in the backend object store and the Teacher downloads it via an in-app signed link. A reciprocal **recovery** flow lets the Teacher restore an archived AY by uploading the same ZIP back into the app — same flow regardless of where the ZIP was originally stored.

**Toggle**: `organisations.full_ay_archive_enabled` (boolean, default `false`). Configurable by the Teacher (Organisation owner) only — Sub-members, Editors, and Super Admins cannot toggle this. Surfaced in Teacher Dashboard → Organisation Settings → Data → "Full-AY Archive".

**Relationship to §26.4 image-only archive**:

* When `full_ay_archive_enabled = true`: the image-only archive cron from §26.4 is **suppressed**; the full-AY cron (§26A.2) handles images as part of the full ZIP. Single ZIP, single source of truth per AY.
* When `full_ay_archive_enabled = false`: the §26.4 image-only flow runs as documented in v4. No full-AY archive is produced.

### 26A.2 Archive Cron Job

* **Schedule**: Fires at the **end of the month following the Teacher's configured academic year end** (same grace period as §26.4 — one calendar month after AY end).
* **Independence**: Each Organisation's cron evaluates independently against its own AY end + toggle state.
* **Concurrency**: The cron job for a given Org is **mutually exclusive** with that Org's §26.4 image-only cron — only one runs per AY boundary, dispatch decided by the toggle state at trigger time.

**Cron steps**:

1. Snapshot all relevant Neon DB rows for the concluding AY for this Org: `documents`, `document_versions`, `pdf_outputs`, `images`, `revision_screenshots`, `qa_items`, `pipeline_events` (scoped to the AY).
2. For each document, write `content.json` (the latest `documents.content_json`) and all `versions/*.json` (from `document_versions`).
3. For each PDF in `pdf_outputs`, fetch the file and write to `pdfs/`.
4. For each PPTX exported within the AY whose binary was retained (note: PPTX is client-generated in v5; only PPTX explicitly saved to Drive is collected here), write to `pptx/`.
5. Fetch every Cloudinary asset for the AY (originals, **not** crop-transformed) and write to `images/`. Image filenames follow §26.3 convention.
6. Compute SHA-256 of every file written.
7. Build `manifest.json` (see §26A.3).
8. Sign the manifest with the Org-scoped HMAC key.
9. ZIP the entire tree (DEFLATE level 9) as `[Organisation Name]-AY-[YYYY-YYYY].zip`.
10. **Destination** (depends on Drive sync state, §25.0):
    * If `organisations.drive_sync_enabled = true` → upload to Teacher's Drive at `[Organisation]/[AY]/ARCHIVE/FULL-AY-[YYYY-YYYY].zip`. Teacher downloads from Drive.
    * If `organisations.drive_sync_enabled = false` → write to the backend object store at `org-archives/<org_id>/<academic_year>.zip`. Teacher downloads via a signed in-app link surfaced in Organisation Settings → Data → Archives. The same HMAC manifest, recovery flow (§26A.4), and conflict rules (§26A.5) apply identically — recovery accepts an upload from local disk regardless of where the ZIP came from.
11. After Drive upload is confirmed: purge Cloudinary assets for the AY (same as §26.4 step 6), mark `images.is_archived = true`, set `documents.is_archived = true` for AY documents, set `organisations.full_ay_archived_years += [YYYY]`.
12. Email Teacher: _"Your AY \[YYYY-YYYY] data has been archived. Download or recover via Organisation Settings → Data → Archives."_

### 26A.3 Archive ZIP Structure & Signed Manifest

```
[Organisation Name]-AY-2026-2027.zip
├── manifest.json              ← signed; see below
├── manifest.sig               ← HMAC-SHA256 of manifest.json bytes
├── content/                   ← latest content_json per document
│   ├── NL-20260301-0001.json
│   └── ...
├── versions/                  ← all document_versions
│   ├── NL-20260301-0001/
│   │   ├── 1.0.0.json
│   │   └── 1.1.0.json
│   └── ...
├── pdfs/                      ← all pdf_outputs binaries
│   ├── 01-03-26-Daily-Newsletter.pdf
│   └── ...
├── pptx/                      ← optional, only PPTX explicitly saved to Drive
├── images/                    ← Cloudinary originals (WebP)
│   ├── 1-IndiaLaunc-SCIENCE-01-03-26.webp
│   └── ...
├── qa/                        ← qa_items snapshot
│   └── NL-20260301-0001-qa.json
└── revisions/                 ← revision_screenshots (PNG)
    └── NL-20260301-0001/
        └── REV-001-shot-1.png
```

**`manifest.json` shape**:

```json
{
  "schema_version": "1.0.0",
  "org_id": "uuid-of-org",
  "org_name": "Sample Coaching",
  "academic_year": "2026-2027",
  "archived_at": "2027-05-31T18:00:00.000Z",
  "archived_by_cron": "ay_full_archive_v1",
  "totals": {
    "documents": 245,
    "versions": 612,
    "pdfs": 245,
    "pptx": 12,
    "images": 1850,
    "qa_items": 980,
    "revision_screenshots": 47
  },
  "files": [
    { "path": "content/NL-20260301-0001.json", "sha256": "...", "size_bytes": 4821 }
    /* one entry per file under content/, versions/, pdfs/, pptx/, images/, qa/, revisions/ */
  ],
  "atomic_uid_index": [
    { "atomic_uid": "010326-thumb-Img-...", "image_path": "images/1-IndiaLaunc-SCIENCE-01-03-26.webp" }
    /* full index mapping atomic_uid → file path for recovery rewiring */
  ]
}
```

**Signing**:

* Backend holds an **Org-scoped HMAC secret** (`organisations.archive_hmac_secret`, generated on Org creation, encrypted at rest).
* `manifest.sig = HMAC-SHA256(manifest.json bytes, org's secret)`.
* Signature is verified by the same secret on recovery. The secret never leaves the backend; the ZIP carries the signature, not the secret.

### 26A.4 Recovery Flow

**Trigger**: Teacher (Organisation owner only) → Organisation Settings → Data → "Recover AY" → uploads a `FULL-AY-[YYYY-YYYY].zip` from local disk or Google Drive picker.

**Backend recovery sequence**:

1. **Receive ZIP**, extract to a sandboxed temp directory.
2. **Validate structure**: required top-level dirs and `manifest.json` + `manifest.sig` must exist. If missing → reject with error: _"Archive file is malformed."_
3. **Parse `manifest.json`**: read `org_id` and `academic_year`.
4. **Authorisation check**: `manifest.org_id` must equal the current Teacher's `org_id`. If not → reject: _"Archive belongs to a different organisation."_ (Logged to `audit_logs`.)
5. **Verify signature**: recompute `HMAC-SHA256(manifest.json bytes, current_org.archive_hmac_secret)` and compare to `manifest.sig`. If mismatch → reject: _"Archive signature is invalid — the file may be corrupted or tampered with."_ (Logged to `audit_logs` with HMAC-failure event.)
6. **Verify file hashes**: each entry in `manifest.files[]` is recomputed and compared. Any mismatch → reject with the specific failing path.
7. **Conflict check** (§26A.5): refuse if any data already exists for `(org_id, academic_year)`.
8. **Image re-upload** (§26A.6): re-upload all `images/*` to Cloudinary; collect new URLs keyed by `atomic_uid`.
9. **Restore DB rows**: insert documents, versions, pdf\_outputs, qa\_items, revision\_screenshots, images, pipeline\_events. For every image URL reference in `content_json` (thumbnail `url`, Brief Lexical tree image-node `src`, derived `reference-images[*].url`), look up the new Cloudinary URL by `atomic_uid` and rewrite.
10. **Re-upload PDFs/PPTX** to Drive at their original paths (overwrite-safe because §26A.5 guarantees no AY data exists).
11. **Mark archive consumed**: `organisations.full_ay_archived_years -= [YYYY]`; flag the ZIP in Drive as "recovered" (rename suffix `.recovered.zip`).
12. **Notify Teacher**: _"AY \[YYYY-YYYY] recovered successfully. \[N] documents, \[M] images restored."_

If recovery fails at any step beyond extraction, the sandbox temp directory is purged; no DB writes are committed (transactional). The original ZIP in Drive is left untouched.

### 26A.5 Conflict Handling

**Rule**: Recovery is refused if **any** data already exists in the target Org for the target AY.

Concretely, the backend runs:

```sql
SELECT COUNT(*) FROM documents
WHERE org_id = :org_id
  AND document_date BETWEEN :ay_start AND :ay_end
  AND is_deleted = false;
```

If `count > 0`, the recovery is aborted with: _"AY \[YYYY-YYYY] already contains \[N] documents. Recovery would overwrite or duplicate existing data. To proceed, the existing AY data must be deleted first (Organisation Settings → Data → Delete AY)."_ The Teacher is offered a CTA to navigate to the destructive-delete flow (separate confirmation, separate audit log entry).

**Rationale**: Full-AY recovery is a high-impact operation. Per-record merge dialogs are out of scope for v5 (overengineering). The lean rule — refuse on conflict, force explicit user-driven cleanup — yields zero ambiguity and zero partial-state bugs.

### 26A.6 Image Re-Upload on Recovery

Cloudinary deletes archived images per §26.4 step 6 (or as part of §26A.2 step 11). On recovery the URLs in `content_json` are dead. Re-uploading is required.

**Procedure**:

1. For each file in `images/`, the backend uploads to Cloudinary with `f_webp`, preserving the file content bit-for-bit (no re-encoding).
2. Inject `atomic_uid` into Cloudinary MIME metadata (§11A.5).
3. Insert a new `images` row with the new `cloudinary_url` and `cloudinary_public_id`, reusing the original `atomic_uid` (the UID is the stable identity, not the URL).
4. Build an in-memory map `atomic_uid → new_cloudinary_url` from the manifest's `atomic_uid_index`.
5. For each document's restored `content_json`, walk:
   * `Newsletter.categories[*].News-Items[*].images.thumbnail.url` — rewrite by `atomic_uid`.
   * `Newsletter.categories[*].News-Items[*].Brief.root.children[]` Lexical tree — rewrite every image node's `src` by its `atomic_uid`.
   * Derived `images.reference-images[*].url` — rebuilt from the Brief walk.
6. Persist the rewritten `content_json` to `documents.content_json` and to all corresponding `document_versions.content_json`.

**Atomic UID immutability** (per §11A.3) makes this re-wiring deterministic — no fuzzy matching, no orphan images.

### 26A.7 Retention of Archive ZIPs in Drive

* Archive ZIPs in Drive are **never auto-deleted** by Notesglider. They remain in the Teacher's Drive at `[Organisation]/[AY]/ARCHIVE/FULL-AY-[YYYY-YYYY].zip` indefinitely or until the Teacher manually deletes them in Drive.
* If the Teacher deletes the ZIP from Drive, recovery becomes impossible. No app-side warning is enforced for Drive-side deletions; the Teacher owns their Drive.
* Drive storage usage warning surfaces during the §26A.2 upload step if Drive is approaching its quota (reuses §28.4 storage-limit handling).
