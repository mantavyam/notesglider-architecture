# Newsletter

## SECTION 10 — NEWSLETTER CREATION FLOW (TEACHER — CANVAS EDITOR)

### 10.1 Flow Initiation

1. Teacher clicks the "Daily Newsletter" entry-point card on their Dashboard.
2. A confirmation or direct navigation triggers the Newsletter creation flow.
3. A new dedicated page opens styled as a **Google Docs-style canvas interface** with:
   * A top menu bar (formatting tools, action buttons)
   * A left sidebar minimap (outline / table of contents — see Section 10.5)
   * A main content canvas (block editor powered by Lexical.dev)

### 10.2 Metadata Sheet (Right Slide-in Panel)

Simultaneously with the canvas opening, a sheet panel slides in from the right side of the screen containing:

| Field         | Default Value              | Required?                           |
| ------------- | -------------------------- | ----------------------------------- |
| Document Date | Today's date (auto-filled) | Yes — editable by Teacher           |
| Video URL     | Empty                      | No — Teacher can skip and set later |

* A "Skip for Now" button allows the Teacher to dismiss this panel and fill details later.
* The Video URL field is where the Teacher attaches a YouTube video link (see Section 27 for the custom URL-based YouTube video integration — no YouTube API OAuth scope is used).
* This sheet can be re-opened at any time during document creation.

### 10.3 Block Editor Behaviour (Lexical.dev Canvas)

The canvas uses a block-based structure where content is organised into **Categories** (nodes) and **News Items** (child nodes/sub-nodes):

#### Adding Content

* A rounded **`+` (plus) button** is displayed at the Category level and News Item level.
* **Clicking `+` at Category level**:
  * Creates a new Category block with one News Item placeholder auto-added inside it (head start for the user).
  * Placeholder fields: Category Name (H1), one News Item with Headline, Image, and Description fields pre-populated as editable placeholders.
* **Clicking `+` at News Item level**:
  * Creates a new News Item block within the current Category.
  * All fields are added together as a unit: Headline block, Image block, Description/Brief block.

#### Field-Level Controls

**Clear Field Button**:

* A subtle clear button is positioned at the corner of each individual field.
* On click: An alert dialog appears asking the user to confirm the clear action.
* The dialog includes a checkbox: _"Don't ask again — you can re-enable this in Settings."_
* If checked, the alert dialog for that action type is globally suppressed until the user re-enables it in Settings.
* This preference is stored as a boolean in user settings (`clear_field_warning_enabled: true/false`).

**Delete Functionality**:

* A delete icon button is placed at the Category (node) level and News Item (sub-node) level.
* Same alert dialog behaviour as the Clear Field button (with "Don't ask again" checkbox).
* **Deleting a Category (node)**: Deletes the category AND all news items and their content within it — irreversible after confirmation.
* **Deleting a News Item (sub-node)**: Deletes only that news item and its fields (headline, image, description) without affecting the parent category or sibling news items.

#### Image Fields

Each News Item supports three image configurations:

| Configuration     | Fields                                                                    |
| ----------------- | ------------------------------------------------------------------------- |
| Full images block | Thumbnail (explicit upload) + Reference Images (inline-pasted, see §10.9) |
| Thumbnail only    | Thumbnail only (no inline reference images present in Brief)              |
| No images         | No thumbnail + no inline images in Brief                                  |

Image URLs are uploaded to Cloudinary and the returned Cloudinary URL is stored. Images are not stored as raw file references in the document JSON — only the CDN URL. Thumbnail is an explicit upload action by user (drag-drop or file picker into the dedicated thumbnail slot). Reference images enter via paste into the Brief field — see §10.9.

#### Text Formatting Support

The Lexical editor must support the following formatting within Brief/content fields:

* Headings (H1, H2, H3)
* Paragraph text
* Bold, Italic, Underline
* Bullet lists and numbered lists
* Tables (with optional tabular data for news items where relevant)
* Code blocks (lower priority)
* Separator lines
* Inline links

### 10.4 Drag-and-Drop Reordering

* Each Category block has a drag handle on its left edge.
* Dragging a Category moves the entire category including all its News Items to a new position in the document order.
* Each News Item has its own drag handle.
* Dragging a News Item can:
  * **Reorder within the same Category** — move it up or down within the parent category.
  * **Move to a different Category** — drag the News Item handle over a different Category's drop zone to re-parent it. The system removes the link to the old parent category and establishes the link to the new category.

### 10.5 Sidebar Minimap (Table of Contents)

