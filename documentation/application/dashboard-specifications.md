---
icon: tv
---

# Dashboard Specifications

## SECTION 8 — DASHBOARD SPECIFICATIONS

### 8.0 Super Admin Dashboard

The Super Admin Dashboard is the **system-level command centre** of the entire platform. It is only accessible to the Super Admin account. It is completely separate from the Editor and Teacher dashboards — different route, different navigation, different data scope. It has no document creation or PDF pipeline controls — those are operational concerns delegated to Editors and Teachers.

#### 8.0.1 Editor Management Panel

The primary tool for spawning and governing Editors.

**Interface:**

* A data table listing all Editor accounts with columns: Name, Email, Status (`pending` / `active` / `suspended`), Organisations Assigned (count), Date Created, Last Active.
* Sortable and filterable by all columns.

**Actions:**

| Action             | Trigger                                      | Outcome                                                                                                                                                         |
| ------------------ | -------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Create New Editor  | "Create Editor" button → form (name, email)  | Editor record created with `status: pending`; Editor receives account creation email with password-set link                                                     |
| Authorise Editor   | "Authorise" button on a `pending` Editor row | Editor `status → active`; Editor receives activation email                                                                                                      |
| Suspend Editor     | "Suspend" action on an `active` Editor row   | Editor `status → suspended`; all Organisation assignments remain but Editor cannot log in; Teachers and sub-members in mapped Orgs are unaffected operationally |
| Reactivate Editor  | "Reactivate" on a `suspended` Editor row     | Editor `status → active`                                                                                                                                        |
| View Editor Detail | Click Editor row                             | Opens Editor detail view: profile, mapped Organisations list, activity log, billing summary across all mapped Orgs                                              |

#### 8.0.2 Organisation Management Panel

The primary tool for approving incoming Organisation registrations and assigning them to Editors.

**Interface:**

* Two tabs: "Pending Approvals" and "All Organisations."
* **Pending Approvals tab**: Lists all Organisations with `status: pending_approval`, ordered by submission date (oldest first). Each row shows: Organisation name, Teacher name, Teacher email, submission date, region/timezone.
* **All Organisations tab**: Full paginated list of all Organisations in any status, with all columns + assigned Editor column.

**Actions:**

| Action                   | Trigger                                                         | Outcome                                                                                               |
| ------------------------ | --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Approve Organisation     | "Approve" button on pending Org row                             | Org `status → active`; Organisation becomes eligible for Editor assignment                            |
| Reject Organisation      | "Reject" button → rejection reason input                        | Teacher and Org notified with reason; Org and Teacher account soft-deleted after 30-day grace period  |
| Assign Editor            | "Assign Editor" on an approved Org → dropdown of active Editors | `org.editor_id` set; assigned Editor notified to authorise the Teacher                                |
| Reassign Editor          | "Reassign" on any Org → new Editor dropdown                     | Previous Editor loses visibility; new Editor gains full access; all data preserved                    |
| Suspend Organisation     | "Suspend" action                                                | All accounts within the Org (Teacher + sub-members) are suspended immediately                         |
| View Organisation Detail | Click Org row                                                   | Opens Org detail: Teacher info, sub-member count, document volume, billing summary, activity timeline |

#### 8.0.3 System-Wide Activity & Audit Log

A full, chronological log of every significant system event:

* **Log entries include**: Account creation, authorisation, suspension, reactivation — at all levels (Editor, Teacher, Sub-member). Organisation approval, rejection, reassignment. Billing events. Pipeline failures. Login events.
* **Columns**: Timestamp, Event Type, Actor (who triggered it), Target (which account or entity was affected), Scope (which Org), Details.
* Filterable by: date range, event type, Actor role, target entity.
* The log is **append-only during normal operation** — no entry can be edited, even by the Super Admin.
* **Audit Log Deletion (Super Admin only)**: The Super Admin can manage audit log deletion through a dedicated management interface:
  * **Deletion options**: `last-hour`, `last-day`, `last-week`, `last-month`, `last-year`, `specific date range`
  * **Mandatory archival before deletion**: Before any audit log records are deleted, they are first archived to Google Drive as a compressed file and a downloadable CSV is generated for the Super Admin's browser.
  * **Automated deletion settings**: The Super Admin can configure automatic audit log deletion schedules in Super Admin Settings → Audit Log Retention, with archival performed automatically before each scheduled deletion.
  * **Storage flow**: Neon DB (active) → Google Drive archive (compressed) + CSV download → Deletion from Neon DB.
* Export to CSV available for compliance purposes at any time.

#### 8.0.4 Platform-Level Metrics

A high-level metrics panel giving the Super Admin global visibility across all tenants:

| Metric                                    | Description                                               |
| ----------------------------------------- | --------------------------------------------------------- |
| Total Editors                             | Count active, pending, suspended                          |
| Total Organisations                       | Count active, pending, suspended                          |
| Total Teachers                            | Sum across all Orgs                                       |
| Total Sub-members                         | Sum across all Orgs                                       |
| Total PDF pages generated (platform-wide) | Current month + all-time                                  |
| Total documents in pipeline               | By status across all Orgs                                 |
| System health indicators                  | Cloud Run uptime, API error rate, WeasyPrint failure rate |

