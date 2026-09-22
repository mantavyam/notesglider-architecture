# Mindmap

## SECTION 16 — MINDMAP WORKFLOW

### 16.1 Overview

The Mindmap is a visual representation of a Newsletter's content, rendered using Mind Elixir Core. It is generated as the **optional final step** of the Newsletter pipeline, but can also be created independently as a standalone document.

> **RBAC**: Mindmap creation and PDF generation is **Editor-only**. Teachers have no Mindmap controls, buttons, or visibility into the Mindmap generation process. Teachers only receive the final Mindmap PDF output.

### 16.2 Default Flow — Newsletter Pipeline (Optional Final Step)

This flow is triggered after the Newsletter PDF has been delivered to the Teacher and the Editor optionally continues to create a Mindmap from the same Newsletter's assets.

**Step 1 — Data Transformation**

* Backend reads the Newsletter's structured JSON.
* Transforms the Newsletter data into Mind Elixir-compatible JSON format.
* Categories become root branches; News Items become sub-nodes; Headlines become node labels.

**Step 2 — Auto-Population**

* The Mind Elixir canvas opens in the Editor's view.
* Nodes and branches are auto-populated from the transformed JSON.
* Editor sees the complete initial Mindmap layout.

**Step 3 — Editor Customisation & Approval**

* Editor can customise: node text, node colours, branch arrangement, node sizes, connections.
* Or Editor can approve as-is without modification.

**Step 4 — SLA Auto-Continuation (same rules as Newsletter pipeline)**

* The 2-hour SLA auto-continuation applies to the Mindmap step.
* If Editor takes no action within 2 hours → Mindmap auto-proceeds to PDF export.
* For revision-flagged Mindmaps → no auto-continuation; Editor must manually approve.

**Step 5 — PDF Export**

* On approval (manual or auto-continuation):
  * Mind Elixir exports a **high-resolution SVG** from the canvas.
  * **If a template is configured** for this organisation and applies to Mindmap documents: SVG is embedded into a WeasyPrint HTML/CSS canvas with the template HTML (header/footer/full-page) injected around it -> WeasyPrint renders the combined output to PDF.
  * **If no template is configured**: SVG is converted directly to PDF by WeasyPrint — clean, no wrapper.
* Mindmap PDF is saved to the database and Drive: `YYYY/MMMYY/DAILY/FINAL/MINDMAP-DD-MM-YY.pdf`

### 16.3 Standalone Flow — Independent Mindmap Creation

The Editor can create a Mindmap entirely independently of any Newsletter:

**Mode A — From Scratch**

* Editor opens the Mind Elixir canvas with a blank state.
* Creates nodes and branches manually using the Mind Elixir toolbar.
* No pre-populated data.

**Mode B — From Structured File Upload**

* Editor uploads a structured file (format: Mind Elixir native JSON, or a markdown file following the Newsletter hierarchy which the backend can transform).
* Backend processes the uploaded file and populates the Mind Elixir canvas.

Both standalone modes follow the same approval and PDF export logic as Step 3–5 above.

### 16.4 Mind Elixir Technical Notes for Developer

* Library: `mind-elixir-core` from [https://github.com/SSShooter/mind-elixir-core](https://github.com/SSShooter/mind-elixir-core)
* Documentation: [https://docs.mind-elixir.com/](https://docs.mind-elixir.com/)
* The library is JavaScript and framework-agnostic — integrate it within the Next.js frontend as a client-side component.
* SVG export is available via Mind Elixir's built-in export API.
* The SVG output must be high-resolution — test with content-dense Mindmaps (20+ nodes) to ensure quality.

### 16.5 Mindmap in Translation Pipeline

* Translation of Mindmaps can be individually disabled per Teacher in their settings.
* Check `metadata.translation-disabled-for` array — if `"mindmap"` is present, skip translation for Mindmap documents for that Teacher.