* A vertical sidebar on the left side of the canvas displays an auto-generated, dynamically updated **table of contents / outline** view.
* The outline reflects the document hierarchy as it is built: Category names at the top level, News Item headlines indented below.
* Clicking any item in the outline scrolls the canvas to that section.
* The sidebar is **drag-scrollable** (similar to VSCode's minimap) — user can drag the viewport indicator to navigate through long documents quickly.
* The outline updates in real-time as the user adds, edits, or reorders blocks.

### 10.6 Finalisation & Actions

Once the Teacher is satisfied with the document, they have the following options accessible from the top menu bar:

#### Action 1 — Send to Editor

* Submits the document to the Editor's review queue for the PDF pipeline.
* A confirmation warning dialog appears before submission: _"Once submitted, this document will be locked for editing while under review. Do you want to proceed?"_
* On confirmation: document `doc-status` changes to `review`, document is locked (read-only for Teacher), and the Editor receives a notification.

#### Action 2 — Export Document

The Teacher can export the document in the following formats:

| Format             | Technical Implementation                                                                                 |
| ------------------ | -------------------------------------------------------------------------------------------------------- |
| `.pptx`            | Client-side via PptxGenJS (browser walks Lexical tree → writes `.pptx` directly, no backend). See §17.1. |
| `.pdf` (Reveal.js) | Reveal.js HTML opens in a new browser tab, browser print-to-PDF is triggered programmatically            |
| `.txt`             | Plain text export of document content, stripping all formatting                                          |
| `.zip`             | A ZIP archive containing: all images in an `/images` sub-folder + the `.txt` document export             |

#### Action 3 — Launch Presentation Mode

* Converts the structured document JSON to Reveal.js format entirely **client-side** in the browser (no backend call for the live view).
* The presentation is embedded as an iframe within the app UI shell.
* Controls available in presentation mode:
  * **Open in New Tab** button — opens presentation in a standalone browser tab
  * **Enter Full Screen** button — triggers browser fullscreen on the iframe
  * **Exit / Escape** — returns Teacher to the canvas editor

#### Action 4 — Translate Document

* Opens a language selection dropdown/modal.
* Teacher selects one or more target languages (e.g. Hindi, Tamil, Marathi).
* Google Translate API is called in the backend to translate the document content.
* A new **parallel document version** is created, linked to the original document via `doc-id`.
* The original document is **never overwritten or replaced**.
* A language switcher appears in the document header to toggle between the original and translated versions.
* See Section 18 for full translation system specification.

### 10.7 Locked Document Behaviour

After a Teacher submits a document:

* The canvas switches to **read-only mode** for the Teacher.
* A clear banner displays: _"This document is currently with the Editor for review. You cannot edit it while it is in the review queue."_
* The Teacher can still **view** the document in real-time, including watching the Editor's live changes as they are made (Google Docs collaborative view, read-only for Teacher — see Section 13.4).
* If the Teacher attempts any edit action, the system shows a non-dismissible tooltip: _"Document locked for editing."_

### 10.8 Image Crop (Non-Destructive)

Applies to **all image entities** in a News Item, with no distinction between source paths:

* **Thumbnail** (explicitly uploaded by the user via the dedicated thumbnail slot).
* **Reference images** — including those inserted via **inline paste** into Brief (§10.9). Every image node in the Brief Lexical tree exposes the same crop affordance as the thumbnail. The crop transform attaches to the image node's underlying `images.reference-images[]` entry (matched by `atomic_uid`).

Behaviour is identical across the two:

* Image placeholder displays an **edit (crop) icon** in the corner.
* Click → in-app crop interface opens (rectangular crop only; aspect ratio free unless template enforces one).
* On crop apply: a **reset icon** appears next to the edit icon. User can revert the crop at any time before final submission.
* On Teacher "Send to Editor" (document lock): the crop transform is **frozen**. Reset icon is hidden. Crop is no longer reversible.
* **Storage model — non-destructive**:
  * Original Cloudinary asset is **never overwritten**.
  * `atomic_uid` is **never reissued** (preserves §11A audit chain).
  * Crop is stored as a transform record `{x, y, w, h, applied_at}` on the image entity in the document JSON and in the `images` table.
  * Renderers (WeasyPrint, PptxGenJS, HTML export) apply the crop at render time via Cloudinary URL transformation parameters (e.g. `c_crop,x_,y_,w_,h_`). Zero new infra.
* If the document is unlocked later (e.g. revision queue), the frozen crop remains applied but is still non-destructive — the original asset stays intact and could be re-cropped if Editor edits the image.

### 10.9 Inline Pasted Reference Images

When a Teacher pastes copied web content (headline body, charts, infographics) into a News Item Brief field, embedded `<img>` tags must be preserved **inline at their original textual position** for downstream rendering.

* **Editor behaviour**: Lexical detects pasted `<img>` nodes and inserts them as image nodes in the Brief's Lexical tree at the cursor/paste position — between paragraph nodes, in the order encountered.
* **Background upload**: Each pasted image is uploaded asynchronously to Cloudinary by the FastAPI backend. Until upload completes, the image node shows a low-opacity preview with a small spinner badge. On upload success, `src` is swapped to the Cloudinary URL and `atomic_uid` is assigned.
* **Upload failure handling**: If a paste-image upload fails after retries, the image node shows a broken-placeholder badge with a "Retry upload" action. The Teacher cannot finalise the document while broken placeholders exist (blocking validation on "Send to Editor").
* **`reference-images` array**: This array in the document JSON becomes **derived metadata** — built backend-side by walking the Lexical tree of all Brief fields in the document. Used for billing counts, Drive sync naming convention (§26.3), Cloudinary archival (§26.4 / §26A), and audit. The array is **not the source of truth for position** — the Lexical tree is.
* **Schema impact**: `News-Items[].Brief` changes from a plain string to a serialised Lexical JSON tree. See §11.1 for the updated schema.
* **Renderer expectation**: WeasyPrint, PptxGenJS (client-side, §17.1), and HTML export (§17.7) walk the Brief Lexical tree — paragraph nodes → `<p>`, image nodes → `<img>` — preserving inline order automatically.

### 10.10 Pre-Submission Q\&A Wizard

A pre-submission step appears **immediately before** the "Send to Editor" confirmation. Skippable — the Teacher can dismiss it and submit without Q\&A. If skipped, the Q\&A may later be added by a Sub-member (with permission) or by the Editor during Stage 1; see §13.6 for late-add behaviour.

**Trigger UI**: Right-side sheet component, opens on clicking "Generate Q\&A" in the top menu bar or via a prompt after clicking "Send to Editor".

**Q\&A Wizard Steps**:

1. **Number of questions**: Numeric input. Min 1, max 10. Default 1.
2. **News item selection grid**: After the count is set, the sheet renders a grid view of all News Items currently in the document. Each grid cell shows the news item's headline, category, and a scrollable preview of the Brief. A rounded checkbox at the corner toggles selection; selected cells get a blue outline. Teacher can select up to `N` items where `N` = the number chosen in step 1.
3. **Question type selection**: Default = `subjective.straightforward` for every question. An edit icon expands per-question customisation, allowing each of the N questions to be set independently to one of the three types. Canonical JSON shapes (see §11.1 `qa.items[]` for full examples):
   * **`subjective.straightforward`**: `{ statement, answer }`
   * **`objective.direct`**: `{ statement, options: { A, B, C, D }, correct_option }` where `options` is a fixed-key map (all four keys required, values are the answer text), and `correct_option ∈ { "A", "B", "C", "D" }`.
   * **`objective.statement_analysis`**: `{ topic, statements: [s1, s2, s3], options: { A, B, C, D }, correct_option }`. Backend renders the displayed stem as: _"Consider the following statements about {topic}: {s1}, {s2}, {s3}. Which of the above statements is/are true?"_ The four `options` values must each be drawn (without repetition) from the **allowed set** `{ "1", "2", "3", "1+2", "2+3", "1+3", "All of the above", "None" }`. Exactly one option is marked correct via `correct_option`.
4. **Generate** → backend call.

**Generation Backend**:

* OpenRouter API. Model resolution per §22B (OpenRouter Q\&A Service).
* Only the selected news items' headlines + briefs are sent in the prompt — never the full document.
* OpenAI-style **Structured Outputs** mode required. Backend supplies the JSON Schema for the requested question type(s); the response must conform.
* Validation chain on response: parse JSON → schema validate → if any failure, retry up to **3 times on the same model** → on continued failure, iterate through the configured **fallback chain of 4 models** → if all 8 attempts fail, surface error in the wizard with a "Retry" action.
* Spend guardrails: see §22B.4 (per-org monthly token cap + per-request cap).

**Review & Acceptance**:

* Validated Q\&A items appear in the wizard as editable cards. Teacher can edit text/options/correct-answer of any item before acceptance.
* Teacher clicks "Accept Q\&A" → items written to the document JSON under the `qa` node (§11.1).
* Teacher returns to the Send-to-Editor confirmation and proceeds normally.

**Skip Behaviour**:

* "Skip for now" button at any wizard step.
* No Q\&A node is written. Document proceeds without Q\&A. Pipeline is **never halted** by the absence of Q\&A.
