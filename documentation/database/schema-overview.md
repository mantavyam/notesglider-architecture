---
icon: database
---

# Schema Overview

## SECTION 29 — DATABASE SCHEMA OVERVIEW

### 29.1 Core Tables (Neon / PostgreSQL via Prisma)

The following table list covers all primary data entities. This is a schema overview — the developer should implement the full Prisma schema based on these definitions. The **primary isolation key** for all tenant-scoped data is `org_id` (not `editor_id` alone). All tables containing document, user, or billing data must include `org_id`. The `editor_id` on the `organisations` table is the tenancy assignment link — queries are scoped via JOIN or predicated on `org_id IN (SELECT id FROM organisations WHERE editor_id = current_editor_id)`.

***

#### `super_admins`

Stores the Super Admin account(s). Provisioned at deployment time — no in-app registration.

| Column                             | Type                        | Notes                                                   |
| ---------------------------------- | --------------------------- | ------------------------------------------------------- |
| `id`                               | UUID (PK)                   |                                                         |
| `name`                             | String                      |                                                         |
| `email`                            | String (unique)             |                                                         |
| `password_hash`                    | String                      | bcrypt, cost >= 12                                      |
| `google_id`                        | String (nullable)           | If Google OAuth is used for Super Admin login           |
| `status`                           | Enum: `active`, `suspended` | Default: `active` on seed                               |
| `razorpay_key_id`                  | String (encrypted)          | Master Razorpay Key ID for platform billing             |
| `razorpay_key_secret`              | String (encrypted)          | Master Razorpay Key Secret for platform billing         |
| `monthly_rate_per_page`            | Decimal                     | Default: 1.00 — base rate for monthly billing           |
| `annual_rate_per_page`             | Decimal                     | Default: 0.90 — discounted rate for annual billing      |
| `billing_auto_trigger_day`         | Integer                     | Default: 5 — day of month for auto-invoice generation   |
| `billing_timezone`                 | String                      | Default: `Asia/Kolkata` — timezone for billing triggers |
| `audit_log_auto_delete_enabled`    | Boolean                     | Default: false                                          |
| `audit_log_auto_delete_after_days` | Integer (nullable)          | If auto-delete enabled, delete logs older than N days   |
| `created_at`                       | Timestamp                   |                                                         |
| `last_login_at`                    | Timestamp (nullable)        |                                                         |

> There is typically only one Super Admin record. The schema supports multiple if needed (e.g. platform co-owners), but additional records must be provisioned via script, never through the app UI.

***

#### `organisations`

The tenant container. Created by a Teacher on self-registration. Approved and mapped by the Super Admin.

| Column                       | Type                                            | Notes                                                          |
| ---------------------------- | ----------------------------------------------- | -------------------------------------------------------------- |
| `id`                         | UUID (PK)                                       | Primary isolation key for all tenant-scoped queries            |
| `name`                       | String                                          | Organisation display name (set by Teacher during registration) |
| `description`                | String (nullable)                               |                                                                |
| `region`                     | String                                          | e.g. `IN`, `US`                                                |
| `timezone`                   | String                                          | e.g. `Asia/Kolkata`                                            |
| `status`                     | Enum: `pending_approval`, `active`, `suspended` |                                                                |
| `created_by_teacher_id`      | UUID (FK → teachers.id)                         | The Teacher who created this Org                               |
| `editor_id`                  | UUID (FK → editors.id, nullable)                | Null until Super Admin assigns; set on assignment              |
| `approved_by_super_admin_id` | UUID (FK → super\_admins.id, nullable)          |                                                                |
| `approved_at`                | Timestamp (nullable)                            |                                                                |
| `rejection_reason`           | String (nullable)                               | Populated if rejected                                          |
| `suspended_at`               | Timestamp (nullable)                            |                                                                |
| `created_at`                 | Timestamp                                       |                                                                |

***

#### `editors`

Stores Editor accounts. Created by Super Admin — no self-registration.

| Column                      | Type                                   | Notes                                                        |
| --------------------------- | -------------------------------------- | ------------------------------------------------------------ |
| `id`                        | UUID (PK)                              |                                                              |
| `name`                      | String                                 |                                                              |
| `email`                     | String (unique)                        |                                                              |
| `auth_provider`             | Enum: `google`, `email`, `magic_link`  |                                                              |
| `google_id`                 | String (nullable)                      |                                                              |
| `password_hash`             | String (nullable)                      | For email/password auth                                      |
| `status`                    | Enum: `pending`, `active`, `suspended` | `pending` on creation; `active` after Super Admin authorises |
| `created_by_super_admin_id` | UUID (FK → super\_admins.id)           |                                                              |
| `authorised_at`             | Timestamp (nullable)                   | Set when Super Admin authorises the account                  |
| `created_at`                | Timestamp                              |                                                              |
| `last_login_at`             | Timestamp (nullable)                   |                                                              |
| `pdf_retry_count`           | Integer                                | Default: 3 — configurable by Super Admin                     |
| `drive_google_token`        | JSON (encrypted)                       | OAuth token for Editor's Drive                               |
| `notification_preferences`  | JSON                                   | Per-event channel preferences                                |

***

#### `teachers`

Stores Teacher (subscriber) accounts. Self-registered — no Editor invite link. Status gated by Editor authorisation after Super Admin approves the Organisation.

| Column                        | Type                                       | Notes                                                                          |
| ----------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------ |
| `id`                          | UUID (PK)                                  |                                                                                |
| `org_id`                      | UUID (FK → organisations.id)               | **Primary tenant isolation key**                                               |
| `editor_id`                   | UUID (FK → editors.id, nullable)           | Denormalised from org.editor\_id for query convenience; kept in sync           |
| `name`                        | String                                     |                                                                                |
| `email`                       | String                                     |                                                                                |
| `auth_provider`               | Enum: `google`, `email`, `magic_link`      |                                                                                |
| `google_id`                   | String (nullable)                          |                                                                                |
| `password_hash`               | String (nullable)                          |                                                                                |
| `status`                      | Enum: `pending`, `active`, `suspended`     | `pending` on self-registration; `active` after Editor authorises               |
| `authorised_by_editor_id`     | UUID (FK → editors.id, nullable)           | Set when Editor authorises the Teacher                                         |
| `authorised_at`               | Timestamp (nullable)                       |                                                                                |
| `billing_plan`                | Enum: `monthly`, `annual`                  |                                                                                |
| `billing_status`              | Enum: `current`, `grace_period`, `overdue` |                                                                                |
| `billing_overdue_since`       | Timestamp (nullable)                       |                                                                                |
| `grace_period_ends_at`        | Timestamp (nullable)                       |                                                                                |
| `drive_google_token`          | JSON (encrypted)                           | OAuth token for Teacher's Drive                                                |
| `youtube_google_token`        | JSON (encrypted)                           | OAuth token for YouTube                                                        |
| `youtube_channel_id`          | String (nullable)                          | Teacher's YouTube channel ID for custom URL-based access by Editor/Sub-member  |
| `youtube_channel_url`         | String (nullable)                          | Teacher's YouTube channel URL for custom URL-based access by Editor/Sub-member |
| `academic_year_start_month`   | Integer                                    | Default: 1 (January)                                                           |
| `aggregation_trigger_time`    | String                                     | Default: `18:00` — time of day for aggregation triggers (HH:MM in 24hr)        |
| `aggregation_timezone`        | String                                     | Default: `Asia/Kolkata` — timezone for aggregation triggers                    |
| `notification_preferences`    | JSON                                       | Per-event channel preferences                                                  |
| `clear_field_warning_enabled` | Boolean                                    | Default: true                                                                  |
| `raw_drive_sync_enabled`      | Boolean                                    | Default: false                                                                 |
| `retention_policy_months`     | Integer                                    | Min: 2, Max: 12                                                                |
| `created_at`                  | Timestamp                                  |                                                                                |

