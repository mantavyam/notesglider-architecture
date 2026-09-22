# Atomic UID System

## SECTION 11A — ATOMIC UID SYSTEM

### 11A.1 Purpose

The current document schema identifies entities primarily through the document-level `doc-id` in metadata, but lacks unique identifiers for atomic sub-entities such as individual news categories, news items, thumbnail images, and reference images. This creates mapping ambiguity, particularly in the following scenarios:

* **Re-generation of past documents** after Cloudinary images have been archived/deleted at the end of the academic year — without UIDs, image-to-news-item mapping relies solely on CDN URLs which become invalid post-archival.
* **Cross-document referencing** in aggregation documents (compilations, magazines, quarterly collections, etc.) where news items from multiple newsletters must be uniquely identified.
* **Audit and traceability** — linking specific content units to their creation context.

### 11A.2 UID Format Specification

All UIDs are prefixed with the document date to ensure temporal uniqueness and prevent collisions across academic years.

| Entity          | UID Format                                                                            |
| --------------- | ------------------------------------------------------------------------------------- |
| Base UID        | `DDMMYYYY-{uid-string}`                                                               |
| News Category   | `DDMMYY-Cat-{category-name}-{documentID}-{UID}`                                       |
| News Item       | `DDMMYY-News-{news-name[:10]}-Cat-{category-name}-{documentID}-{UID}`                 |
| Thumbnail Image | `DDMMYY-thumb-Img-News-{news-name[:10]}-Cat-{category-name}-{documentID}-{UID}`       |
| Reference Image | `DDMMYY-ref-{index}-Img-News-{news-name[:10]}-Cat-{category-name}-{documentID}-{UID}` |

Where:

* `DDMMYY` = 6-digit date of the document (e.g. `010326` for March 1, 2026)
* `DDMMYYYY` = 8-digit date of the document (e.g. `01032026` for March 1, 2026)
* `{uid-string}` = A unique random string (e.g. UUID v4 short segment or nanoid)
* `{category-name}` = Sanitised category name (uppercase, spaces replaced with underscores)
* `{news-name[:10]}` = First 10 characters of the news headline (sanitised, stripped of special characters)
* `{documentID}` = The document's `doc-id` (e.g. `NL-20260301-0001`)
* `{index}` = Order index of the reference image within the news item (1, 2, 3...)
* `{UID}` = The base UID string for uniqueness

### 11A.3 UID Generation Rules

1. **Backend generates all UIDs** — UIDs are never generated client-side.
2. Backend maintains a **UID log table** (`atomic_uid_log`) that stores every generated UID to ensure uniqueness.
3. Before assigning a UID, the backend checks the log table for collisions. If a collision is detected, a new UID is generated.
4. The `DDMMYYYY` prefix provides a first-pass deduplication layer — UIDs from different dates cannot collide even if the random segment is identical.
5. UIDs are **immutable** once assigned — they are never changed, even if the entity is edited.

### 11A.4 Schema Integration

UIDs are embedded in the document JSON schema at the entity level:

```json
{
  "news-categories": [
    {
      "category-name": "Science & Technology",
      "atomic_uid": "010326-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-a1b2c3d4",
      "news-items": [
        {
          "headline": "New Discovery in Physics",
          "atomic_uid": "010326-News-NewDiscove-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-e5f6g7h8",
          "images": {
            "thumbnail": {
              "url": "https://res.cloudinary.com/.../thumb.jpg",
              "atomic_uid": "010326-thumb-Img-News-NewDiscove-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-i9j0k1l2"
            },
            "reference-images": [
              {
                "url": "https://res.cloudinary.com/.../ref1.jpg",
                "atomic_uid": "010326-ref-1-Img-News-NewDiscove-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-m3n4o5p6"
              }
            ]
          }
        }
      ]
    }
  ]
}
```

### 11A.5 Image MIME-Level Metadata

When images are uploaded to Cloudinary, the `atomic_uid` is injected into the image's MIME-level metadata (EXIF/IPTC/XMP fields) so that even if the image is downloaded or extracted from the system, its origin can be traced:

* **IPTC Caption**: `atomic_uid` value
* **XMP Description**: `atomic_uid` value
* **Cloudinary metadata tag**: `atomic_uid` value

This ensures proper image-to-news-item mapping even when images are served from Google Drive archives (post-Cloudinary deletion) where only the file itself is available.

### 11A.6 UID in HTML Output

When generating `.html` versions of documents, each entity's `atomic_uid` is embedded as a `data-uid` attribute on the corresponding HTML element:

```html
<section class="news-category" data-uid="010326-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-a1b2c3d4">
  <article class="news-item" data-uid="010326-News-NewDiscove-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-e5f6g7h8">
    <img src="..." data-uid="010326-thumb-Img-News-NewDiscove-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-i9j0k1l2" />
  </article>
</section>
```
