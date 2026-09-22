---
icon: google-drive
---

# Drive API Integration

## SECTION 25 — GOOGLE DRIVE INTEGRATION

### 25.0 Google Drive Sync is OPTIONAL (System-Wide Policy)

Google Drive integration is **optional**. Notesglider is **fully functional without it**.

* **Toggle**: `organisations.drive_sync_enabled` (boolean, default `false`). Configurable by the Teacher (Organisation owner) only. Surfaced in Teacher Dashboard → Organisation Settings → Integrations → "Google Drive".
* **When disabled (default)**: every operation that would otherwise have written to Drive is **silently skipped**. The system serves all storage from the primary stack:
  * Document `content_json` and versions → Neon DB (§24).
  * Images → Cloudinary (§26).
  * PDFs → backend object store (the `pdf_outputs` records and the underlying file store).
  * PPTX → client-side download only; no backend or Drive copy is required (§17.1).
  * Reveal.js print-to-PDF → backend auto-persisted copy at `event_type = "delivered"` (§17.2).
  * Full-AY archive ZIPs (§26A) → if Drive is disabled, the archive is written to the backend object store under `org-archives/<org_id>/<academic_year>.zip` and the Teacher downloads it from there; the same HMAC manifest and recovery flow applies.
  * Notifications, billing, Q\&A, translation, mindmap, aggregations — all unaffected.
* **When enabled**: the Drive sync behaviour described in §25.1–§25.4 activates, in addition to the primary stack. Drive is a **mirror**, never the source of truth.
* **No feature is gated behind Drive being enabled.** A document delivered to a Teacher whose Org has Drive disabled is fully retrievable from the in-app UI (download from the dashboard, in-app PDF viewer, email link to a signed backend URL).
* **Switching the toggle**:
  * `false → true`: a backfill job sweeps all documents in the Org's current academic year and uploads them to Drive in dependency order (folder bootstrap first, then assets). Backfill is rate-limited and runs as a background job; the toggle activation does not block the UI.
  * `true → false`: Drive content is **not deleted** from the Teacher's Drive (the user owns their Drive). No further writes occur. A confirmation dialog warns: _"Disabling Drive sync will stop further uploads. Existing files in your Drive will remain. You can re-enable sync at any time."_
* **Sub-member impact**: sub-members inherit the Org's toggle state — they cannot override it.
* **Editor impact**: when an assigned Editor's Org has Drive disabled, the Editor's Drive mirror (§25.1) is also skipped. Editors continue to access documents through the in-app review queue.

> **Implementation note for the developer**: every Drive write call must go through a single `DriveSyncService.maybeSync()` wrapper that no-ops when the Org's `drive_sync_enabled = false`. There must be no inline `drive_client.upload(...)` calls scattered across feature code.

### 25.1 Drive Ownership Architecture

* Each Teacher connects their **own personal Google Drive** via Google OAuth during onboarding.
* The app provisions the full PRD-defined folder structure automatically within the Teacher's Drive upon account approval.
* All finalised documents are **simultaneously synced to the Editor's Google Drive** in a mirrored folder structure organised by a Teacher identifier (e.g. subfolder named with Teacher's name or ID).
* Sub-members inherit their parent Teacher's Drive connection — all sub-member actions write to the same Teacher Drive, not separate Drives.

### 25.2 Drive Folder Structure (Complete)

The following is the canonical folder structure provisioned in both the Teacher's Drive and the Editor's Drive:

