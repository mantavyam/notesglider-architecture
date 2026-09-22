---
icon: image
---

# Cloudinary API Integration

## SECTION 26 — CLOUDINARY & IMAGE LIFECYCLE MANAGEMENT

### 26.1 Three-Tier Image Storage Architecture

| Tier       | System       | Role                                            | Retention                               |
| ---------- | ------------ | ----------------------------------------------- | --------------------------------------- |
| Primary    | Neon DB      | Image metadata, CDN URLs, associations          | Permanent (subject to retention policy) |
| Active CDN | Cloudinary   | Fast image delivery during active academic year | Current academic year only              |
| Archive    | Google Drive | Long-term historical image archive              | Permanent                               |

### 26.2 Image Upload Flow

1. Teacher uploads an image (drag-and-drop or file picker on thumbnail slot), OR pastes web content containing inline `<img>` tags into a Brief field (§10.9).
2. Image is sent to the FastAPI backend.
3. Backend uploads image to **Cloudinary** with **WebP** as the storage format (`f_webp` upload transformation, lossless or quality-auto per §26.6) and receives the CDN URL.
4. CDN URL is stored in Neon DB (in the document's JSON schema `images.thumbnail` or appended to the Brief Lexical tree as an image node — see §10.9; the derived `images.reference-images` array is rebuilt by the backend on save).
5. The document JSON never stores raw image files — only Cloudinary CDN URLs.
6. Metadata record for the image is created in the Neon DB `images` table: `{ image-id, cloudinary_url, cloudinary_public_id, document_id, news_item_headline, category, document_date, teacher_id, editor_id, upload_timestamp, academic_year, image_type, atomic_uid, crop, origin }`.
7. Crop transforms (when applied via §10.8) are recorded as `crop = { x, y, w, h, applied_at }` on the image record and mirrored to the document JSON. The original WebP asset is **never overwritten** by a crop operation — render-time URL params apply the crop instead.

### 26.3 Image Naming Convention (Drive)

When images are synced to Google Drive, they are renamed following this convention:

```
n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension
```

Where:

* `n` = order index of the image within the document (1, 2, 3...)
* `HEADLINE[:10]` = first 10 characters of the news headline (stripped of special characters)
* `CATEGORY` = news category name
* `DD-MM-YY` = document date
* `extension` = original file extension (jpg, png, webp, etc.)

### 26.4 Academic Year Boundary & Year-End Archival

> **v5 note**: When a Teacher enables the **Full-AY Archive** toggle (§26A), the image-only archival flow described below is **subsumed** by the full-AY ZIP. The cron job still fires at the same boundary, but it builds a single per-AY archive containing JSON + PDF + PPTX + images instead of an images-only ZIP. When the toggle is **off** (default), the image-only flow below applies as written.

* **Default academic year**: January 1 — December 31 (calendar year)
* **Per-Teacher override**: Each Teacher can configure a custom academic year start month in Account Settings. The archival cron job evaluates each Teacher's boundary independently.
* **Data retention grace period**: Cloudinary data is retained for **one full calendar month** after the academic year ends. Deletion is scheduled at the **end of the month following the academic year's final month**.
  * Example: Academic Year 2026 (January 2026 – December 2026) → archival and deletion is scheduled on **January 31, 2027**.
  * Example: Custom academic year April 2026 – March 2027 → archival and deletion is scheduled on **April 30, 2027**.
* **Retrospective document creation**: During the one-month grace period, Teachers can still create documents retrospectively for dates in the previous academic year. Backend checks `cloudinary_data_retention_active = true` for the previous year before allowing retrospective document creation. Once the grace period expires and deletion fires, retrospective document creation for the previous academic year is permanently disabled.

**Year-End Archival Process (Python backend cron job):**

1. Cron job fires at the **end of the month following the Teacher's configured academic year end date** (one-month grace period).
2. Backend queries all Cloudinary images for the concluding academic year for that Teacher.
3. Images are downloaded from Cloudinary.
4. Python script compresses images (without quality loss) and packages them into a ZIP archive.
5. ZIP archive is uploaded to the Teacher's Google Drive at: `YYYY/ARCHIVE/IMAGES-YYYY.zip`.
6. Once Drive upload is confirmed, backend calls Cloudinary API to delete all archived images.
7. Neon DB image records are updated: `{ archived: true, cloudinary-url: null, drive-archive-path: "YYYY/ARCHIVE/IMAGES-YYYY.zip", cloudinary_data_retention_active: false }`.
8. Backend sets `retrospective_creation_allowed = false` for the concluded academic year — Teachers can no longer create documents for dates in that year.

### 26.5 PDF Compression

* Python backend applies PDF compression to all PDFs **stored on the backend** before persistence and (optional) Drive sync.
* Compression must not result in visible quality loss.
* Developer should evaluate libraries such as `ghostscript` (via Python subprocess) or `pikepdf` for this purpose.
* **Scope**:
  * **WeasyPrint-generated branded PDFs** (§17.6) — compressed on the server immediately after WeasyPrint completes. Always.
  * **Reveal.js slide-deck PDFs** (§17.2) — compressed **only** for the backend-stored copy. The backend copy is produced by either (a) the auto-persistence step triggered at `event_type = "delivered"` (server-side headless-browser print), or (b) any future flow that uploads a slide-deck PDF to the backend. Compression runs as a preliminary step **immediately after the PDF file is saved to the backend store**.
  * **Local device save** of the Reveal.js print-to-PDF (Teacher invokes the browser print dialog and saves to disk) is **NOT compressed** — it is the raw browser output, lossless and highest-fidelity. No preliminary step runs on the local path.
* Rationale: compression is a backend-cost optimisation. The user's local artefact is unaffected — they keep the highest-quality copy possible, while the backend stores the lean variant.

### 26.6 Image Storage Codec (WebP) & Render Transcoding

* **Storage codec**: All images uploaded to Cloudinary are stored as **WebP** (`f_webp` on upload). WebP delivers \~25–35% smaller file sizes vs JPEG at perceptually equivalent quality.
* **Quality setting**: `q_auto:good` for photographic content; `q_auto:best` for charts/infographics (auto-detected by Cloudinary).
* **Browser display**: Browsers consume WebP directly (modern browser support is universal in our minimum-supported set — see §31.7). No transcoding needed for canvas display.
* **Render-time transcoding for PDF/PPTX**:
  * **WeasyPrint** (PDF generation, §17.6): Backend fetches the Cloudinary asset with `f_jpg` transformation to receive a JPEG variant inline. WeasyPrint embeds the JPEG. The original WebP master stays untouched.
  * **PptxGenJS** (client-side PPTX generation, §17.1): Browser fetches the Cloudinary asset with `f_jpg` (or `f_png` if the source has alpha). PptxGenJS embeds JPEG/PNG natively — it does not consume WebP.
  * **HTML export** (§17.7): Bundled image assets are converted to JPEG (or PNG if alpha) during export to maximise compatibility for offline viewing. CDN URLs are rewritten to local relative paths.

### 26.7 File-Type Compression (Pre-Upload)

All generated artefacts must be compressed before persistence/Drive sync. Compression must not produce visible quality loss.

| File type             | Compression method                                                                                                                |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| PDF                   | `ghostscript -dPDFSETTINGS=/printer` or `pikepdf` linearisation — see §26.5                                                       |
| PPTX                  | After `PptxGenJS` assembly, browser zips the OOXML parts; the underlying ZIP is already deflate-compressed. No extra step needed. |
| Reveal.js HTML (live) | Server gzip on transport; HTML embeds Cloudinary URLs (no asset duplication).                                                     |
| ZIP exports           | Standard `DEFLATE` level 9.                                                                                                       |
| Images                | WebP at upload (§26.6). Pre-render transcoded JPEG uses Cloudinary `q_auto:good`.                                                 |