#### 8.0.5 Super Admin Billing & Invoicing Dashboard

The Super Admin has **full ownership and control** of all billing activities across the platform. This is the central billing command centre.

**Interface:**

* **System-wide view**: Shows billing data for all Editors' respective enrolled Organisations and Teachers.
* **Per-Editor drill-down**: View individual Editor's work for their respective enrolled Organisation owners (Teachers).
* **Per-Teacher drill-down**: All document metadata including all types of documents generated / revised / currently in queue / not yet sent to editor, sortable by date, filterable for a specified period or since inception.

**Capabilities:**

| Capability                  | Details                                                                                                                                                                                              |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Set pricing                 | Super Admin sets per-page rates for monthly and annual billing plans per Teacher or globally                                                                                                         |
| Generate invoices           | Auto-generated on schedule or manually triggered for any Teacher for any desired period                                                                                                              |
| POSTPAID invoicing          | Default trigger: 5th of next calendar month for the previous month's usage. Includes retrospectively created document flags.                                                                         |
| PREPAID invoicing           | Optional platform fee charged before any document creation is allowed. Fully freeform — Super Admin types any amount per invoice per Teacher. Can be enabled/disabled per Teacher per billing cycle. |
| Invoice PDF generation      | Invoices are generated as PDF via WeasyPrint for saving, exporting, or sending as email attachments                                                                                                  |
| Receive payments            | All payments from Teachers are received by Super Admin via the master Razorpay account                                                                                                               |
| Billing cycle configuration | Configure billing cycle triggers in Super Admin Settings. Default: month-to-month basis.                                                                                                             |
| Editor compensation         | Editor compensation is handled externally through company chartered accounts management — not through the application                                                                                |

**Billing Cycle Flow Example:**

* `USER_REGISTRATION_DATE` = 16-07-2026
* `ACADEMIC_YEAR` = 01-01-2026 to 31-12-2026
* `FIRST_BILLING_CYCLE` = 16-07-2026 to 31-07-2026 (prorated)
* `PREPAID_INVOICING_CLEARANCE`: If `true`, user is charged before any usage. If `false`, postpaid only.
  * **Edge case**: Super Admin can set PREPAID to `false` when onboarding a new client and later switch to `true` for subsequent billing cycles to ease adoption resistance.
* `FIRST_POSTPAID_INVOICE` = Generated on 05-08-2026 for July usage
* `GRACE_PERIOD` = 7 days from due date
* `POST_GRACE_BLOCK` = PDF pipeline submission disabled; document creation + PPTX export remain active
* `FUNCTIONALITY_RESTORES`: Upon POSTPAID invoice clearance, the PDF pipeline is restored. Backend automatically schedules documents created during grace period where `status = approved` and queues them for Editor review. Teacher/sub-member can also manually re-submit. Email and in-app notification acknowledges functionality restoration to the Organisation owner and sub-members.

**Billing includes all document types:**

* Daily Newsletters
* Daily Mindmaps
* Weekly Compilation
* Monthly Magazine
* Quarterly Collection
* Bi-Annual Compendium
* Annual Yearbook

***

### 8.1 Teacher Dashboard

The Teacher Dashboard is the primary landing page after login. It contains three sections:

#### 8.1.1 Entry Point Cards

Flip cards serve as the gateway entry points into the primary creation and information flows. Each card:

* Uses ShadCN UI or compatible third-party ShadCN registry components
* Shows a front face with the document type name and an icon/visual
* On hover/click, flips to reveal a back face with a quick-start CTA button and brief description

| Card   | Label                | Target Flow                                        |
| ------ | -------------------- | -------------------------------------------------- |
| Card 1 | DAILY NEWSLETTER     | Opens the Newsletter canvas creation flow          |
| Card 2 | WEEKLY COMPILATION   | Shows upcoming Compilation trigger date and status |
| Card 3 | MONTHLY MAGAZINE     | Shows upcoming Magazine trigger date and status    |
| Card 4 | QUARTERLY COLLECTION | Shows upcoming Quarterly trigger date and status   |
| Card 5 | BI-ANNUAL COMPENDIUM | Shows upcoming Compendium trigger date and status  |
| Card 6 | ANNUAL YEARBOOK      | Shows upcoming Yearbook trigger date and status    |

> Note: Compilations, Magazines, Quarterly Collections, Bi-Annual Compendiums, and Annual Yearbooks are auto-triggered or Editor-created. The Teacher's cards for these are informational — showing status and upcoming dates — not a direct creation CTA (since Teachers do not create these themselves).

#### 8.1.2 Kanban Board

The Teacher's Kanban board provides a visual pipeline overview of all their documents in progress.

**Columns (left to right):**

1. `Draft` — documents the Teacher has started but not yet finalised
2. `Submitted` — documents sent to the Editor's queue, locked for editing
3. `In Review` — documents currently being reviewed/edited by the Editor
4. `PDF Generated` — documents for which PDF has been produced, pending final delivery
5. `Delivered` — documents returned to the Teacher as completed PDFs

