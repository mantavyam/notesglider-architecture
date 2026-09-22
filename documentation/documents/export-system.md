---
icon: download
---

# Export System

## SECTION 17 — EXPORT SYSTEM

### 17.1 PPTX Export (Client-Side via PptxGenJS)

> **v5 change**: PPTX generation moved from server-side `python-pptx` to **client-side `PptxGenJS`** (`https://github.com/gitbrent/PptxGenJS`). All `python-pptx` references for PPTX **generation** are removed from this PRD.

| Property                  | Specification                                                                                                                                                                                                                                                                                                     |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Who can use**           | Teacher, Editor, and Sub-members (with export permission)                                                                                                                                                                                                                                                         |
| **Trigger**               | "Export → .pptx" from the finalised document canvas                                                                                                                                                                                                                                                               |
| **Implementation**        | **Browser-side** using PptxGenJS. The browser fetches the document `content_json`, walks the Lexical tree, and assembles the `.pptx` directly. No FastAPI round trip for PPTX.                                                                                                                                    |
| **Input**                 | The Lexical editor's serialised structured JSON (including Brief Lexical sub-trees with inline image nodes per §10.9)                                                                                                                                                                                             |
| **Output**                | `.pptx` binary blob streamed to the browser as a download via `pptx.writeFile()`                                                                                                                                                                                                                                  |
| **Image handling**        | Browser fetches each Cloudinary URL with `f_jpg` (or `f_png` if alpha) transform — see §26.6. PptxGenJS embeds JPEG/PNG natively (no WebP support).                                                                                                                                                               |
| **Table pagination**      | PptxGenJS supports **auto-pagination for tables** natively. Use the built-in autopage option.                                                                                                                                                                                                                     |
| **Text pagination**       | PptxGenJS does **not** auto-paginate free text blocks. The codebase must include a single shared **height estimator utility** (built once, reused everywhere) that measures the rendered height of a text block at the configured slide size + font and splits content across slides before handing to PptxGenJS. |
| **Crop application**      | Inline image nodes carry `atomic_uid` → resolved to the image record's `crop` transform → applied as Cloudinary URL params on fetch.                                                                                                                                                                              |
| **Reveal.js involvement** | None — PPTX generation is completely independent of Reveal.js                                                                                                                                                                                                                                                     |
| **Multilingual PPTX**     | If Teacher has enabled translation, the client fetches both source and translated documents and assembles a single unified `.pptx` with alternating slide-pairs per language.                                                                                                                                     |
| **Q\&A inclusion**        | If the `qa` node is present in the source JSON, Q\&A items are rendered as a dedicated tail-section of slides (one Q\&A per slide, or grouped depending on type). See §10.10 for type variants.                                                                                                                   |
| **File delivery**         | Browser download triggered by PptxGenJS (`writeFile`)                                                                                                                                                                                                                                                             |
| **Failure mode**          | If browser runs out of memory on very large documents, PptxGenJS throws — UI surfaces _"Document too large for in-browser export. Please contact your Editor."_ No automatic server fallback in v5.                                                                                                               |

### 17.2 PDF Export via Reveal.js (PPTX → PDF pathway)

| Property                | Specification                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Who can use**         | Teacher, Editor, and Sub-members (with export permission)                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **Trigger**             | "Export → .pdf" from the finalised document canvas                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| **Implementation**      | Backend generates a standalone Reveal.js HTML file → file opens in a new browser tab → JavaScript triggers browser's native print dialog programmatically → user saves as PDF                                                                                                                                                                                                                                                                                                                                   |
| **Note**                | This pathway produces a slide-deck-formatted PDF. It is different from the branded WeasyPrint PDF produced by the Editor's PDF pipeline.                                                                                                                                                                                                                                                                                                                                                                        |
| **Local device save**   | **Uncompressed / lossless** — the PDF the user saves to their local device via the browser print dialog is the raw output of the browser. No preliminary compression step is applied client-side. Local copy is the highest-fidelity artefact the user can have.                                                                                                                                                                                                                                                |
| **Backend persistence** | When a Reveal.js print-to-PDF outcome is uploaded to the backend (see Auto-Persistence below), it goes through a **preliminary compression step immediately after the file is saved** (same pipeline as §26.5). Compression is **for backend cost savings only** — the Teacher's local-device copy stays untouched.                                                                                                                                                                                             |
| **Auto-Persistence**    | A Teacher may **never invoke** the Reveal.js print-to-PDF flow for a given Newsletter. To guarantee at least one slide-deck PDF artefact exists per document, the backend **auto-generates and stores** the Reveal.js print-to-PDF server-side (headless browser print) the moment the document's `event_type = "delivered"` event fires (Stage 3, §13.3). The auto-generated copy is the compressed backend variant. The Teacher's own local save (if/when they perform it) remains separate and uncompressed. |
| **Idempotency**         | If the Teacher subsequently invokes Reveal.js print-to-PDF, the server-side auto-generated copy is **not overwritten** — the local save is purely a user-facing download. The backend artefact remains the single canonical slide-deck PDF for that document.                                                                                                                                                                                                                                                   |

