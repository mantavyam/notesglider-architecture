---
icon: circle-dollar
---

# Billing & Monetization

## SECTION 23 — BILLING & MONETIZATION (RAZORPAY)

### 23.1 Billing Model

Notesglider operates on a **usage-based billing model**. Teachers are charged per physical PDF page generated. **Billing is entirely managed by the Super Admin** — not by Editors. Discounts apply to **POSTPAID invoices only** — the usage-based charges calculated after each billing cycle. PREPAID platform fees are a separate payment instance and are not affected by discount tiers.

| Billing Plan           | Commitment Period | Rate per PDF Page | Discount per Page | How to Activate                                |
| ---------------------- | ----------------- | ----------------- | ----------------- | ---------------------------------------------- |
| Monthly Billing        | Month-to-month    | $1.00 per page    | None              | Default — no commitment required               |
| Semi-Annual Commitment | 6 months          | $0.95 per page    | $0.05 per page    | Pay full 6-month PREPAID platform fee upfront  |
| Annual Commitment      | 12 months         | $0.90 per page    | $0.10 per page    | Pay full 12-month PREPAID platform fee upfront |

* The billable unit is **one physical page of the final generated WeasyPrint PDF**.
* This is the literal page count reported by WeasyPrint after PDF generation completes.
* The page count is captured server-side, stored in `metadata.billable-page-count` and `metadata.total-pages-generated`, and added to the Teacher's current billing cycle running total.
* Teacher's plan type (monthly, semi-annual, or annual) is set by the Super Admin.
* **Critical distinction**: PREPAID and POSTPAID are two **separate** payment instances. A Teacher who pays PREPAID for 6/12 months still receives monthly POSTPAID invoices for actual usage — but those invoices apply the respective discount rate.

### 23.1A Discount Commitment Mechanism

#### How Discounts Activate

1. **Annual Discount ($0.10/page)**: The Organisation Owner (Teacher) pays the complete PREPAID platform fee for a 12-month period upfront. The backend marks this Organisation's billing plan as `annual` for the next 12 billing cycles.
2. **Semi-Annual Discount ($0.05/page)**: Same mechanism, but for a 6-month period.
3. Once activated, the discount applies to all POSTPAID invoices generated during the commitment period — month-by-month.

#### Commitment Period Tracking

* If the 12-month PREPAID is paid on `01-01-2026`, the annual discount applies to all POSTPAID invoices covering the period `01-01-2026` to `31-12-2026` (12 billing cycles).
* On `01-01-2027`, the discount **automatically revokes** unless auto-renewal is active.
* Similarly for 6-month: paid on `01-01-2026` → discount applies `01-01-2026` to `30-06-2026` (6 billing cycles) → revokes on `01-07-2026`.

#### Auto-Renewal

