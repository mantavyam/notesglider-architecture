# Document Structure

## SECTION 11 — NEWSLETTER JSON SCHEMA (COMPLETE)

### 11.1 Full Document JSON Structure

This is the canonical data structure produced by the Lexical.dev editor DOM export after transformation. The backend reads this JSON to drive all downstream processes (PDF, PPTX, Mindmap, Compilation aggregation).

> **Production Note**: The `/* */` comments throughout this schema are JSON5-compatible. Strip them using the `strip-json-comments` npm package or a JSON5 parser before feeding into any standard JSON parser in production.

```json
{
  "Newsletter": {
    "Date": "DD-MM-YY",
    "categories": [
      {
        "category": "SCIENCE-&-TECHNOLOGY",
        "atomic_uid": "010326-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-a1b2c3d4",
        /* Backend-generated. Immutable once assigned. Format: DDMMYY-Cat-{CATEGORY_NAME}-{doc-id}-{uid} */
        "News-Items": [
          {
            "Headline": "India Launches Its Most Powerful AI Supercomputer",
            "atomic_uid": "010326-News-IndiaLaunc-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-e5f6g7h8",
            /* Backend-generated. Immutable once assigned. Format: DDMMYY-News-{headline[:10]}-Cat-{CATEGORY_NAME}-{doc-id}-{uid} */
            "images": {
              "thumbnail": {
                "url": "https://res.cloudinary.com/notesglider/...",
                "atomic_uid": "010326-thumb-Img-News-IndiaLaunc-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-i9j0k1l2",
                /* Format: DDMMYY-thumb-Img-News-{headline[:10]}-Cat-{CATEGORY_NAME}-{doc-id}-{uid} */
                "crop": null
                /* Non-destructive crop transform. null when uncropped. When applied: { x: number, y: number, w: number, h: number, applied_at: ISO8601 }. Frozen at document lock. See §10.8. */
              },
              "reference-images": [
                {
                  "url": "https://res.cloudinary.com/notesglider/...",
                  "atomic_uid": "010326-ref-1-Img-News-IndiaLaunc-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-m3n4o5p6",
                  /* Format: DDMMYY-ref-{index}-Img-News-{headline[:10]}-Cat-{CATEGORY_NAME}-{doc-id}-{uid} */
                  "crop": null,
                  "origin": "inline-paste"
                  /* "inline-paste" (default for ref images via §10.9) | "upload" (explicit upload, rare for ref) */
                },
                {
                  "url": "https://res.cloudinary.com/notesglider/...",
                  "atomic_uid": "010326-ref-2-Img-News-IndiaLaunc-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-q7r8s9t0",
                  "crop": { "x": 100, "y": 50, "w": 800, "h": 600, "applied_at": "2026-03-01T11:22:33.000Z" },
                  "origin": "inline-paste"
                }
              ]
            },
            "Brief": {
              /* Lexical serialised JSON tree. Replaces the v4 plain-string Brief. Image nodes appear inline at the position pasted by the user. atomic_uid on each image node matches the corresponding entry in reference-images. See §10.9. */
              "root": {
                "type": "root",
                "children": [
                  { "type": "paragraph", "children": [{ "type": "text", "text": "Full paragraph text of the news item before the chart..." }] },
                  { "type": "image", "src": "https://res.cloudinary.com/notesglider/...", "atomic_uid": "010326-ref-1-Img-News-IndiaLaunc-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-m3n4o5p6", "upload_state": "complete" },
                  { "type": "paragraph", "children": [{ "type": "text", "text": "Continuation after the chart..." }] },
                  { "type": "image", "src": "https://res.cloudinary.com/notesglider/...", "atomic_uid": "010326-ref-2-Img-News-IndiaLaunc-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-q7r8s9t0", "upload_state": "complete" }
                ]
              }
            }
          },
          {
            "Headline": "NASA Europa Clipper Sends First Signal from Jupiter",
            "atomic_uid": "010326-News-NASAEuropa-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-u1v2w3x4",
            "images": {
              "thumbnail": {
                "url": "https://res.cloudinary.com/notesglider/...",
                "atomic_uid": "010326-thumb-Img-News-NASAEuropa-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-y5z6a7b8",
                "crop": null
              },
              "reference-images": []
              /* Empty array when Brief has no inline images. Derived from Brief Lexical tree by backend. */
            },
            "Brief": { "root": { "type": "root", "children": [{ "type": "paragraph", "children": [{ "type": "text", "text": "Full paragraph text..." }] }] } }
          },
          {
            "Headline": "Global Quantum Internet Standards Finalised by IEEE",
            "atomic_uid": "010326-News-GlobalQuan-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-c9d0e1f2",
            "images": null,
            /* null when no thumbnail AND no inline reference images in Brief */
            "Brief": { "root": { "type": "root", "children": [{ "type": "paragraph", "children": [{ "type": "text", "text": "Full paragraph text..." }] }] } }
          }
        ]
      },
      {
        "category": "GEOPOLITICS",
        "atomic_uid": "010326-Cat-GEOPOLITICS-NL-20260301-0001-g3h4i5j6",
        "News-Items": [
          /* Additional news items for this category — each must include atomic_uid on the news item and on each image */
        ]
      }
    ]
  },
  "qa": {
    /* Optional. Present only when at least one Q&A item has been accepted (Teacher via §10.10 wizard, OR Editor/Sub-member via late-add §13.6). Absent or null when no Q&A. */
    "generated_by_role": "teacher",
    /* "teacher" | "editor" | "sub-member" — actor who accepted the items. */
    "generated_at": "2026-03-01T15:45:00.000Z",
    "model_used": "openai/gpt-4o-mini",
    /* OpenRouter model slug that succeeded. Useful for audit and re-generation parity. */
    "items": [
      {
        /* --- TYPE 1: Subjective → Straightforward --- */
        "qa_id": "QA-NL-20260301-0001-001",
        /* BACKEND-ASSIGNED ONLY. Sequential per document. Format: QA-{doc-id}-{3-digit seq}.
           The AI model NEVER produces qa_id — it is appended server-side post-validation by the §22B.3.D pipeline. */
        "source_atomic_uids": [
          "010326-News-IndiaLaunc-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-e5f6g7h8"
        ],
        /* BACKEND-RESOLVED ONLY. Atomic UIDs of news items selected as input for this question.
           The model returns ephemeral `source_refs` (e.g. ["n1"]) per §22B.3.A; backend swaps refs → atomic_uids via the request-scoped lookup before persistence.
           Used for traceability and aggregation linkage (Compilation/Magazine Q&A merging). */
        "type": "subjective.straightforward",
        "statement": "What is the peak performance of India's new AI supercomputer?",
        "answer": "210 petaflops"
      },
      {
        /* --- TYPE 2: Objective → Direct (MCQ with 4 options, exactly one correct) --- */
        "qa_id": "QA-NL-20260301-0001-002",
        "source_atomic_uids": [
          "010326-News-NASAEuropa-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-u1v2w3x4"
        ],
        "type": "objective.direct",
        "statement": "Which planet's moon is NASA's Europa Clipper mission designed to study?",
        "options": {
          "A": "Mars",
          "B": "Jupiter",
          "C": "Saturn",
          "D": "Neptune"
        },
        /* Fixed-key map A–D. All four keys are required. Values are the answer text. */
        "correct_option": "B"
        /* Must be one of "A" | "B" | "C" | "D". Validator rejects any other value. */
      },
      {
        /* --- TYPE 3: Objective → Statement Analysis ---
           Stem fixed-form: "Consider the following statements about {topic}: {s1}, {s2}, {s3}. Which of the above statements is/are true?"
           Backend assembles the displayed stem from `topic` + `statements`.
           `options` must be exactly 4 entries chosen from the allowed set:
             { "1", "2", "3", "1+2", "2+3", "1+3", "All of the above", "None" }
           with exactly one marked correct. */
        "qa_id": "QA-NL-20260301-0001-003",
        "source_atomic_uids": [
          "010326-News-GlobalQuan-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-c9d0e1f2"
        ],
        "type": "objective.statement_analysis",
        "topic": "the IEEE Global Quantum Internet Standards",
        "statements": [
          "The IEEE standards define a uniform protocol stack for quantum key distribution.",
          "The standards are binding on all member states of the United Nations.",
          "The standards include interoperability requirements for entanglement-based networks."
        ],
        /* Exactly 3 statements required. */
        "options": {
          "A": "1",
          "B": "1+3",
          "C": "2+3",
          "D": "All of the above"
        },
        /* Exactly 4 options. Each value drawn (without repeat) from the allowed set listed above. */
        "correct_option": "B"
      }
    ]
  },
  "ads-graphics": {
    "ads-graphic-1": {
      "image": "https://res.cloudinary.com/notesglider/ads/...",
      "alt-text": "Course ad banner",
      "full-page": false,
      "target-url": "https://example.com/courses/fullstack",
      "caption": "Limited seats available — Register Now"
    },
    "ads-graphic-2": {
      /* Additional ad graphics if configured for this document */
    }
  },
  "metadata": {
    /* --- Identity --- */
    "doc-id": "NL-20260301-0001",
    "doc-version": "1.0.0",
    "doc-type": "Newsletter",
    "doc-status": "draft",
    /* draft | review | approved | published | archived */
    "doc-slug": "newsletter-01-03-26",

    /* --- Ownership --- */
    "created-by": {
      "user-id": "USR-001",
      "user-name": "Teacher Full Name",
      "user-role": "teacher"
    },
    "editor-id": "EDIT-001",
    /* Tenant identifier — all queries must scope to this value */
    "created-at": "2026-03-01T10:00:00.000Z",

    /* --- Modification Tracking --- */
    "last-modified-by": {
      "user-id": "USR-001",
      "user-name": "Teacher Full Name",
      "user-role": "teacher"
    },
    "last-modified-at": "2026-03-01T16:42:00.000Z",

    /* --- Version Control --- */
    "version-history": [
      {
        "version": "1.0.0",
        "modified-by": "USR-001",
        "modified-at": "2026-03-01T10:00:00.000Z",
        "change-summary": "Initial draft created"
      },
      {
        "version": "1.1.0",
        "modified-by": "USR-001",
        "modified-at": "2026-03-01T14:20:00.000Z",
        "change-summary": "Added SCIENCE category, 3 news items"
      }
    ],

    /* --- Lock State --- */
    "is-locked": false,
    /* true once submitted to Editor queue */
    "locked-at": null,
    "locked-by": null,

    /* --- Pipeline SLA Tracking --- */
    "submitted-to-editor-at": null,
    "stage1-sla-deadline": null,
    /* submitted-to-editor-at + 2 hours */
    "stage1-approved-by-editor-at": null,
    "pdf-generation-triggered-at": null,
    "stage2-sla-deadline": null,
    /* pdf-generation-triggered-at + 2 hours */
    "stage2-approved-by-editor-at": null,
    "delivered-to-teacher-at": null,
    "sla-paused": false,
    "sla-pause-reason": null,
    /* e.g. "weasyprint_failure" */

    /* --- Revision Tracking --- */
    "revision-count": 0,
    "revision-history": [],
    /* Array of { flagged-at, flagged-by, reason, screenshots: [ { screenshot_id, page_range, mime: "image/png", b64_size_bytes, captured_at } ], resolved-at }.
       Screenshot binary content is NOT inlined here — it is stored in the revision_screenshots table (§29) as base64 PNG. revision-history holds the metadata pointers only.
       See §13.3.4A for the mandatory screenshot wizard. */

    /* --- Review & Approval --- */
    "reviewed-by": null,
    /* Populated with Editor's user-id when they review */
    "reviewed-at": null,
    "approved-by": null,
    "approved-at": null,
    "review-notes": null,

    /* --- Publishing --- */
    "published-by": null,
    "published-at": null,
    "scheduled-publish-at": null,
    "unpublish-at": null,
    "publish-channel": ["web", "email", "mobile"],

    /* --- YouTube Link --- */
    "video-url": null,
    /* Optional YouTube video URL attached to this Newsletter */
    "video-metadata": null,
    /* Optionally populated via server-side public URL parse: { title, thumbnail, duration, channel } — no YouTube API key required */

    /* --- Translation --- */
    "is-translated": false,
    "translations": [],
    /* Array of { language-code, language-name, doc-id (linked parallel document) } */
    "translation-disabled-for": [],
    /* e.g. ["mindmap"] if translation is disabled for this doc type */

    /* --- Localization --- */
    "locale": "en-IN",
    "timezone": "Asia/Kolkata",
    "region": "IN",

    /* --- Content Stats --- */
    "total-categories": 0,
    "total-news-items": 0,
    "total-images": 0,
    "total-reference-images": 0,
    "total-pages-generated": null,
    /* Populated by WeasyPrint after PDF generation — used for billing */

    /* --- Ad Graphics --- */
    "ads-configured": false,
    /* true if any ads-graphics are present */

    /* --- Templates --- */
    "header-template-id": null,
    /* Reference to Teacher's configured header template */
    "footer-template-id": null,
    /* Reference to Teacher's configured footer template */

    /* --- Image Failure Log --- */
    "image-failures": [],
    /* Array of { news-item-headline, failure-reason, placeholder-generated: true/false } */

    /* --- Billing --- */
    "billable-page-count": null,
    /* Set after WeasyPrint generation. One page = one billable unit. */
    "billing-cycle-id": null,
    /* Reference to the billing cycle this document's pages are charged in */

    /* --- Access & Permissions --- */
    "visibility": "internal",
    /* internal | public | restricted */
    "access-roles": ["editor", "teacher"],
    /* Only editor and teacher — no other roles exist */

    /* --- Soft Delete & Archival --- */
    "is-deleted": false,
    "deleted-by": null,
    "deleted-at": null,
    "is-archived": false,
    "archived-at": null,
    "auto-delete-date": null,
    /* Computed from Teacher's retention policy setting */

    /* --- Google Drive Sync --- */
    "drive-sync-status": "pending",
    /* pending | synced | failed */
    "drive-raw-path": null,
    "drive-final-path": null,
    "drive-sync-failures": [],
    /* Log of any Drive sync failures for this document */

    /* --- System --- */
    /* --- Q&A --- */
    "has-qa": false,
    /* false or true, Mirrors presence of the top-level "qa" node. Query convenience flag. */
    "qa-item-count": 0,
    "qa-late-added": false,
    /* true when Q&A was accepted post-Stage-3 delivery and triggered the §13.6 regen path. */

    "schema-version": "5.0.0",
    "source-system": "notesglider-cms",
    "environment": "production",
    /* development | staging | production */
    "tags": [],
    "notes": null
  }
}
```