***

#### `sub_members`

Stores sub-member accounts (Teacher's team members). Pending Teacher authorisation upon registration.

| Column                      | Type                                           | Notes                                                      |
| --------------------------- | ---------------------------------------------- | ---------------------------------------------------------- |
| `id`                        | UUID (PK)                                      |                                                            |
| `org_id`                    | UUID (FK → organisations.id)                   | Primary tenant isolation key                               |
| `teacher_id`                | UUID (FK → teachers.id)                        | Parent Teacher who enrolled them                           |
| `editor_id`                 | UUID (FK → editors.id)                         | Denormalised from org; for query convenience               |
| `name`                      | String                                         |                                                            |
| `email`                     | String                                         |                                                            |
| `auth_provider`             | Enum                                           |                                                            |
| `password_hash`             | String (nullable)                              |                                                            |
| `status`                    | Enum: `pending`, `active`, `suspended`, `left` | `pending` until Teacher authorises                         |
| `authorised_by_teacher_at`  | Timestamp (nullable)                           | Set when Teacher authorises sub-member                     |
| `permissions`               | JSON                                           | `{ create: bool, edit: bool, submit: bool, export: bool }` |
| `can_receive_notifications` | Boolean                                        | Controlled by Teacher                                      |
| `notification_preferences`  | JSON                                           | Controlled by sub-member independently                     |
| `joined_at`                 | Timestamp (nullable)                           |                                                            |
| `left_at`                   | Timestamp (nullable)                           |                                                            |

***

#### `documents`

Master table for all document records (Newsletter, Compilation, Magazine, Mindmap).

| Column                      | Type                                                                                                                      | Notes                                                                                                                                          |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`                        | UUID (PK)                                                                                                                 |                                                                                                                                                |
| `doc_id`                    | String                                                                                                                    | e.g. `NL-20260301-0001`                                                                                                                        |
| `org_id`                    | UUID (FK → organisations.id)                                                                                              | **Primary tenant isolation key**                                                                                                               |
| `publication_id`            | UUID (FK → publications.id)                                                                                               | Publication stream this document belongs to                                                                                                    |
| `editor_id`                 | UUID (FK → editors.id)                                                                                                    | Denormalised from org; for queue queries                                                                                                       |
| `teacher_id`                | UUID (FK → teachers.id)                                                                                                   |                                                                                                                                                |
| `created_by_id`             | UUID                                                                                                                      | Can be teacher\_id, sub\_member\_id, or editor\_id                                                                                             |
| `created_by_role`           | Enum: `teacher`, `sub-member`, `editor`                                                                                   |                                                                                                                                                |
| `doc_type`                  | Enum: `Newsletter`, `Compilation`, `Magazine`, `Mindmap`, `Quarterly`, `BiAnnual`, `AnnualYearbook`, `CategoryExtraction` | For default publication; custom publications use `doc_type_name`                                                                               |
| `doc_type_name`             | String                                                                                                                    | Custom document type name (unique per org). For default publication, mirrors `doc_type` value. For custom publications, holds the custom name. |
| `document_date`             | Date                                                                                                                      | The target date this document is "for" (e.g. the newsletter date). Used for duplicate detection and retrospective billing.                     |
| `doc_status`                | Enum: `draft`, `review`, `approved`, `published`, `archived`                                                              |                                                                                                                                                |
| `title`                     | String                                                                                                                    | Auto-generated from date or manual                                                                                                             |
| `content_json`              | JSON                                                                                                                      | Lexical editor DOM serialised JSON                                                                                                             |
| `doc_version`               | String                                                                                                                    | Semantic version e.g. `1.0.0`                                                                                                                  |
| `is_locked`                 | Boolean                                                                                                                   | Default: false                                                                                                                                 |
| `locked_at`                 | Timestamp (nullable)                                                                                                      |                                                                                                                                                |
| `is_deleted`                | Boolean                                                                                                                   | Default: false                                                                                                                                 |
| `is_archived`               | Boolean                                                                                                                   | Default: false                                                                                                                                 |
| `auto_delete_date`          | Timestamp (nullable)                                                                                                      | Computed from retention policy                                                                                                                 |
| `template_ids`              | JSON (nullable)                                                                                                           | Array of template IDs applied to this document (FK -> templates.id)                                                                            |
| `video_url`                 | String (nullable)                                                                                                         |                                                                                                                                                |
| `video_metadata`            | JSON (nullable)                                                                                                           | YouTube metadata                                                                                                                               |
| `locale`                    | String                                                                                                                    | Default: `en-IN`                                                                                                                               |
| `timezone`                  | String                                                                                                                    | Default: `Asia/Kolkata`                                                                                                                        |
| `is_billable`               | Boolean                                                                                                                   | Default: true. See Section 23.3B for billability rules                                                                                         |
| `editor_creation_purpose`   | Enum: `proofreading`, `processing` (nullable)                                                                             | Only set for Editor-created documents. Determines `is_billable` value.                                                                         |
| `count_in_billing`          | Boolean                                                                                                                   | Default: true. Set to false for CategoryExtraction documents used for proofreading/crosschecking only                                          |
| `is_retrospective`          | Boolean                                                                                                                   | Default: false. True if document was created for a past date outside the current billing cycle                                                 |
| `is_billed`                 | Boolean                                                                                                                   | Default: false. Set to true when document has been included in a finalized invoice                                                             |
| `billing_cycle_id`          | UUID (FK → billing\_cycles.id, nullable)                                                                                  | The billing cycle this document was counted in. NULL until invoice generation assigns it                                                       |
| `billing_cleared_at`        | Timestamp (nullable)                                                                                                      | Timestamp when the document's billing was finalized (invoice paid/cleared)                                                                     |
| `source_aggregation_doc_id` | UUID (nullable)                                                                                                           | FK -> documents.id. For CategoryExtraction: the aggregation document this was extracted from                                                   |
| `extraction_categories`     | JSON (nullable)                                                                                                           | For CategoryExtraction: array of category names selected for extraction                                                                        |
| `created_at`                | Timestamp                                                                                                                 |                                                                                                                                                |
| `updated_at`                | Timestamp                                                                                                                 |                                                                                                                                                |

> **Unique constraint**: `(org_id, publication_id, doc_type_name, document_date)` — enforces duplicate document prevention (Section 9A.7).

***

#### `document_versions`

Version history records for each document.

| Column           | Type                     | Notes                               |
| ---------------- | ------------------------ | ----------------------------------- |
| `id`             | UUID (PK)                |                                     |
| `document_id`    | UUID (FK → documents.id) |                                     |
| `version`        | String                   | e.g. `1.0.0`                        |
| `content_json`   | JSON                     | Snapshot of content at this version |
| `modified_by_id` | UUID                     |                                     |
| `modified_at`    | Timestamp                |                                     |
| `change_summary` | String (nullable)        |                                     |

***

#### `document_comments`

Linear comment thread per document for communication and dispute resolution (see Section 4.11.3).

| Column        | Type                                                   | Notes                                               |
| ------------- | ------------------------------------------------------ | --------------------------------------------------- |
| `id`          | UUID (PK)                                              |                                                     |
| `document_id` | UUID (FK → documents.id)                               | The document this comment belongs to                |
| `author_id`   | UUID                                                   | The user who posted the comment                     |
| `author_role` | Enum: `super_admin`, `editor`, `teacher`, `sub-member` | Role at time of posting                             |
| `author_name` | String                                                 | Denormalized display name for rendering             |
| `message`     | Text                                                   | Comment body (plain text, max 2000 chars)           |
| `created_at`  | Timestamp                                              | Immutable — comments cannot be edited after posting |
| `is_deleted`  | Boolean                                                | Default: false. Only Super Admin can soft-delete    |
| `deleted_by`  | UUID (nullable)                                        | Super Admin who deleted, if applicable              |
| `deleted_at`  | Timestamp (nullable)                                   |                                                     |

> **Indexing**: Composite index on `(document_id, created_at ASC)` for efficient chronological retrieval. Index on `author_id` for per-user comment queries.

***

#### `pipeline_events`

Tracks all SLA pipeline events for each document.

| Column                        | Type                                | Notes                                                                                 |
| ----------------------------- | ----------------------------------- | ------------------------------------------------------------------------------------- |
| `id`                          | UUID (PK)                           |                                                                                       |
| `document_id`                 | UUID (FK → documents.id)            |                                                                                       |
| `editor_id`                   | UUID (FK → editors.id)              |                                                                                       |
| `event_type`                  | String                              | e.g. `submitted`, `stage1_approved`, `pdf_generated`, `delivered`, `revision_flagged` |
| `triggered_by`                | Enum: `teacher`, `editor`, `system` |                                                                                       |
| `triggered_at`                | Timestamp                           |                                                                                       |
| `stage1_sla_deadline`         | Timestamp (nullable)                |                                                                                       |
| `stage1_approved_at`          | Timestamp (nullable)                |                                                                                       |
| `pdf_generation_triggered_at` | Timestamp (nullable)                |                                                                                       |
| `stage2_sla_deadline`         | Timestamp (nullable)                |                                                                                       |
| `stage2_approved_at`          | Timestamp (nullable)                |                                                                                       |
| `delivered_at`                | Timestamp (nullable)                |                                                                                       |
| `sla_paused`                  | Boolean                             | Default: false                                                                        |
| `sla_pause_reason`            | String (nullable)                   |                                                                                       |
| `revision_count`              | Integer                             | Default: 0                                                                            |
| `is_revision_queue`           | Boolean                             | Default: false                                                                        |

***

#### `pdf_outputs`

Records of generated PDF files.

| Column                   | Type                           | Notes                               |
| ------------------------ | ------------------------------ | ----------------------------------- |
| `id`                     | UUID (PK)                      |                                     |
| `document_id`            | UUID (FK → documents.id)       |                                     |
| `editor_id`              | UUID (FK → editors.id)         |                                     |
| `teacher_id`             | UUID (FK → teachers.id)        |                                     |
| `file_path`              | String                         | Storage path of PDF                 |
| `page_count`             | Integer                        | Physical page count — billable unit |
| `file_size_bytes`        | Integer                        |                                     |
| `generated_at`           | Timestamp                      |                                     |
| `generation_duration_ms` | Integer                        | For performance tracking            |
| `weasyprint_retry_count` | Integer                        | How many retries were needed        |
| `billing_cycle_id`       | UUID (FK → billing\_cycles.id) |                                     |

***

#### `billing_cycles`

Monthly billing periods per Teacher. Managed by the Super Admin.

| Column                        | Type                                        | Notes                                                          |
| ----------------------------- | ------------------------------------------- | -------------------------------------------------------------- |
| `id`                          | UUID (PK)                                   |                                                                |
| `super_admin_id`              | UUID (FK -> super\_admins.id)               | Super Admin who manages billing                                |
| `editor_id`                   | UUID (FK -> editors.id)                     | Editor assigned to the Teacher's org                           |
| `teacher_id`                  | UUID (FK -> teachers.id)                    |                                                                |
| `org_id`                      | UUID (FK -> organisations.id)               |                                                                |
| `cycle_start`                 | Date                                        | First day of billing period                                    |
| `cycle_end`                   | Date                                        | Last day of billing period                                     |
| `total_pages`                 | Integer                                     | Running total                                                  |
| `rate_per_page`               | Decimal                                     | Snapshot of rate at billing time                               |
| `discount_per_page`           | Decimal                                     | Default: 0.00. Set to 0.10 for annual, 0.05 for semi-annual    |
| `discount_plan`               | Enum: `none`, `semi-annual`, `annual`       | Active discount plan for this cycle                            |
| `total_amount`                | Decimal                                     | `total_pages x (rate_per_page - discount_per_page)`            |
| `status`                      | Enum: `open`, `invoiced`, `paid`, `overdue` |                                                                |
| `includes_retrospective_docs` | Boolean                                     | Default: false. True if cycle includes retrospective documents |

***

#### `invoices`

Invoice records. Generated and approved by the Super Admin.

| Column                       | Type                                                          | Notes                                                                                            |
| ---------------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `id`                         | UUID (PK)                                                     |                                                                                                  |
| `super_admin_id`             | UUID (FK -> super\_admins.id)                                 | Super Admin who manages this invoice                                                             |
| `editor_id`                  | UUID (FK -> editors.id)                                       | Editor assigned to the Teacher's org                                                             |
| `teacher_id`                 | UUID (FK -> teachers.id)                                      |                                                                                                  |
| `org_id`                     | UUID (FK -> organisations.id)                                 |                                                                                                  |
| `billing_cycle_id`           | UUID (FK -> billing\_cycles.id, nullable)                     | Null for manual or PREPAID invoices                                                              |
| `invoice_type`               | Enum: `postpaid`, `prepaid`, `manual`                         | Type of invoice                                                                                  |
| `invoice_number`             | String                                                        | Auto-generated sequential number                                                                 |
| `amount_due`                 | Decimal                                                       |                                                                                                  |
| `discount_applied`           | Decimal                                                       | Total discount amount if annual plan                                                             |
| `final_amount`               | Decimal                                                       | amount\_due - discount\_applied                                                                  |
| `status`                     | Enum: `pending_review`, `approved`, `sent`, `paid`, `overdue` |                                                                                                  |
| `generated_at`               | Timestamp                                                     |                                                                                                  |
| `approved_by_super_admin_at` | Timestamp (nullable)                                          |                                                                                                  |
| `sent_at`                    | Timestamp (nullable)                                          |                                                                                                  |
| `paid_at`                    | Timestamp (nullable)                                          |                                                                                                  |
| `razorpay_payment_id`        | String (nullable)                                             |                                                                                                  |
| `invoice_pdf_url`            | String (nullable)                                             | URL of the WeasyPrint-generated invoice PDF                                                      |
| `line_items`                 | JSON                                                          | Array of `{ document_id, doc_title, doc_type, page_count, languages, rate, discount, subtotal }` |
| `super_admin_notes`          | String (nullable)                                             | Super Admin's notes or adjustments                                                               |

***

#### `images`

Image metadata records.

| Column                 | Type                                                | Notes                                                       |
| ---------------------- | --------------------------------------------------- | ----------------------------------------------------------- |
| `id`                   | UUID (PK)                                           |                                                             |
| `editor_id`            | UUID (FK → editors.id)                              |                                                             |
| `teacher_id`           | UUID (FK → teachers.id)                             |                                                             |
| `document_id`          | UUID (FK → documents.id)                            |                                                             |
| `news_item_headline`   | String                                              |                                                             |
| `category`             | String                                              |                                                             |
| `document_date`        | Date                                                |                                                             |
| `cloudinary_url`       | String (nullable)                                   | Null after archival                                         |
| `cloudinary_public_id` | String (nullable)                                   | For deletion API                                            |
| `drive_archive_path`   | String (nullable)                                   | Set after year-end archival                                 |
| `academic_year`        | Integer                                             | e.g. 2026                                                   |
| `is_archived`          | Boolean                                             | Default: false                                              |
| `upload_timestamp`     | Timestamp                                           |                                                             |
| `image_type`           | Enum: `thumbnail`, `reference`, `ad`, `placeholder` |                                                             |
| `atomic_uid`           | String (nullable)                                   | Atomic UID for image-to-news-item mapping (see Section 11A) |

***

#### `ad_banners`

Teacher's ad banner library.

| Column           | Type                    | Notes                                     |
| ---------------- | ----------------------- | ----------------------------------------- |
| `id`             | UUID (PK)               |                                           |
| `teacher_id`     | UUID (FK → teachers.id) |                                           |
| `editor_id`      | UUID (FK → editors.id)  |                                           |
| `cloudinary_url` | String                  |                                           |
| `alt_text`       | String (nullable)       |                                           |
| `full_page`      | Boolean                 | Default: false                            |
| `target_url`     | String                  |                                           |
| `caption`        | String (nullable)       |                                           |
| `document_types` | JSON                    | Array of doc types this banner applies to |
| `schedule_type`  | JSON                    | Scheduling configuration                  |
| `is_active`      | Boolean                 | Default: true                             |

***

#### `templates`

Organisation-wide template library. Templates are shared across the organisation — not personal documents of individual roles. Teachers, Editors, and Sub-members see a unified template view from their RBAC dashboard.

| Column                   | Type                                                                            | Notes                                                                                                                  |
| ------------------------ | ------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `id`                     | UUID (PK)                                                                       |                                                                                                                        |
| `org_id`                 | UUID (FK -> organisations.id)                                                   | Templates are org-wide entities                                                                                        |
| `created_by_id`          | UUID                                                                            | Can be teacher\_id, sub\_member\_id, or editor\_id                                                                     |
| `created_by_role`        | Enum: `teacher`, `sub-member`, `editor`                                         |                                                                                                                        |
| `template_type`          | Enum: `header`, `footer`, `full_page`, `custom`                                 | Type of template                                                                                                       |
| `name`                   | String                                                                          | Template name                                                                                                          |
| `description`            | String (nullable)                                                               | Template description                                                                                                   |
| `html_content`           | Text                                                                            | Rendered HTML of the template (drag-and-drop builder output)                                                           |
| `builder_json`           | JSON                                                                            | Serialised builder state for re-editing in the template builder                                                        |
| `canvas_size`            | String                                                                          | Pre-defined layout size (e.g. `A4`, `Letter`, `Custom`)                                                                |
| `scope_document_types`   | JSON                                                                            | Array of doc types this template applies to                                                                            |
| `scope_page_application` | Enum: `first_only`, `last_only`, `all`, `range`, `all_except_first`, `specific` |                                                                                                                        |
| `scope_page_range_start` | Integer (nullable)                                                              |                                                                                                                        |
| `scope_page_range_end`   | Integer (nullable)                                                              |                                                                                                                        |
| `dynamic_fields`         | JSON                                                                            | Array of dynamic placeholder identifiers used: `date`, `yt_url`, `page_number`, `ad_banner`, `title`, `subtitle`, etc. |
| `is_active`              | Boolean                                                                         |                                                                                                                        |
| `created_at`             | Timestamp                                                                       |                                                                                                                        |
| `updated_at`             | Timestamp                                                                       |                                                                                                                        |

***

#### `team_activity_logs`

Sub-member action logs visible to parent Teacher.

| Column          | Type                        | Notes                                                              |
| --------------- | --------------------------- | ------------------------------------------------------------------ |
| `id`            | UUID (PK)                   |                                                                    |
| `teacher_id`    | UUID (FK → teachers.id)     |                                                                    |
| `sub_member_id` | UUID (FK → sub\_members.id) |                                                                    |
| `action_type`   | String                      | e.g. `document_created`, `document_submitted`, `document_exported` |
| `document_id`   | UUID (nullable)             |                                                                    |
| `timestamp`     | Timestamp                   |                                                                    |
| `metadata`      | JSON (nullable)             | Additional context                                                 |

***

#### `translations`

Parallel translation document records.

| Column                   | Type                     | Notes                            |
| ------------------------ | ------------------------ | -------------------------------- |
| `id`                     | UUID (PK)                |                                  |
| `source_document_id`     | UUID (FK → documents.id) | Original document                |
| `translated_document_id` | UUID (FK → documents.id) | Parallel translated version      |
| `language_code`          | String                   | e.g. `hi`, `ta`, `mr`            |
| `language_name`          | String                   | e.g. `Hindi`, `Tamil`, `Marathi` |
| `created_at`             | Timestamp                |                                  |

***

#### `audit_logs`

System-wide audit trail with Super Admin managed deletion capability. Every significant event at every role level is recorded here. Records can only be deleted by the Super Admin (manually or via automated archival schedule). The deletion process archives logs to Google Drive and downloads a CSV to the Super Admin's browser before final deletion from the database.

| Column        | Type                                                             | Notes                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ------------- | ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`          | UUID (PK)                                                        |                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `event_type`  | String                                                           | Enum-like string: e.g. `editor_created`, `editor_authorised`, `editor_suspended`, `org_submitted`, `org_approved`, `org_rejected`, `org_assigned_to_editor`, `org_reassigned`, `teacher_authorised`, `teacher_suspended`, `sub_member_authorised`, `sub_member_left`, `document_submitted`, `document_delivered`, `invoice_approved`, `payment_received`, `login_success`, `login_failure`, `audit_log_deleted`, `billing_cycle_closed`, `prepaid_invoice_issued` |
| `actor_role`  | Enum: `super_admin`, `editor`, `teacher`, `sub-member`, `system` | Who triggered the event                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `actor_id`    | UUID                                                             | ID of the acting entity                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `actor_email` | String                                                           | Denormalised for audit readability                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `target_type` | String                                                           | Type of entity affected: `editor`, `organisation`, `teacher`, `sub_member`, `document`, `invoice`, `audit_log`                                                                                                                                                                                                                                                                                                                                                    |
| `target_id`   | UUID                                                             | ID of the affected entity                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `org_id`      | UUID (nullable)                                                  | Null for system-level events (Super Admin actions that precede Org creation)                                                                                                                                                                                                                                                                                                                                                                                      |
| `editor_id`   | UUID (nullable)                                                  | Denormalised for scope-filtered views                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `details`     | JSON (nullable)                                                  | Additional context: before/after state, error info, reason, etc.                                                                                                                                                                                                                                                                                                                                                                                                  |
| `ip_address`  | String (nullable)                                                | Client IP at time of event                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `created_at`  | Timestamp                                                        | Event timestamp — this is the primary ordering field                                                                                                                                                                                                                                                                                                                                                                                                              |

**Deletion policy**: Only the Super Admin can delete audit log records. Deletion options: `last-hour`, `last-day`, `last-week`, `last-month`, `last-year`, or a specific date range. Before deletion: (1) archive to Google Drive as CSV, (2) download CSV to Super Admin's browser, (3) then delete from Neon database. Automated deletion can be configured in Super Admin Settings with archival-before-delete as a mandatory pre-condition.

***

#### `editor_org_assignments`

Historical record of all Organisation → Editor assignment and reassignment events. Allows Super Admin to see the full assignment history of any Organisation.

| Column                       | Type                         | Notes                                         |
| ---------------------------- | ---------------------------- | --------------------------------------------- |
| `id`                         | UUID (PK)                    |                                               |
| `org_id`                     | UUID (FK → organisations.id) |                                               |
| `editor_id`                  | UUID (FK → editors.id)       | The Editor being assigned                     |
| `assigned_by_super_admin_id` | UUID (FK → super\_admins.id) |                                               |
| `assigned_at`                | Timestamp                    |                                               |
| `unassigned_at`              | Timestamp (nullable)         | Null if this is the current active assignment |
| `reassignment_reason`        | String (nullable)            | Optional note from Super Admin                |

***

#### `atomic_uid_log`

Backend-maintained log of all generated Atomic UIDs to ensure uniqueness across the system. See Section 11A for UID format specification.

| Column          | Type                                                          | Notes                                                  |
| --------------- | ------------------------------------------------------------- | ------------------------------------------------------ |
| `id`            | UUID (PK)                                                     |                                                        |
| `atomic_uid`    | String (unique)                                               | The full Atomic UID string                             |
| `entity_type`   | Enum: `category`, `news_item`, `thumbnail`, `reference_image` | Type of entity this UID identifies                     |
| `document_id`   | UUID (FK -> documents.id)                                     | Document containing this entity                        |
| `document_date` | Date                                                          | Date of the document (for DDMMYY prefix deduplication) |
| `org_id`        | UUID (FK -> organisations.id)                                 |                                                        |
| `created_at`    | Timestamp                                                     |                                                        |

***

#### `prepaid_invoices`

Tracks PREPAID platform fee invoices issued by the Super Admin. Separate from usage-based POSTPAID invoices.

| Column                | Type                                       | Notes                                   |
| --------------------- | ------------------------------------------ | --------------------------------------- |
| `id`                  | UUID (PK)                                  |                                         |
| `super_admin_id`      | UUID (FK -> super\_admins.id)              |                                         |
| `teacher_id`          | UUID (FK -> teachers.id)                   |                                         |
| `org_id`              | UUID (FK -> organisations.id)              |                                         |
| `invoice_number`      | String                                     | Auto-generated sequential number        |
| `amount`              | Decimal                                    | Freeform amount set by Super Admin      |
| `description`         | String (nullable)                          | Super Admin's description of the charge |
| `status`              | Enum: `pending`, `sent`, `paid`, `overdue` |                                         |
| `generated_at`        | Timestamp                                  |                                         |
| `sent_at`             | Timestamp (nullable)                       |                                         |
| `paid_at`             | Timestamp (nullable)                       |                                         |
| `razorpay_payment_id` | String (nullable)                          |                                         |
| `invoice_pdf_url`     | String (nullable)                          | WeasyPrint-generated invoice PDF        |

***

#### `billing_prepaid_config`

Per-Teacher PREPAID billing configuration managed by the Super Admin.

| Column                  | Type                             | Notes                                                                  |
| ----------------------- | -------------------------------- | ---------------------------------------------------------------------- |
| `id`                    | UUID (PK)                        |                                                                        |
| `teacher_id`            | UUID (FK -> teachers.id, unique) |                                                                        |
| `org_id`                | UUID (FK -> organisations.id)    |                                                                        |
| `prepaid_invoicing`     | Boolean                          | Default: false. When true, all functionality is gatekept until cleared |
| `set_by_super_admin_id` | UUID (FK -> super\_admins.id)    |                                                                        |
| `updated_at`            | Timestamp                        |                                                                        |

***

#### `publications`

Publication streams per Organisation. Each org has at least one publication (default: CURRENT-AFFAIRS).

| Column                       | Type                                                          | Notes                                                                        |
| ---------------------------- | ------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `id`                         | UUID (PK)                                                     |                                                                              |
| `org_id`                     | UUID (FK → organisations.id)                                  | Organisation this publication belongs to                                     |
| `name`                       | String                                                        | Unique within org. e.g. `CURRENT-AFFAIRS`, `SCIENCE-OUTLOOK`                 |
| `is_default`                 | Boolean                                                       | Default: false. True only for the auto-created `CURRENT-AFFAIRS` publication |
| `status`                     | Enum: `pending_approval`, `active`, `suspended`, `terminated` |                                                                              |
| `created_by_id`              | UUID                                                          | Teacher or Editor who initiated creation                                     |
| `created_by_role`            | Enum: `teacher`, `editor`                                     |                                                                              |
| `approved_by_super_admin_id` | UUID (FK → super\_admins.id, nullable)                        | Super Admin who approved. NULL for default publication                       |
| `approved_at`                | Timestamp (nullable)                                          |                                                                              |
| `suspended_by`               | UUID (nullable)                                               | Super Admin who suspended, if applicable                                     |
| `suspended_at`               | Timestamp (nullable)                                          |                                                                              |
| `created_at`                 | Timestamp                                                     |                                                                              |
| `updated_at`                 | Timestamp                                                     |                                                                              |

> **Constraint**: `(org_id, name)` must be unique. At least one publication per org (default cannot be deleted).

***

#### `publication_document_types`

Custom document types configured within each publication stream.

| Column                 | Type                                                                                              | Notes                                                                         |
| ---------------------- | ------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `id`                   | UUID (PK)                                                                                         |                                                                               |
| `publication_id`       | UUID (FK → publications.id)                                                                       | Publication this document type belongs to                                     |
| `org_id`               | UUID (FK → organisations.id)                                                                      | Denormalised for unique constraint                                            |
| `name`                 | String                                                                                            | Custom name. Must be unique across all publications in the same org           |
| `is_atomic`            | Boolean                                                                                           | True = single-date document (building block). False = aggregated document     |
| `aggregation_range`    | Enum: `weekly`, `bi-weekly`, `monthly`, `quarterly`, `semi-annual`, `annual`, `custom` (nullable) | Only for aggregated types                                                     |
| `aggregation_days`     | Integer (nullable)                                                                                | For `custom` range: number of days                                            |
| `auto_trigger_enabled` | Boolean                                                                                           | Default: true for default publication types, false for custom                 |
| `trigger_date_rule`    | JSON (nullable)                                                                                   | Trigger schedule configuration (date, time). See Sections 9.2–9.3C for format |
| `trigger_time`         | Time                                                                                              | Default: `18:00` IST                                                          |
| `status`               | Enum: `active`, `suspended`                                                                       | Super Admin can suspend individual document types                             |
| `created_at`           | Timestamp                                                                                         |                                                                               |
| `updated_at`           | Timestamp                                                                                         |                                                                               |

> **Constraint**: `(org_id, name)` must be unique across all publications. This enforces cross-publication naming uniqueness within an org.

***

#### `support_tickets`

Customer support tickets raised by tenants. See Section 22A.

| Column                | Type                                                                                                                                    | Notes                                                 |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| `id`                  | UUID (PK)                                                                                                                               |                                                       |
| `ticket_number`       | String                                                                                                                                  | Auto-generated sequential: `TKT-YYYYMMDD-NNNN`        |
| `org_id`              | UUID (FK → organisations.id, nullable)                                                                                                  | Null for system-level issues                          |
| `reporter_id`         | UUID                                                                                                                                    | User who created the ticket                           |
| `reporter_role`       | Enum: `editor`, `teacher`, `sub-member`                                                                                                 |                                                       |
| `reporter_name`       | String                                                                                                                                  | Denormalised for display                              |
| `reporter_email`      | String                                                                                                                                  | Denormalised for email notifications                  |
| `subject`             | String (max 200 chars)                                                                                                                  | Brief issue description                               |
| `category`            | Enum: `bug-report`, `feature-request`, `billing-inquiry`, `account-issue`, `technical-support`, `compliance-dispute`, `general-inquiry` |                                                       |
| `priority`            | Enum: `low`, `medium`, `high`, `critical`                                                                                               |                                                       |
| `description`         | Text (max 5000 chars)                                                                                                                   | Detailed description                                  |
| `related_document_id` | UUID (FK → documents.id, nullable)                                                                                                      | Optional link to a specific document                  |
| `status`              | Enum: `open`, `in-progress`, `awaiting-user-response`, `escalated`, `resolved`, `closed`, `reopened`                                    |                                                       |
| `internal_notes`      | Text (nullable)                                                                                                                         | Super Admin's private notes (not visible to reporter) |
| `resolved_at`         | Timestamp (nullable)                                                                                                                    |                                                       |
| `closed_at`           | Timestamp (nullable)                                                                                                                    |                                                       |
| `auto_close_date`     | Timestamp (nullable)                                                                                                                    | 14 days after resolution                              |
| `created_at`          | Timestamp                                                                                                                               |                                                       |
| `updated_at`          | Timestamp                                                                                                                               |                                                       |

***

#### `ticket_messages`

Conversation thread messages within a support ticket.

| Column        | Type                                                   | Notes                                                                       |
| ------------- | ------------------------------------------------------ | --------------------------------------------------------------------------- |
| `id`          | UUID (PK)                                              |                                                                             |
| `ticket_id`   | UUID (FK → support\_tickets.id)                        |                                                                             |
| `author_id`   | UUID                                                   |                                                                             |
| `author_role` | Enum: `super_admin`, `editor`, `teacher`, `sub-member` |                                                                             |
| `author_name` | String                                                 | Denormalised for display                                                    |
| `message`     | Text (max 2000 chars)                                  | Message body                                                                |
| `attachments` | JSON (nullable)                                        | Array of `{ filename, url, size_bytes, mime_type }`. Max 3 files, 10MB each |
| `created_at`  | Timestamp                                              |                                                                             |

> **Indexing**: Composite index on `(ticket_id, created_at ASC)` for chronological retrieval.

***

#### `ticket_attachments`

File attachments on tickets (initial submission attachments).

| Column        | Type                            | Notes                               |
| ------------- | ------------------------------- | ----------------------------------- |
| `id`          | UUID (PK)                       |                                     |
| `ticket_id`   | UUID (FK → support\_tickets.id) |                                     |
| `filename`    | String                          | Original filename                   |
| `url`         | String                          | Storage URL                         |
| `size_bytes`  | Integer                         | File size                           |
| `mime_type`   | String                          | e.g. `image/png`, `application/pdf` |
| `uploaded_by` | UUID                            |                                     |
| `uploaded_at` | Timestamp                       |                                     |

***

#### `compliance_records`

Per-publication-stream compliance tracking for evader detection (Section 23.8).

| Column                        | Type                                                                                                   | Notes                                                             |
| ----------------------------- | ------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| `id`                          | UUID (PK)                                                                                              |                                                                   |
| `org_id`                      | UUID (FK → organisations.id)                                                                           |                                                                   |
| `publication_id`              | UUID (FK → publications.id)                                                                            |                                                                   |
| `billing_cycle_id`            | UUID (FK → billing\_cycles.id)                                                                         |                                                                   |
| `total_atomic_docs_created`   | Integer                                                                                                | Total atomic documents created in this publication for this cycle |
| `total_atomic_docs_submitted` | Integer                                                                                                | Total atomic documents submitted to PDF pipeline                  |
| `compliance_percentage`       | Decimal                                                                                                | `(submitted / created) * 100`                                     |
| `threshold`                   | Decimal                                                                                                | Configured threshold for this org (default: 75.00)                |
| `is_compliant`                | Boolean                                                                                                | `compliance_percentage >= threshold`                              |
| `grace_period_active`         | Boolean                                                                                                | True if org is still within 2-cycle grace period                  |
| `status`                      | Enum: `compliant`, `grace-period`, `flagged`, `warned`, `exception-granted`, `suspended`, `terminated` |                                                                   |
| `action_taken`                | Enum: `none`, `allow`, `warning`, `temporary-suspension`, `termination` (nullable)                     |                                                                   |
| `action_taken_by`             | UUID (FK → super\_admins.id, nullable)                                                                 |                                                                   |
| `action_taken_at`             | Timestamp (nullable)                                                                                   |                                                                   |
| `action_notes`                | Text (nullable)                                                                                        | Exception details or warning message                              |
| `created_at`                  | Timestamp                                                                                              |                                                                   |

***

#### `discount_commitments`

Tracks 6-month and 12-month PREPAID discount commitments per Organisation.

| Column               | Type                                   | Notes                                                 |
| -------------------- | -------------------------------------- | ----------------------------------------------------- |
| `id`                 | UUID (PK)                              |                                                       |
| `org_id`             | UUID (FK → organisations.id)           |                                                       |
| `teacher_id`         | UUID (FK → teachers.id)                |                                                       |
| `plan`               | Enum: `semi-annual`, `annual`          | Commitment type                                       |
| `commitment_start`   | Date                                   | First day of commitment period                        |
| `commitment_end`     | Date                                   | Last day of commitment period                         |
| `discount_per_page`  | Decimal                                | 0.05 for semi-annual, 0.10 for annual                 |
| `prepaid_invoice_id` | UUID (FK → prepaid\_invoices.id)       | The PREPAID invoice that activated this commitment    |
| `auto_renewal`       | Boolean                                | Default: false                                        |
| `razorpay_token_id`  | String (nullable)                      | Tokenised payment method for auto-renewal (encrypted) |
| `status`             | Enum: `active`, `expired`, `cancelled` |                                                       |
| `created_at`         | Timestamp                              |                                                       |
| `updated_at`         | Timestamp                              |                                                       |

***

### 29.2 v5 Additions

The tables below were added in PRD v5.0.0. Existing tables also gained columns — see §29.3.

#### `qa_items`

Q\&A items generated via the OpenRouter service (§22B). One row per accepted Q\&A item.

| Column                | Type                                                                                   | Notes                                                                                                                                                                   |
| --------------------- | -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`                  | UUID (PK)                                                                              |                                                                                                                                                                         |
| `qa_id`               | String (unique within document)                                                        | Format `QA-{doc-id}-{3-digit seq}`. Backend-assigned only (see §22B.3.D / §22B.3.E).                                                                                    |
| `qa_id_seq`           | Integer                                                                                | Numeric sequence (1, 2, 3, …) parsed from `qa_id`. Indexed for `SELECT MAX(...) FOR UPDATE` concurrency lock in §22B.3.E step 7. Unique per `(document_id, qa_id_seq)`. |
| `document_id`         | UUID (FK → documents.id)                                                               |                                                                                                                                                                         |
| `org_id`              | UUID (FK → organisations.id)                                                           | Tenant isolation                                                                                                                                                        |
| `source_atomic_uids`  | JSON                                                                                   | Array of atomic\_uids of news items used as input. Backend-resolved from ephemeral `source_refs` (§22B.3.D). Drives aggregation linkage.                                |
| `slot_index`          | Integer                                                                                | The wizard slot index (1..N) this item was generated for. Preserved for audit; ordering authority during multi-call assembly (§22B.3.E).                                |
| `type`                | Enum: `subjective.straightforward`, `objective.direct`, `objective.statement_analysis` |                                                                                                                                                                         |
| `payload`             | JSON                                                                                   | Type-specific body. `{statement, answer}` OR `{statement, options, correct_option}` OR `{stem, statements, options, correct_option}`                                    |
| `generated_by_role`   | Enum: `teacher`, `editor`, `sub-member`                                                |                                                                                                                                                                         |
| `generated_by_id`     | UUID                                                                                   | Actor who accepted the item                                                                                                                                             |
| `model_used`          | String                                                                                 | OpenRouter model slug                                                                                                                                                   |
| `is_late_added`       | Boolean                                                                                | True when accepted post-Stage-3 (triggers §13.6 regen)                                                                                                                  |
| `accepted_at`         | Timestamp                                                                              |                                                                                                                                                                         |
| `edited_after_accept` | Boolean                                                                                | True if any field was edited after initial acceptance                                                                                                                   |

***

#### `revision_screenshots`

PNG screenshots attached to revision flags via the §13.3.4A wizard. Stored as base64 in Neon DB. Auto-purged 30 days after resolution.

| Column           | Type                     | Notes                                                                                                   |
| ---------------- | ------------------------ | ------------------------------------------------------------------------------------------------------- |
| `id`             | UUID (PK)                |                                                                                                         |
| `document_id`    | UUID (FK → documents.id) |                                                                                                         |
| `revision_id`    | String                   | Identifier matching the entry in `revision-history[]` (e.g. `REV-001`)                                  |
| `screenshot_id`  | String                   | Per-flag sequential (e.g. `REV-001-shot-1`)                                                             |
| `page_range`     | String                   | e.g. `"3"`, `"5-7"`, or `"3,region"` when a quadrilateral was drawn                                     |
| `quad_points`    | JSON (nullable)          | When region was selected: array of 4 `{x, y}` points in PDF page coordinates. Null for whole-page caps. |
| `mime`           | String                   | `image/png` only (validated)                                                                            |
| `b64_data`       | Text                     | Base64 PNG. Max 1 MB decoded (\~1.37 MB encoded). Enforced both client- and server-side.                |
| `b64_size_bytes` | Integer                  |                                                                                                         |
| `captured_at`    | Timestamp                |                                                                                                         |
| `purge_after`    | Timestamp                | Computed: `revision-history[-1].resolved-at + 30 days`. NULL until resolution. Cron job hard-deletes.   |

> **Constraint**: per `(document_id, revision_id)`, `COUNT(screenshot_id) <= 5`. Enforced via insert trigger.

***

#### `openrouter_config`

Single-row table (or system-settings JSON) holding global model resolution and fallback chain. Per-Org overrides live in `organisations` (see §29.3).

| Column                         | Type      | Notes                                            |
| ------------------------------ | --------- | ------------------------------------------------ |
| `id`                           | UUID (PK) |                                                  |
| `default_model`                | String    | OpenRouter model slug, e.g. `openai/gpt-4o-mini` |
| `fallback_models`              | JSON      | Ordered array of exactly 4 model slugs           |
| `default_qa_monthly_token_cap` | Integer   | Default: 100000                                  |
| `api_key_encrypted`            | String    | Platform OpenRouter API key (encrypted at rest)  |
| `updated_by_super_admin_id`    | UUID      |                                                  |
| `updated_at`                   | Timestamp |                                                  |

***

#### `openrouter_call_log`

Per-call telemetry for Q\&A spend tracking (§22B.4).

| Column              | Type                              | Notes                                         |
| ------------------- | --------------------------------- | --------------------------------------------- |
| `id`                | UUID (PK)                         |                                               |
| `org_id`            | UUID (FK → organisations.id)      |                                               |
| `document_id`       | UUID (FK → documents.id, null OK) |                                               |
| `model_used`        | String                            | OpenRouter model slug for this attempt        |
| `attempt_index`     | Integer                           | 0-indexed within the request's retry chain    |
| `prompt_tokens`     | Integer                           | Reported by OpenRouter                        |
| `completion_tokens` | Integer                           |                                               |
| `total_tokens`      | Integer                           |                                               |
| `success`           | Boolean                           |                                               |
| `validation_errors` | JSON (nullable)                   | Structured Outputs / schema / business errors |
| `created_at`        | Timestamp                         |                                               |

***

#### `ay_archives`

Tracks Full-AY archive ZIPs created and their recovery lifecycle (§26A).

| Column                    | Type                                          | Notes                                         |
| ------------------------- | --------------------------------------------- | --------------------------------------------- |
| `id`                      | UUID (PK)                                     |                                               |
| `org_id`                  | UUID (FK → organisations.id)                  |                                               |
| `academic_year`           | String                                        | e.g. `2026-2027`                              |
| `drive_path`              | String                                        | Path to the ZIP in Teacher's Drive            |
| `manifest_sha256`         | String                                        | SHA-256 of the `manifest.json` inside the ZIP |
| `manifest_hmac`           | String                                        | The HMAC signature stored in `manifest.sig`   |
| `total_documents`         | Integer                                       |                                               |
| `total_images`            | Integer                                       |                                               |
| `total_size_bytes`        | BigInt                                        |                                               |
| `status`                  | Enum: `created`, `recovered`, `drive_deleted` |                                               |
| `created_at`              | Timestamp                                     |                                               |
| `recovered_at`            | Timestamp (nullable)                          |                                               |
| `recovered_by_teacher_id` | UUID (nullable)                               |                                               |

***

### 29.3 v5 Column Additions to Existing Tables

| Table           | Column                          | Type                           | Notes                                                                                                                                                                        |
| --------------- | ------------------------------- | ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `organisations` | `drive_sync_enabled`            | Boolean                        | Default `false`. Teacher-controlled. Master toggle for §25. When false, all Drive writes are silently skipped; storage served from Neon + Cloudinary + backend object store. |
| `organisations` | `full_ay_archive_enabled`       | Boolean                        | Default `false`. Teacher-controlled. Enables §26A flow.                                                                                                                      |
| `organisations` | `full_ay_archived_years`        | JSON (array of strings)        | e.g. `["2025-2026"]`. Append on archive, remove on recovery.                                                                                                                 |
| `organisations` | `archive_hmac_secret_encrypted` | String                         | Org-scoped HMAC secret for archive signing (§26A.3). Generated on Org creation.                                                                                              |
| `organisations` | `qa_monthly_token_cap`          | Integer (nullable)             | Override for §22B.4 default. NULL = use system default.                                                                                                                      |
| `organisations` | `qa_model_override`             | String (nullable)              | Per-Org primary model. NULL = use system `default_model`.                                                                                                                    |
| `organisations` | `qa_fallback_models_override`   | JSON (nullable)                | Per-Org fallback chain. NULL = use system fallbacks.                                                                                                                         |
| `organisations` | `qa_tokens_used_current_cycle`  | Integer                        | Running total, reset at billing-cycle start.                                                                                                                                 |
| `images`        | `crop`                          | JSON (nullable)                | Non-destructive crop transform `{x, y, w, h, applied_at}`. NULL when uncropped.                                                                                              |
| `images`        | `origin`                        | Enum: `upload`, `inline-paste` | `upload` for thumbnails + ads. `inline-paste` for Brief-pasted reference images.                                                                                             |
| `images`        | `storage_format`                | Enum: `webp`                   | Hardcoded `webp` from v5 onward (§26.6).                                                                                                                                     |
| `sub_members`   | `permissions.qa` (JSON key)     | Boolean                        | Default `false`. Granted by Teacher (§21.3). Required for Q\&A wizard access.                                                                                                |
| `documents`     | `has_qa`                        | Boolean                        | Mirrors presence of `qa` node in `content_json`. Query convenience flag.                                                                                                     |
| `documents`     | `qa_late_added`                 | Boolean                        | True when Q\&A was accepted post-Stage-3 → triggered §13.6 regen.                                                                                                            |