```json
{
  "YYYY": {
    "MMMYY": {
      "DAILY": {
        "RAW": {
          "DD-MM-YY": {
            "NEWSLETTER-DD-MM-YY.md": null,
            "NEWSLETTER-DD-MM-YY.html": null,
            "NEWSLETTER-DD-MM-YY.gdoc": null,
            "IMG-DD-MM-YY": [
              "n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension"
            ]
          }
        },
        "FINAL": {
          "DD-MM-YY": {
            "NEWSLETTER-DD-MM-YY.pdf": null,
            "MINDMAP-DD-MM-YY.pdf": null
          }
        }
      },
      "WEEKLY": {
        "RAW": {
          "WK1-MMMYY": {
            "COMPILATION-WK1-MMMYY.md": null,
            "COMPILATION-WK1-MMMYY.html": null,
            "IMG-WK1-MMMYY": {
              "CATEGORY1": ["n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension"],
              "CATEGORYn": ["n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension"]
            }
          },
          "WK2-MMMYY": { },
          "WK3-MMMYY": { },
          "WK4-MMMYY": { }
        },
        "FINAL": {
          "WK1-COMPILATION-MMMYY.pdf": null,
          "WK2-COMPILATION-MMMYY.pdf": null,
          "WK3-COMPILATION-MMMYY.pdf": null,
          "WK4-COMPILATION-MMMYY.pdf": null
        }
      },
      "MONTHLY": {
        "RAW": {
          "MAGAZINE-MMMYY.md": null,
          "MAGAZINE-MMMYY.html": null,
          "IMG-MMMYY": {
            "CATEGORY1": ["n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension"],
            "CATEGORYn": ["n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension"]
          }
        },
        "FINAL": {
          "MAGAZINE-MMMYY.pdf": null
        }
      },
      "QUARTERLY": {
        "RAW": {
          "Qn-YYYY": {
            "COLLECTION-Qn-YYYY.md": null,
            "COLLECTION-Qn-YYYY.html": null,
            "IMG-Qn-YYYY": {
              "CATEGORY1": ["n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension"],
              "CATEGORYn": ["n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension"]
            }
          }
        },
        "FINAL": {
          "COLLECTION-Q1-YYYY.pdf": null,
          "COLLECTION-Q2-YYYY.pdf": null,
          "COLLECTION-Q3-YYYY.pdf": null,
          "COLLECTION-Q4-YYYY.pdf": null
        }
      }
    },
    "BIANNUAL": {
      "RAW": {
        "COMPENDIUM-H1-YYYY": {
          "COMPENDIUM-H1-YYYY.md": null,
          "COMPENDIUM-H1-YYYY.html": null,
          "IMG-H1-YYYY": {
            "CATEGORY1": ["n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension"],
            "CATEGORYn": ["n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension"]
          }
        },
        "COMPENDIUM-H2-YYYY": {
          "COMPENDIUM-H2-YYYY.md": null,
          "COMPENDIUM-H2-YYYY.html": null,
          "IMG-H2-YYYY": {
            "CATEGORY1": ["n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension"],
            "CATEGORYn": ["n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension"]
          }
        }
      },
      "FINAL": {
        "COMPENDIUM-H1-YYYY.pdf": null,
        "COMPENDIUM-H2-YYYY.pdf": null
      }
    },
    "ANNUAL": {
      "RAW": {
        "YEARBOOK-YYYY": {
          "YEARBOOK-YYYY.md": null,
          "YEARBOOK-YYYY.html": null,
          "IMG-YYYY": {
            "CATEGORY1": ["n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension"],
            "CATEGORYn": ["n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension"]
          }
        }
      },
      "FINAL": {
        "YEARBOOK-YYYY.pdf": null
      }
    }
  }
}
```

**Notation key**:

* `YYYY` = 4-digit year (e.g. `2026`)
* `MMMYY` = Month abbreviation + 2-digit year (e.g. `MAR26`)
* `DD-MM-YY` = Day-Month-Year (e.g. `01-03-26`)
* `WKn` = Week serial number (1–4)
* `Qn` = Quarter serial number (1–4)
* `Hn` = Half-year serial number (1–2)
* `n` = Order index of image appearance
* `HEADLINE[:10]` = First 10 characters of the news headline

**HTML versions**: Every `.md` file is generated alongside a corresponding `.html` file. The `.html` version preserves rich text formatting and can be viewed directly in a browser as a webpage experience without requiring a markdown renderer. When stored in the database, HTML files reference Cloudinary CDN URLs for images. When exported via the HTML webpage export (Section 17.7), images are decoupled from CDN and linked to local asset files.

**Category Extraction documents**: Cherry-picked category extraction documents are stored in the same folder structure as their source aggregation type. For example, a Science & Technology extraction from a Weekly Compilation is stored under `WEEKLY/RAW/WK1-MMMYY/extraction-SCIENCE_AND_TECHNOLOGY-COMPILATION-WK1-MMMYY.md` (and `.html`).

### 25.3 Editor's CRUD Permissions on Drive

* The Editor has **full CRUD permissions** on the Drive folder structure for their entire tenant.
* Editor can create, read, update, and delete any file or folder in the structure.
* This includes the Teacher's Drive folders (since the Editor's OAuth token is scoped appropriately).

### 25.4 Google Docs & HTML Mirror

* When RAW document sync is enabled, the document content is saved in three formats:
  * A `.md` file (for programmatic use and raw editing)
  * An `.html` file (for browser-viewable webpage experience and rich text formatting preservation — see Section 17.7 for export details)
  * A Google Docs file (`.gdoc`) in the same Drive folder — for human-readable online preview via the Google ecosystem at any future time.
* The `.html` file stored in the database and Drive contains Cloudinary CDN URLs for images. When exported via the HTML webpage export (Section 17.7), images are decoupled from CDN and linked to local asset files.
* Each entity in the `.html` includes its `atomic_uid` as a `data-uid` attribute (see Section 11A.6).