### 11.2 Schema Notes for Developer

* The `doc-id` format is: `NL-YYYYMMDD-{4-digit sequence}` for Newsletters. Adjust prefix for other types: `CM-` for Compilation, `MG-` for Magazine, `MM-` for Mindmap.
* `total-pages-generated` and `billable-page-count` are set by the backend **after** WeasyPrint completes PDF generation. These values must not be set before that point.
* `org_id` must be present on every document record. It is the **primary tenant isolation key** for all queries. `editor_id` is denormalised for convenience in Editor queue queries but is not the authoritative isolation boundary.
* `image-failures` array is populated server-side during image upload processing and surfaced to the Editor in the review queue UI.
* `sla-paused` is set to `true` by the backend when a WeasyPrint failure escalation occurs. The SLA deadline timers must respect this flag — do not auto-continue while this is `true`.
* `translation-disabled-for` is an array of document type strings where translation has been disabled. Check this array before invoking the Google Translate API for a given operation.
* All entities (news categories, news items, images) must include their `atomic_uid` in the JSON schema. See Section 11A for UID format specification.
* `atomic_uid` values are **always backend-generated** — never set client-side. The backend checks the `atomic_uid_log` table for collisions before assignment.
* `atomic_uid` values are **immutable** — once assigned they are never changed, even if the entity content is edited.
* `thumbnail` is an **object** `{ url, atomic_uid, crop }`, not a bare URL string. `reference-images` is an **array** of `{ url, atomic_uid, crop, origin }` objects, not a keyed map. Both fields may be `null` as described in the image configuration table in Section 10.3.
* `crop` is `null` when the image has not been cropped. When cropped, it is `{ x, y, w, h, applied_at }` (pixels relative to the original Cloudinary asset). See §10.8 for non-destructive crop semantics. The crop is **frozen** at document lock (`is-locked = true`) and the reset action becomes unavailable.
* The `atomic_uid` for each image entity is also injected into Cloudinary MIME-level metadata (IPTC/XMP) and into the `data-uid` attribute of the corresponding HTML element during WeasyPrint generation. See Sections 11A.5 and 11A.6.
* **`Brief` schema change (v5)**: `Brief` is now a **Lexical serialised JSON tree**, no longer a plain string. Image nodes appear inline at the paste position. Each inline image node carries `atomic_uid` and `upload_state` (`pending` | `complete` | `failed`). The `reference-images` array is **derived metadata**, built backend-side by walking each Brief tree in document order. Renderers (WeasyPrint, PptxGenJS client-side, HTML export) walk the Brief tree directly and preserve inline order.
* **Q\&A `qa` node** is optional. When absent, the document has no associated Q\&A. When present, each item carries `source_atomic_uids` linking back to the news item(s) used as input — required for aggregation Q\&A merging in Compilations / Magazines. See §10.10 and §13.6.