* Teachers can enable auto-renewal in their Account Settings → Billing → Auto-Renewal.
* **Requirements for auto-renewal**:
  1. Payment details securely stored in the application (encrypted with AES-256 or equivalent, tokenised via Razorpay's secure vault / token API).
  2. **Dual-step verification** (2FA) must be enabled on the Teacher's account if auto-renewal is active.
  3. Teacher must explicitly opt-in — auto-renewal is never enabled by default.
* On commitment period expiry, if auto-renewal is ON: the system automatically charges the PREPAID amount for the next commitment period and extends the discount.
* On commitment period expiry, if auto-renewal is OFF: the Teacher reverts to monthly billing (no discount) starting from the next billing cycle.
* The Teacher receives a **reminder notification** (email + in-app) 14 days before commitment period expiry.

#### Refund Policy

* Refunds for early termination of 6-month or 12-month commitments are **not automatic**.
* The Super Admin has full discretion to issue a prorated refund, partial credit, or no refund — handled manually through the Super Admin Billing Dashboard → Teacher → "Issue Refund/Credit."
* Refunds are processed through the Razorpay refund API and logged in the audit trail.

### 23.2 Razorpay Integration Architecture

* The **Super Admin** connects a single shared/master Notesglider Razorpay account via **Super Admin Settings → Billing Configuration**.
* Required fields in billing configuration:
  * `Razorpay Key ID`
  * `Razorpay Key Secret`
* These credentials are stored **encrypted at rest** in the Neon database (AES-256 or equivalent encryption standard).
* All Teachers across all Editors are billed through the Super Admin's master Razorpay account.
* **TRUE** = A shared or master Notesglider Razorpay account exists — each Editor receives payments through the company accounts manually and never directly from the application logic.
* **Editor has no Razorpay connection** — Editors never add any Razorpay account in their settings. They have no access to billing, invoicing, or pricing data.
* Editor compensation is handled externally through the company's chartered accounts management performed by humans manually sending payments to Editors based on their work.

### 23.3 Billing Cycle Lifecycle

#### POSTPAID Billing (Default)

**Step 1 — Usage Tracking**

* Every time WeasyPrint completes a PDF generation, the page count is added to the Teacher's running total for the current billing cycle.
* Usage data is stored per-document in the `metadata.billable-page-count` field and aggregated in a dedicated billing records table.
* Each document's `billing_cycle_id` is set to the currently open billing cycle at the time of PDF generation. The `is_billed` flag on the document remains `false` until the billing cycle invoice is paid.

**Step 2 — Invoice Auto-Generation (POSTPAID)**

* **Default trigger**: the 5th date of the next calendar month at 6:00 PM IST for the calculation of the previous month's invoice.
* Example: Invoice for January 2026 usage is prepared on February 5, 2026.
* **Retrospective document sweep**: Before generating the invoice, the billing trigger runs a retrospective document scan — querying all documents where `is_billed = false` AND `count_in_billing = true` across all previous billing cycles for the Teacher. Any unbilled documents are included in the current invoice under a separate "Retrospective Documents" section (see Section 23.3A).
* Invoice contains: line items per document (document name, date, page count, rate, subtotal), total page count, total amount due, Teacher's billing plan (monthly or annual).
* Billing includes all document types processed and approved within the calendar month:
  * Daily Newsletters
  * Daily Mindmaps
  * Weekly Compilation
  * Monthly Magazine
  * Quarterly Collection
  * Bi-Annual Compendium
  * Annual Yearbook
* Each document group shows:
  * Document type label
  * Number of documents in this group
  * Total pages per document
  * Rate per page
  * Subtotal per group
* At the end: a total count of all pages for all document types is shown.
* Discount is applied if subscription type = Annual.
* Final Payable Amount is displayed.

**Step 3 — Super Admin Review & Approval**

* The auto-generated invoice enters the Super Admin's **Billing Dashboard -> Pending Invoices**.
* Super Admin reviews the invoice.
* Super Admin can edit line items if adjustments are needed (e.g. credit for a billing error).
* Super Admin clicks "Approve and Send."

**Step 4 — Invoice Delivery to Teacher**

* On Super Admin approval, the invoice is:
  * Emailed to the Teacher as a **PDF attachment** (generated via WeasyPrint)
  * Available for download in the Teacher's Account Settings -> Billing -> Invoice Archive
  * Exportable and saveable by the Super Admin

**Step 5 — Payment & Batch Update**

* Teacher pays via Razorpay (payment link in the invoice email or in-app payment CTA).
* Razorpay webhook notifies the backend of payment confirmation.
* Backend updates Teacher's billing status -> if overdue, access restoration fires automatically.
* **Batch billing metadata update**: On payment confirmation, a backend job batch-updates all documents included in the paid invoice:
  * `is_billed = true` for each document in the invoice's `line_items`.
  * `billing_cycle_id` is confirmed on each document (already set during PDF generation).
  * `billing_cleared_at` timestamp is set.
* This ensures any future billing sweep will not re-include these documents.

### 23.3A Retrospective Document Billing Rules

Retrospective documents are documents created for a **past date** that falls outside the current billing cycle. These require special handling in the billing pipeline to ensure no document ever evades payment.

#### 23.3A.1 What Makes a Document "Retrospective"

A document is flagged as retrospective (`is_retrospective = true`) when:

1. The document's **target date** (the date the document is "for", e.g. a Newsletter for March 1) falls in a **past billing cycle** that has already been invoiced and closed.
2. The document was created or submitted **after** the billing trigger for its target date's cycle has already fired.
3. Example: A Teacher creates a Newsletter dated January 15, 2026 on February 10, 2026. The January 2026 billing cycle closed on February 5, 2026. This document is retrospective.

#### 23.3A.2 Backend Detection Logic

The backend must implement the following retrospective detection logic:

1. **On document creation**: Compare the document's target date against the current open billing cycle's date range.
   * If the target date falls within the current open cycle -> `is_retrospective = false`, `billing_cycle_id = current_cycle.id`.
   * If the target date falls in any past cycle -> `is_retrospective = true`, `billing_cycle_id = NULL` (will be assigned to the next invoice).
2. **On billing trigger (5th of month)**: Before generating the invoice, query: `SELECT * FROM documents WHERE teacher_id = ? AND is_billed = false AND count_in_billing = true AND is_retrospective = true`. Include all results in the "Retrospective Documents" section of the invoice.
3. **Retrospective documents cannot be evaded**: This is a strict system requirement. The billing sweep must always catch unbilled retrospective documents regardless of how old they are. The Super Admin must be able to see all retrospective documents across all organisations at all times from the Billing Dashboard.

#### 23.3A.3 Retrospective Documents in Invoice Layout

Retrospective documents appear as a **separate section** at the bottom of the invoice, visually distinguished from the current billing cycle's documents:

```
═══════════════════════════════════════════════════════
INVOICE - BILLING CYCLE: January 2026 (01-01-2026 to 31-01-2026)
═══════════════════════════════════════════════════════

--- CURRENT CYCLE DOCUMENTS ---

[Standard document groups by type: Newsletters, Mindmaps,
 Compilations, Magazines, etc. with page counts and subtotals]

CURRENT CYCLE SUBTOTAL: $XX.XX

--- RETROSPECTIVE DOCUMENTS ---
(Documents created for past dates, not included in previous invoices)

| # | Document Name | Target Date | Created On | Type | Pages | Rate | Subtotal |
|---|---------------|-------------|------------|------|-------|------|----------|
| 1 | NL-20251215-0003 | 15-12-2025 | 10-02-2026 | Newsletter | 4 | $1.00 | $4.00 |
| 2 | NL-20260105-0002 | 05-01-2026 | 12-02-2026 | Newsletter | 3 | $1.00 | $3.00 |

RETROSPECTIVE SUBTOTAL: $7.00

═══════════════════════════════════════════════════════
TOTAL PAGES: XX | TOTAL AMOUNT: $XX.XX
DISCOUNT (Annual): -$X.XX
FINAL PAYABLE: $XX.XX
═══════════════════════════════════════════════════════
```

#### 23.3A.4 UI Indicators for Retrospective Documents

Throughout the application, retrospective documents must have clear visual indicators:

* **Teacher's Kanban**: Retrospective documents display a "Retrospective" badge (distinct colour, e.g. amber/orange) with a tooltip: _"This document was created for a past date (\[target date]). It will be included in your next billing cycle."_
* **Editor's Kanban**: Same badge. Editor can process retrospective documents identically to normal documents — no workflow difference.
* **Super Admin's Billing Dashboard**: A dedicated "Retrospective Documents" filter/toggle showing all unbilled retrospective documents across the system, grouped by Organisation.
* **Document Detail View**: A persistent banner at the top: _"This is a retrospective document created on \[created\_at] for \[target\_date]. Billing: \[Pending / Included in Invoice #XXX / Paid]."_

#### 23.3A.5 Retrospective Documents from Previous Academic Year

When a Teacher creates a document for a date in the **previous academic year** (while the Cloudinary data retention grace period is still active):

1. The document is flagged: `is_retrospective = true`.
2. The document includes a **UI-level warning banner**: _"This document is for the previous academic year (\[YYYY]). Image data for this year will be archived on \[archival\_date]. After that date, retrospective document creation for this academic year will no longer be possible."_
3. **Billing**: The document is counted in the **next upcoming billing cycle** regardless of which academic year or billing period the target date belongs to. It will never be retroactively inserted into a past, already-paid billing cycle.
4. **After grace period expiry**: If Cloudinary data retention has triggered auto-deletion for the previous academic year (`cloudinary_data_retention_active = false`), the backend must **block** retrospective document creation for dates in that academic year. The UI shows: _"Retrospective document creation for academic year \[YYYY] is no longer available. Image data has been archived."_

#### PREPAID Billing (Optional)

* The Super Admin can optionally charge the Teacher a **platform fee** before any document creation is allowed.
* PREPAID invoicing is **fully freeform** — Super Admin types any amount they want per invoice per Teacher.
* When PREPAID is enabled (`prepaid_invoicing = true`), all functionality is gatekept from the Teacher until the PREPAID invoice is cleared — not even PPTX export (which is allowed during POSTPAID grace period) is permitted.
* **Edge case**: The Super Admin can set PREPAID to `false` when onboarding a new client and later switch to `true` for subsequent billing cycles to ease adoption resistance.
* PREPAID and POSTPAID can coexist: a Teacher may have both a cleared PREPAID platform fee AND ongoing POSTPAID usage billing.
* **Commitment-linked PREPAID**: When a Teacher opts for a 6-month or 12-month commitment, the PREPAID platform fee is charged for the full commitment period upfront. This payment simultaneously activates the per-page discount on all POSTPAID invoices during the commitment window (see Section 23.1A).

### 23.3B Document Billability Rules (`is_billable`)

Every document carries an `is_billable` boolean flag that determines whether it is included in POSTPAID invoice calculations.

#### Default Behaviour

| Document Creator | Default `is_billable`              | Rationale                                                                            |
| ---------------- | ---------------------------------- | ------------------------------------------------------------------------------------ |
| Teacher          | `true`                             | Teacher-created documents are always billable by default                             |
| Sub-member       | `true`                             | Sub-member documents serve the Organisation's purpose                                |
| Editor           | **Depends on purpose** (see below) | Editor may create for proofreading (non-billable) or on behalf of Teacher (billable) |

#### Editor Document Purpose Selection

When an Editor creates a document within an Organisation, the system presents a **purpose selection UI** before creation:

| Purpose        | `is_billable` Value | Description                                                                                                             |
| -------------- | ------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `proofreading` | `false`             | Editor is creating the document for internal cross-checking, QA, or proofreading. Not billed to Teacher.                |
| `processing`   | `true`              | Editor is creating the document on behalf of the Teacher for the Organisation's operational purpose. Billed to Teacher. |

* The purpose selection is **mandatory** — the Editor cannot proceed with document creation without selecting a purpose.
* The selected purpose is stored in `document.editor_creation_purpose` (nullable — only set for Editor-created documents).
* The purpose can be changed later by the Editor before the document enters the pipeline, but once submitted to the pipeline, the purpose and `is_billable` flag are **locked**.

#### Super Admin Override During Invoice Generation

During the invoice review process (Step 3 of Section 23.3), the Super Admin has the following powers over document-level billing:

1. **Toggle `is_billable`**: Super Admin can flip `is_billable` from `true` to `false` (or vice versa) for any individual document in the invoice, regardless of who created it.
2. **Remove a document from the invoice**: Super Admin can exclude any specific document from the current invoice entirely. The document remains in the system but is not charged.
3. **Apply per-document discount**: Super Admin can apply a percentage discount (0–100%) to any individual document's billing line item. This is independent of the plan-level discount.
4. **All overrides are logged** in `audit_logs` with `event_type = 'billing_override'` including the before/after values.

> **Important**: The Super Admin has access to ALL documents in the shared pool — both `is_billable = true` and `is_billable = false` — during invoice generation. This provides full transparency for billing decisions.

### 23.4 Manual Invoice Generation

* **Super Admin can generate a manual invoice** for any Teacher at any time from the Billing Dashboard → select Teacher → "Generate Invoice."
* Super Admin can also trigger invoicing for a specific date range or filtered set of documents.
* **Teacher can request an invoice** at any time from their Account Settings → Billing → "Request Invoice."
  * This request is forwarded to the **Super Admin** (not the Editor) for review and approval.

### 23.5 Invoice Archive

* All invoices (auto-generated and manual) are stored in the database permanently.
* Both Teacher and Super Admin can access the full invoice archive with on-demand download.
* Archive is filterable by date range and document type.
* Invoices can be exported as PDF (via WeasyPrint), sent via email as PDF attachment, or saved locally.

### 23.6 Non-Payment Access Policy

<table><thead><tr><th width="171.4921875">Phase</th><th width="166.97265625">Duration</th><th>Teacher Access State</th></tr></thead><tbody><tr><td>Paid and current</td><td>N/A</td><td>Full access</td></tr><tr><td>Invoice overdue — Grace Period</td><td>7 days from due date</td><td>Full access. Persistent in-app banner: <em>"You have an outstanding invoice due [date]. Please clear your balance."</em></td></tr><tr><td>Invoice overdue — Post Grace Period</td><td>Until payment received</td><td>"Send to Editor" button disabled. Tooltip: <em>"Your account has an outstanding balance. Please clear your dues to submit documents for PDF generation."</em> Document creation and PPTX export remain active.</td></tr><tr><td>Payment cleared</td><td>Immediately</td><td>Full access restored automatically via Razorpay webhook — no manual intervention required. Backend automatically schedules documents created during grace period where <code>status = approved</code> and queues them for Editor review. Email and in-app notification acknowledges functionality restoration to the Organisation owner and sub-members.</td></tr></tbody></table>

### 23.7 Super Admin Billing Dashboard Specification

The Super Admin has a comprehensive invoicing dashboard with system-wide tenant information:

* **System-wide view**: All Editors' work across their respective enrolled Organisations
* **Per-Editor view**: Individual Editor's work for their specific enrolled Organisations
* **Per-Teacher view**: Complete document metadata including all document types (generated / revised / currently in queue / not yet sent to editor), sortable, filterable for a specified period or since inception
* **Default scope**: Current calendar month
* **Filter controls**: Date range picker (any range), Editor name selector, Teacher name selector (multi-select), Document type selector (multi-select)
* **Metrics per Teacher row**:
  * Total PDF pages generated (billable units this period)
  * Number of documents processed per type
  * Rate per page (monthly or annual)
  * Total amount due
  * Payment status
* **Visualisations**:
  * Line chart: page count over time (x-axis: days, y-axis: pages)
  * Bar chart: document type breakdown
  * Raw activity data table with all events (sortable, filterable)
* **Sortable by**: Teacher name, Editor name, total page count, total documents, total amount, processing date
* **Export**: Download billing data as CSV or PDF for any filter selection

### 23.8 Evader Detection & Compliance Enforcement

#### 23.8.1 Problem Statement

It is possible that an Organisation Owner (Teacher) could exploit the platform by using all non-PDF document capabilities (document creation, PPTX export, presentations, translations) while intentionally avoiding the PDF pipeline — thereby evading billable usage. Since billing is based on PDF pages generated, a Teacher who never submits documents for PDF processing effectively uses the platform for free.

#### 23.8.2 Existing Safeguards (Already in Place)

The following existing mechanisms provide partial protection:

1. **Super Admin system-wide visibility**: The Super Admin can view all documents, logs, and activity across all Organisations.
2. **Super Admin suspension authority**: The Super Admin can suspend any pipeline or document type at will.
3. **POSTPAID invoice blocking**: Outstanding POSTPAID invoices auto-block PDF pipeline submission after the grace period, preventing document creation abuse during non-payment.

However, these mechanisms do not detect the **intentional** pattern of a Teacher who creates documents regularly but systematically avoids the PDF pipeline.

#### 23.8.3 Compliance Threshold

The compliance rule is measured **per-publication-stream** within each Organisation:

> **At least N% of all atomic (single-date) documents created within a publication stream must be submitted to the Editor for PDF pipeline processing within each billing cycle.**

* **Default threshold**: 75%
* **Configurable**: Super Admin can set a custom threshold per Organisation in Super Admin Settings → Organisation Management → \[Org Name] → Compliance Settings.
* **Measured against**: Only **atomic** document types count (single-date documents like Newsletters). Aggregated documents (Compilations, Magazines, etc.) are excluded from the compliance calculation since they are derived from atomic documents.
* **Per-publication**: Each publication stream is evaluated independently. An Organisation with `CURRENT-AFFAIRS` at 90% compliance and `SCIENCE-OUTLOOK` at 50% compliance is flagged only for the `SCIENCE-OUTLOOK` stream.

#### 23.8.4 Grace Period for New Organisations

* The first **2 complete billing cycles** after Organisation activation are treated as a **grace period**.
* During the grace period:
  * No compliance enforcement is applied — the Teacher is free to explore all platform features.
  * The platform absorbs the full cost of non-PDF usage during this period.
  * However, the system **silently monitors** compliance metrics during this period.
  * If the Organisation falls below the threshold during the grace period, the Organisation Owner is **flagged internally** and reported to the Super Admin via **email + in-app notification**: _"\[Organisation Name] has used only \[X%] PDF pipeline processing during their trial period. Monitoring continues — no action required at this time."_
* The 2-cycle grace period is **per-Organisation**, not per-publication. All publications in a new Organisation share the same grace window.

#### 23.8.5 Post-Grace Enforcement Actions

After the 2-cycle grace period, if any publication stream within an Organisation falls below the compliance threshold, the system triggers the following escalation path. The Super Admin has **exclusive authority** to choose from these actions:

**Action 1: `allow` — Set Exception**

The Super Admin grants a formal exception for the specific non-compliant scenario:

* **Exception template**: _"Allow non-compliance of {X}% rule for {Organisation Name} managed by {Organisation Owner} with sub-members \[{sub-member-1}, ... {sub-member-n}] and mapped Editor {Editor Name} for document type = {document-type} under publication = {publication-stream} in current AY = {academic\_year}."_
* The exception is logged in `audit_logs` with full details.
* The exception applies for the current academic year only — it must be re-evaluated at year boundary.

**Action 2: `warning` — Issue Compliance Warning**

* The Super Admin triggers a **formal warning** sent to the Organisation Owner via email + in-app notification:
  * _"Your Organisation \[{Organisation Name}] is not meeting the minimum PDF processing compliance of \[{X}%] for publication \[{publication-stream}]. Only \[{Y}%] of your atomic documents have been submitted for PDF processing. If compliance is not met by the end of the current billing cycle, your Organisation will be temporarily suspended starting next billing cycle."_
* The warning is visible as a **persistent banner** in the Teacher's dashboard.
* The warning auto-escalates to `temporary-suspension` in the next billing cycle if the Organisation Owner does not reach compliance.

**Action 3: `temporarily-suspend` — Suspend Organisation**

* The Super Admin suspends the Organisation: **all participants** (Organisation Owner, Editor, Sub-members) lose access to the application.
* All affected users receive email + in-app notification explaining the suspension and the compliance requirement.
* **Resumable**: If the Organisation Owner agrees to follow compliance requirements, the Super Admin can lift the suspension. All assets, documents, and access are restored to normal immediately.
* During suspension, all data is preserved — no deletion occurs.

**Action 4: `terminate` — Permanently Terminate Organisation**

* The Super Admin **permanently terminates** the Organisation.
* All Organisation-wide data is **permanently deleted** from the backend database.
* Access is revoked for all participants: Organisation Owner, Editor (for this Org), and all Sub-members.
* All affected users receive email + in-app notification.
* **This action is destructive and non-resumable.** All data is permanently lost.
* Requires a **double-confirmation dialog** in the Super Admin UI: _"This action is irreversible. All data for \[{Organisation Name}] will be permanently deleted. Type the Organisation name to confirm."_

#### 23.8.6 Super Admin Compliance Dashboard

The Super Admin's Billing Dashboard includes a **Compliance** tab showing:

| Column                     | Description                                                                      |
| -------------------------- | -------------------------------------------------------------------------------- |
| Organisation Name          | Name of the flagged Organisation                                                 |
| Publication Stream         | Specific publication stream that is non-compliant                                |
| Compliance %               | Current percentage of atomic documents submitted for PDF pipeline                |
| Threshold                  | Configured threshold for this Organisation                                       |
| Billing Cycles Since Grace | Number of billing cycles since grace period ended                                |
| Current Status             | `grace-period` / `compliant` / `flagged` / `warned` / `suspended` / `terminated` |
| Last Action                | Most recent Super Admin action (if any)                                          |
| Action Buttons             | `Allow` / `Warning` / `Suspend` / `Terminate`                                    |

* Real-time filterable by status, Organisation, publication stream, and compliance percentage range.
* Exportable to CSV.