### 17.3 TXT Export

| Property           | Specification                                                                |
| ------------------ | ---------------------------------------------------------------------------- |
| **Who can use**    | Teacher, Editor, and Sub-members (with export permission)                    |
| **Trigger**        | "Export → .txt"                                                              |
| **Content**        | Plain text extraction of document content — all markdown formatting stripped |
| **Implementation** | Client-side or backend string processing of the Lexical JSON                 |

### 17.4 ZIP Export

| Property           | Specification                                                                                                                      |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| **Who can use**    | Teacher, Editor, and Sub-members (with export permission)                                                                          |
| **Trigger**        | "Export → .zip"                                                                                                                    |
| **Content**        | ZIP archive containing: (1) `/images/` sub-folder with all images from the document, (2) the `.txt` export of the document content |
| **Implementation** | Backend assembles ZIP from Cloudinary image URLs + text content                                                                    |

### 17.5 Reveal.js Presentation Mode (Live)

| Property                 | Specification                                                                                                                                               |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Who can use**          | Teacher, Editor, and Sub-members (with export permission)                                                                                                   |
| **Trigger**              | "Launch Presentation Mode" button from the canvas                                                                                                           |
| **Rendering**            | Fully client-side — structured JSON converted to Reveal.js format in the browser. No backend call for live presentation.                                    |
| **Display mode**         | Embedded as an iframe within the app UI shell                                                                                                               |
| **Controls**             | "Open in New Tab" button (opens standalone browser tab), "Enter Full Screen" button (browser fullscreen API on iframe), "Exit / Escape" (returns to canvas) |
| **Database persistence** | On document lock/approval: backend converts structured JSON → Reveal.js HTML → saved to Neon DB as the canonical raw presentation artefact                  |

### 17.6 PDF Pipeline Exports (Editor-Only — Branded WeasyPrint PDF)

This is a **distinct, separate export pathway** from the Reveal.js PDF export in Section 17.2. This pathway produces the branded, template-styled WeasyPrint PDF output.

| Property           | Specification                                                                                                                                               |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Who can use**    | Editor only. Super Admin, Teacher, and Sub-member never see this functionality.                                                                             |
| **Document types** | All document types: Newsletter, Compilation, Magazine, Mindmap, Quarterly Collection, Bi-Annual Compendium, Annual Yearbook, Category Extraction            |
| **Trigger**        | Editor approves document in review queue → triggers WeasyPrint PDF generation pipeline                                                                      |
| **Implementation** | FastAPI backend; WeasyPrint converts structured HTML with templates, headers/footers, ad banners to branded PDF                                             |
| **RBAC gating**    | PDF pipeline UI elements, buttons, and status indicators must never appear on any Teacher-facing or Sub-member-facing screen. Enforced via RBAC middleware. |

### 17.7 HTML Webpage Export (Editor-Only)

The Editor can view and export any document as a standalone HTML webpage.

| Property                 | Specification                                                                                                                                                            |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Who can use**          | Editor only                                                                                                                                                              |
| **View in browser**      | Editor can open any document in an iframe or separate new tab directly from the `.html` version                                                                          |
| **Export content**       | A webpage export bundle containing: `.html` file, `.css` stylesheets, and image assets linked locally (decoupled from CDN)                                               |
| **Local asset bundling** | On export, all Cloudinary CDN URLs in the HTML are replaced with local relative paths, and the corresponding image assets are downloaded and bundled alongside the HTML  |
| **Normal storage**       | When saved to databases normally (not exported), HTML files retain their Cloudinary CDN URLs for images and include the Atomic UID references (see Section 11A)          |
| **Use case**             | Allows Editor to open the webpage locally on a browser without requiring any markdown renderer to present the same information, providing a blog-like viewing experience |
