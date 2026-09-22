# Template System

## SECTION 19 — TEMPLATE SYSTEM

### 19.1 Overview

The Template System is a **micro-app-like functionality** (similar to beefree.io email template builder) that allows authorised users to create, manage, and assign reusable templates for any part of their PDF documents. Templates are **Organisation-wide entities** — not personal documents of individual roles. All authorised users (Teacher, Editor, Sub-member) see a single unified template library from their respective dashboards.

**Template types are not limited to headers and footers.** The system supports:

* Header templates
* Footer templates
* Full-page templates (e.g. a Preface page for Monthly Magazine with pre-written text)
* Custom individual page templates (any purpose — title pages, section dividers, acknowledgment pages, etc.)

**Who can create/edit templates:**

* **Teacher**: CRUD templates for their own Organisation
* **Editor**: CRUD templates on behalf of any Organisation mapped to them
* **Sub-member**: View-only access to Organisation templates (cannot create/edit unless Teacher grants explicit template editing permission in the future)

**Unified view**: The templates are created for an Organisation-wide scope. When a Teacher, Sub-member, or Editor views the template library, they see the same set of templates belonging to that Organisation.

### 19.2 Template Builder (Drag-and-Drop Micro App)

The template builder provides a block-based HTML editor with full drag-and-drop capabilities:

**Pre-made Templates:**

* A library of pre-made branded template layouts (e.g. Channel Branding, Minimal, Educational, Magazine Preface, Certificate, etc.).
* User selects a template as a starting point or starts from a blank canvas.

**Template Configuration:**

| Configuration Step  | Description                                                                                                                                                                    |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Set Document Type   | Select which document type(s) this template applies to: Newsletter, Compilation, Magazine, Mindmap, Quarterly Collection, Bi-Annual Compendium, Annual Yearbook (multi-select) |
| Select Canvas Size  | Choose from a range of pre-defined layout sizes (A4, Letter, Custom dimensions)                                                                                                |
| Design with Builder | Drag-and-drop block-based builder for visual template composition                                                                                                              |
| Set Applicability   | Map template inclusion to respective documents in the pipeline's dynamic template ingestion process                                                                            |

**Custom Block Types Available:**

| Block Type        | Description                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| Logo/Image        | Upload or URL-reference a brand logo                                                                    |
| Text              | Editable text block with font/size/colour controls. Can contain pre-written text (e.g. Preface content) |
| Link / Hyperlink  | Clickable hyperlink with custom label — applicable on any element                                       |
| Social Icon       | Platform icon (YouTube, Instagram, Twitter, etc.) with link                                             |
| Dynamic Date      | Auto-inserts document date at render time                                                               |
| Dynamic Title     | Auto-inserts document title at render time                                                              |
| Dynamic Video URL | Auto-inserts the attached YouTube video URL                                                             |
| Page Number       | Auto-inserts current page number                                                                        |
| Divider Line      | Visual separator                                                                                        |
| Ad Banner         | References an ad banner from the Organisation's ad library                                              |
| Spacer            | Empty space block with configurable height                                                              |
| Table             | Configurable table block with rows, columns, and cell content                                           |

**Customisation Controls:**

* Drag-and-drop block arrangement within the template canvas.
* Background colour, border, padding controls per block.
* Font family, size, weight, colour per text block.
* Alignment (left, centre, right) per block.
* Hyperlinks on any interactive element.

### 19.3 Template Types & Use Cases

| Template Type   | Use Case Example                                                                         |
| --------------- | ---------------------------------------------------------------------------------------- |
| Header          | Branded header with logo, social icons, dynamic date — appears at top of PDF pages       |
| Footer          | Footer with page numbers, copyright text, video URL — appears at bottom of PDF pages     |
| Preface Page    | A full-page template with pre-written text, mapped to Monthly Magazine as the first page |
| Title Page      | A branded cover page with dynamic document title and date                                |
| Section Divider | A page inserted between major sections of a Compilation or Magazine                      |
| Acknowledgment  | A thank-you page inserted at the end of Annual Yearbook                                  |
| Custom Page     | Any user-defined page layout with mixed content                                          |

### 19.4 Scope Controls (Per Template)

Each saved template has scope configuration that determines where and when it is applied:

| Scope Dimension       | Options                                                                                                                                  |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **Template Type**     | Header / Footer / Full Page / Custom Page                                                                                                |
| **Document Type**     | Multi-select: Newsletter, Compilation, Magazine, Mindmap, Quarterly Collection, Bi-Annual Compendium, Annual Yearbook (any combination)  |
| **Page Application**  | First page only / Last page only / All pages / Specific page range (start–end) / All pages except first / Before content / After content |
| **Optional Elements** | Include/exclude page numbers (checkbox), Include/exclude date stamp (checkbox)                                                           |

### 19.5 Multiple Templates & Assignment

* Users can save multiple templates of any type per Organisation.
* In settings, users assign active templates per document type. Multiple templates can be active simultaneously (e.g. one header template, one footer template, and one preface page template all active for Magazine documents).
* Template changes apply prospectively — already-generated PDFs are not retroactively re-rendered.
* Templates created by the Editor on behalf of an Organisation are visible to and usable by the Teacher and Sub-members of that Organisation.

### 19.6 Template Injection During PDF Generation

* During WeasyPrint rendering, the backend:
  1. Looks up the Organisation's active templates for the document type being rendered.
  2. Renders each template's HTML, substituting all dynamic placeholders (`{date}`, `{doc-title}`, `{video-url}`, `{page-number}`, etc.) with actual values.
  3. For templates: applies the rendered HTML to the WeasyPrint CSS `@page` rules for the correct page range based on template scope configuration.
  4. For full-page/custom-page templates: inserts the rendered HTML at the configured position (before content, after content, at specific page index).
  5. Generates the PDF with all templates integrated.

### 19.7 Mindmap PDF Template Injection

* When exporting a Mindmap to PDF:
  * Check if templates are configured for this Organisation and the Mindmap document type.
  * If yes: Mind Elixir exports SVG → SVG is embedded into a WeasyPrint HTML/CSS canvas → template HTML is injected around the SVG (headers, footers, wrapper pages) → WeasyPrint renders to PDF.
  * If no: SVG is converted directly to PDF by WeasyPrint — clean single-image PDF output.