**Kanban Cards:**

* Each card represents a single document.
* Cards display: document title, document type badge (Newsletter / Compilation / Mindmap), creation date, current status timestamp.
* Cards are draggable for visual reordering within a column (display only — does not change document status).
* Clicking a card opens a document detail drawer with full status history and action buttons appropriate to current status.

#### 8.1.3 Logs Table

A tabular view of recent document processing events:

* Columns: Document Name, Type, Event, Timestamp, Status, Actor
* Sortable by all columns
* Filterable by document type and status
* Displays the last 50 events by default, with pagination for older records
* Real-time updates as pipeline events fire

***

### 8.2 Editor Dashboard

The Editor Dashboard is purpose-scoped to pipeline management and oversight.

#### 8.2.0 Entry Point Cards

The Editor sees Entry Point Cards aggregated across all their assigned Organisations. Each card shows the total count for the document type across all assigned orgs. Cards use ShadCN UI or compatible third-party ShadCN registry components.

| Card   | Label                | Description                                                                  |
| ------ | -------------------- | ---------------------------------------------------------------------------- |
| Card 1 | DAILY NEWSLETTER     | Count of Newsletters in queue / in review / pending delivery across all orgs |
| Card 2 | WEEKLY COMPILATION   | Upcoming Compilation trigger dates + count of pending Compilations           |
| Card 3 | MONTHLY MAGAZINE     | Upcoming Magazine trigger dates + count of pending Magazines                 |
| Card 4 | QUARTERLY COLLECTION | Upcoming Quarterly trigger dates + count of pending Collections              |
| Card 5 | BI-ANNUAL COMPENDIUM | Upcoming Compendium trigger dates + count of pending Compendiums             |
| Card 6 | ANNUAL YEARBOOK      | Upcoming Yearbook trigger dates + count of pending Yearbooks                 |
| Card 7 | MINDMAP              | Count of Mindmaps pending generation / in review / delivered                 |

> Note: Clicking any card filters the Kanban below to show only documents of that type. Card 7 (Mindmap) is exclusive to the Editor — Teachers do not have a Mindmap creation card since Mindmaps are generated by the Editor from Newsletter assets.

#### 8.2.1 Editor Kanban Board

The Editor's Kanban shows all documents across all their assigned Teachers in a pipeline view:

**Columns (left to right):**

1. `Unassigned` — documents created by Teachers or Sub-members on their dashboard that have **not yet been submitted** to the Editor for PDF pipeline processing. These are visible to the Editor as **view-only**. Each card displays the Organisation name and the document creator (Teacher or Sub-member name with role badge). The Editor can see these documents but cannot edit them until the Teacher/Sub-member submits them.
2. `Queued` — documents submitted by Teachers and now in the Editor's active queue (2-hour SLA clock running)
3. `In Review` — Editor is actively working on this document
4. `PDF Generated` — PDF pipeline has completed, awaiting Editor final approval
5. `Delivered` — document sent to Teacher as completed output
6. `Flagged for Revision` — Teacher flagged a delivered PDF — requires Editor attention (highlighted/badged)

**Kanban Cards (Editor view):**

* Display: Teacher name, document title, document type badge, submission timestamp, SLA countdown timer (if in Queued or In Review state), revision flag indicator.
* Flagged cards must have a distinct visual treatment (red border, warning icon) to ensure they are immediately noticeable.

#### 8.2.2 Editor Queue Management

* Clicking any Kanban card opens the full document detail view with editing capabilities.
* Documents in `Queued` state show a real-time countdown timer to the 2-hour SLA auto-continuation deadline.
* The Editor can: Edit document content, approve document, trigger PDF generation, approve PDF, deliver to Teacher.
* For revision-flagged documents: all actions require manual Editor confirmation — no auto-continuation.

#### 8.2.3 Priority Notification Panel

A dedicated panel (sidebar or notification centre) surfaces items requiring immediate attention:

* Revision-flagged documents (priority — highlighted, audible alert optional)
* WeasyPrint failure escalations with error details
* Storage limit alerts (Drive or Neon)
* Teacher pending approval requests
* Drive sync failures with error logs

#### 8.2.4 Editor Document Metrics Dashboard

Located within the Editor's RBAC panel under a dedicated "Document Metrics" tab. This dashboard shows **document processing metrics only — no pricing, invoicing, or payment data** is visible to the Editor.

* **Default scope**: Current calendar month (1st to today)
* **Filter controls**: Date range picker, Teacher name selector, document type selector
* **Metrics displayed per Teacher**:
  * Total PDF pages generated
  * Number of documents processed
  * Average processing time per document
  * Document type breakdown (Newsletter / Compilation / Magazine / Mindmap / Quarterly / Bi-Annual / Annual Yearbook)
* **Visualisations**: Line charts for page count trends, bar charts for document type breakdown, data table for raw granular activity logs
* **Sortable by**: Teacher name, page count, document count, processing date
* **Export**: Download document metrics data as CSV on demand

> **Important**: The Editor never sees any invoicing dashboard with pricing. No outstanding or paid dues are visible. They can only see month-wise document processing metrics without price parameters and charges.
