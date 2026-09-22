---
description: Multi Tenant Application - Role Based Access Control
icon: shield-check
---

# RBAC

## SECTION 4 — USER ROLES & RBAC SYSTEM

### 4.1 Architecture: Multi-Tenant Hierarchically Delegated RBAC

Notesglider implements a **classic multi-tenant, hierarchically delegated RBAC** architecture — the same pattern used by production SaaS platforms such as Google Workspace, Salesforce, and Notion. Every role can only see and act **downward** within its own scope. No lateral or upward access is possible at any level. This enforces the **principle of least privilege** across the entire platform.

#### The 5-Layer Role Hierarchy

```
┌─────────────────────────────────────────────────────┐
│           SUPER ADMIN  (System Level)               │  ← Platform Owner
├─────────────────────────────────────────────────────┤
│    EDITOR-1          EDITOR-2         EDITOR-N      │  ← Tenant Managers
├──────────────┬──────────────────────────────────────┤
│    ORG-1     │   ORG-2          ORG-3               │  ← Tenant Containers (Isolated)
├──────────────┴──────────────────────────────────────┤
│  TEACHER-1   TEACHER-2     TEACHER-3                │  ← Org-Level Admins
├─────────────────────────────────────────────────────┤
│  sub-m  sub-m  sub-m  sub-m  sub-m  sub-m  sub-m    │  ← End Users
└─────────────────────────────────────────────────────┘
```

### 4.2 Login Role Types

There are exactly **four login role types** in the system. No other roles exist beyond these:

| Role            | Identifier    | Layer          | Description                                                                                                              |
| --------------- | ------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **Super Admin** | `super_admin` | System         | Platform root authority. Spawns Editors, approves Organisations, maps tenants. Operates outside any tenant scope.        |
| **Editor**      | `editor`      | Tenant Manager | Manages one or more Organisations assigned by Super Admin. Runs PDF pipeline, billing, and review queue for mapped Orgs. |
| **Teacher**     | `teacher`     | Org Admin      | Creates and administers their Organisation. Creates documents, generates PPTX, manages sub-members.                      |
| **Sub-member**  | `sub-member`  | End User       | Team member enrolled by a Teacher. Restricted access scoped to parent Teacher's document pool.                           |

> **Metadata Clarification**: The Newsletter JSON schema contains `access-roles` values such as `reviewer`, `publisher`, and `admin`. These are document-level metadata labels only — not separate login identities. When the Editor reviews a document, the `reviewed-by` field is populated with the Editor's identity. When approved, `approved-by` is populated with the Editor's identity. The `publisher` and `admin` metadata labels always resolve to the Editor role at runtime.

### 4.3 The Organisation — Tenant Container (Not a Role)

An **Organisation** is not a user role — it is the **tenant isolation boundary** of the system. Think of it as Google Workspace treats a domain, or how Slack treats a workspace.

* Each Organisation is created by a Teacher during self-registration.
* An Organisation begins with `status: pending_approval` and has zero operational access until the Super Admin explicitly authorises it.
* Once authorised by the Super Admin, the Organisation is mapped to a specific Editor.
* An Organisation is **fully isolated**: no cross-org reads, writes, or role assignments are possible under any circumstance.
* Role assignments are always **org-scoped** — a Teacher in Org-1 has zero visibility into Org-2, even if the same person were to hold accounts in both.

**Organisation Lifecycle:**

```
Teacher self-registers + creates Org  →  org.status = pending_approval
Super Admin reviews + authorises      →  org.status = active
Super Admin maps Org to an Editor     →  org.editor_id = assigned Editor
Editor authorises Teacher account     →  teacher.status = active
Teacher can now enroll sub-members
```

### 4.4 Role 1: Super Admin — Capabilities

The Super Admin is the **root authority** of the entire system. This role operates **entirely outside any tenant scope** — it has no Organisation assignment and its data scope is `tenant_id = NULL` (system-level).

**The Super Admin is a single designated account** — the platform software owner. There is no UI for teachers or editors to create or discover Super Admin accounts. Super Admin credentials are provisioned at system setup time by the developer/platform owner.

| Capability                     | Details                                                                                                                                                                                    |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Spawn Editor accounts          | Create new Editor credentials in the system; account begins as `status: pending`                                                                                                           |
| Authorize Editors              | Explicitly activate an Editor account — `status: pending → active`; no Editor can operate without this step                                                                                |
| Authorize Organisations        | Review and approve incoming Organisation registration requests from Teachers — `status: pending_approval → active`                                                                         |
| Map Organisations to Editors   | Assign which Editor is responsible for a given Teacher's Organisation                                                                                                                      |
| Reassign Organisations         | Move a Teacher's Organisation from one Editor to another at any time                                                                                                                       |
| System-wide visibility         | View logs, metrics, activity trails, and account statuses for every entity across the entire platform — all tenants                                                                        |
| Suspend any account            | Suspend, reactivate, or permanently deactivate any account at any level (Editor, Teacher, Sub-member)                                                                                      |
| Platform configuration         | Manage global system settings, billing tier defaults, and platform-level policies                                                                                                          |
| Billing & invoicing management | Full ownership of all billing activities: set pricing, generate invoices, receive payments via Razorpay, manage POSTPAID and PREPAID billing modes. Invoice PDF generation via WeasyPrint. |
| **Cannot**                     | Perform document-level operations (create, edit, submit Newsletters) — that is the Teacher's domain                                                                                        |

**Data scope**: Queries span all tenants — `WHERE tenant_id IS NULL` or unrestricted. Super Admin sees everything.

### 4.5 Role 2: Editor — Capabilities

An Editor is a **Tenant Manager** — they own and oversee one or more Organisations that have been mapped to them by the Super Admin. This is delegated administration: the Super Admin offloads day-to-day tenant governance to the Editor.

**Key constraint**: An Editor's visibility is strictly bounded to their mapped Organisations. They cannot access Organisations assigned to other Editors. They cannot spawn other Editors (only the Super Admin can do this).

| Capability                      | Details                                                                                                                                                                                  |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Authorize Teacher accounts      | Activate Teacher accounts within their mapped Organisations — delegated from Super Admin                                                                                                 |
| Review queue management         | View, edit, and process all documents submitted by Teachers in their Organisations                                                                                                       |
| PDF generation pipeline         | Trigger WeasyPrint PDF generation for all document types                                                                                                                                 |
| Mindmap generation              | Create, edit, and approve Mindmaps                                                                                                                                                       |
| Document export                 | Export all document types on demand (on par with Teacher export capabilities)                                                                                                            |
| HTML webpage export             | Open documents in iframe or new tab from `.html`; export data as a webpage bundle (`.html`, `.css`, and image assets bundled locally, CDN URLs rewritten to relative paths)              |
| Document metrics dashboard      | View month-wise dashboard showing document processing metrics (document counts, page counts, processing times) grouped by Teacher and document type — without pricing or invoicing data  |
| Drive management                | Full CRUD on the Drive folder structure within their tenant scope                                                                                                                        |
| Monitor mapped Org activity     | View logs, metrics, and usage data for all Teachers and sub-members within their assigned Organisations                                                                                  |
| Suspend Teachers                | Suspend or flag Teacher accounts within their mapped scope                                                                                                                               |
| System settings (tenant-scoped) | Configure retry counts, SLA defaults, notification preferences, aggregation trigger times for their Organisations                                                                        |
| **Cannot**                      | Access Organisations mapped to other Editors, spawn other Editor accounts, view system-wide metrics across all Editors, access billing/invoicing/pricing data, connect Razorpay accounts |

**Scoping rule**: Every Editor database query is predicated on `WHERE org_id IN (SELECT id FROM organisations WHERE editor_id = current_editor_id)` to prevent cross-tenant privilege bleed.

### 4.6 Role 3: Teacher — Capabilities

A Teacher is the **Org-level Admin** — they function as the workspace owner within the strict boundaries of their single Organisation.

| Capability               | Details                                                                                                                                                                                                                                                                     |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Create Organisation      | Self-registers and creates their Organisation during signup (submitted to Super Admin for approval)                                                                                                                                                                         |
| Document creation        | Create Newsletters using the canvas block editor                                                                                                                                                                                                                            |
| PPTX export              | Generate `.pptx` from any finalised Newsletter                                                                                                                                                                                                                              |
| Reveal.js presentation   | Launch live presentation mode from any finalised Newsletter                                                                                                                                                                                                                 |
| Send to Editor           | Submit finalised documents to the Editor's review queue                                                                                                                                                                                                                     |
| Export                   | Download `.pptx`, `.pdf` (via Reveal.js print), `.txt`, `.zip`                                                                                                                                                                                                              |
| Translate                | Request Google Translate of a document into a target language                                                                                                                                                                                                               |
| Sub-member management    | Invite, configure permissions for, and manage sub-members within their Org                                                                                                                                                                                                  |
| Authorize sub-members    | Approve or reject sub-member account activation requests                                                                                                                                                                                                                    |
| View sub-member activity | Full activity log of all sub-members within their Org                                                                                                                                                                                                                       |
| Suspend sub-members      | Suspend or remove sub-members from their Organisation                                                                                                                                                                                                                       |
| Ad banner management     | Upload and schedule ad banners per document type                                                                                                                                                                                                                            |
| Templates                | Configure branded templates (headers, footers, full pages) per document type via the drag-and-drop Template System (Section 19)                                                                                                                                             |
| YouTube attachment       | Attach a video from their channel to a Newsletter                                                                                                                                                                                                                           |
| Settings                 | Configure notifications, Drive sync, academic year, data retention policy                                                                                                                                                                                                   |
| **Cannot access**        | PDF pipeline, Mindmap generation, Compilation/Magazine or any other form of document creation behind the scenes workflow abstracted from them apart from CRUD the newsletter designated for their role scope, billing panel, Editor's queue view, other Organisations' data |

**Key constraint**: A Teacher's administrative visibility is **strictly bounded to their own Organisation**. They cannot see other Teachers, other Orgs, or any Editor/Super Admin data.

### 4.7 Role 4: Sub-member — Capabilities

Sub-members are the **leaf nodes** of the hierarchy — enrolled by Teachers. They represent the primary end-users of the application's core document features.

* Sub-members belong to a Teacher's **Organisation** — not the Editor's scope directly.
* Each sub-member has their own independent app login with role = `sub-member`.
* Sub-member access is scoped **exclusively** to the parent Teacher's document pool — no cross-Organisation visibility.
* The parent Teacher has a **per-sub-member permission configuration panel** within Team Settings.
* Configurable permissions per sub-member (independently toggleable per person):
  * Document creation rights
  * Document editing rights
  * Document submission-to-Editor rights
  * Export rights
* All permissions are enforced server-side on every relevant API endpoint — client-side hiding is a UX convenience only and never a security substitute.
* Sub-members **cannot**:
  * Manage or enroll other sub-members
  * Access billing or settings outside their own account preferences
  * See documents outside the parent Teacher's pool
  * Change their own permission level
  * Perform any administrative action within the Organisation

**Account activation**: Sub-member self-registers or receives an invite link from a Teacher. Account sits as `status: pending` until the Teacher explicitly authorises it.

### 4.8 Sub-member Activity Logging

* The Teacher has a dedicated **Team Activity Log** view in their dashboard.
* Every sub-member action is recorded with: action type, document affected, timestamp, sub-member identity.
* This log is read-only for the Teacher — they cannot edit or delete log entries.
* The Editor (and Super Admin) also have scope-inherited visibility into sub-member activity through their respective dashboard views.

### 4.9 Complete Permission Matrix

| Capability                                              | Super Admin |           Editor          |         Teacher         |  Sub-member |
| ------------------------------------------------------- | :---------: | :-----------------------: | :---------------------: | :---------: |
| Spawn Editor accounts                                   |     true    |         false (X)         |        false (X)        |  false (X)  |
| Authorize Editor accounts                               |     true    |         false (X)         |        false (X)        |  false (X)  |
| Authorize Organisation creation                         |     true    |         false (X)         |        false (X)        |  false (X)  |
| Map Organisation -> Editor                              |     true    |         false (X)         |        false (X)        |  false (X)  |
| Reassign Organisation to different Editor               |     true    |         false (X)         |        false (X)        |  false (X)  |
| Suspend any account (system-wide)                       |     true    |         false (X)         |        false (X)        |  false (X)  |
| View system-wide logs & metrics (all tenants)           |     true    |         false (X)         |        false (X)        |  false (X)  |
| View sub-member activity/logs (system-wide)             |     true    |         false (X)         |        false (X)        |  false (X)  |
| Suspend/remove sub-members (system-wide)                |     true    |         false (X)         |        false (X)        |  false (X)  |
| Authorize Teacher accounts (in mapped Orgs)             |     true    |            true           |        false (X)        |  false (X)  |
| View logs/metrics for mapped Orgs                       |     true    |            true           |        false (X)        |  false (X)  |
| Suspend Teachers in mapped Orgs                         |     true    |            true           |        false (X)        |  false (X)  |
| Run PDF pipeline, Mindmap generation                    |  false (X)  |            true           |        false (X)        |  false (X)  |
| Manage billing, invoicing & Razorpay                    |     true    |         false (X)         |        false (X)        |  false (X)  |
| View document metrics dashboard (no pricing)            |  false (X)  |            true           |        false (X)        |  false (X)  |
| Export all document (non PDF pipeline) types on demand  |  false (X)  |            true           |           true          |   per-perm  |
| PDF pipeline exports (branded WeasyPrint PDF)           |  false (X)  |            true           |        false (X)        |  false (X)  |
| HTML webpage export (bundled assets, local image paths) |  false (X)  |            true           |        false (X)        |  false (X)  |
| Create Organisation (self-register)                     |  false (X)  |         false (X)         |           true          |  false (X)  |
| Authorize sub-member accounts                           |  false (X)  |         false (X)         |           true          |  false (X)  |
| Invite/enroll sub-members                               |  false (X)  |         false (X)         |           true          |  false (X)  |
| Configure sub-member permissions                        |  false (X)  |         false (X)         |           true          |  false (X)  |
| View sub-member activity/logs (own Org)                 |  false (X)  |         false (X)         |           true          |  false (X)  |
| Suspend/remove sub-members (own Org)                    |  false (X)  |         false (X)         |           true          |  false (X)  |
| Create documents                                        |  false (X)  |            true           |           true          |   per-perm  |
| Add comments to any org document                        |  false (X)  |            true           |           true          |     true    |
| Approve new publication stream requests                 |     true    |         false (X)         |        false (X)        |  false (X)  |
| Suspend/block publication or document type              |     true    |         false (X)         |        false (X)        |  false (X)  |
| Create new publication (direct)                         |  false (X)  |         false (X)         | true (req. SA approval) |  false (X)  |
| Request new publication                                 |  false (X)  | true (req. T+SA approval) |           N/A           |  false (X)  |
| Configure publication (doc types, triggers)             |  false (X)  |            true           |           true          |  false (X)  |
| View publication settings                               |     true    |            true           |           true          | true (view) |
| Override `is_billable` during invoice review            |     true    |         false (X)         |        false (X)        |  false (X)  |
| Apply per-document discount on invoice                  |     true    |         false (X)         |        false (X)        |  false (X)  |
| Manage compliance thresholds (per org)                  |     true    |         false (X)         |        false (X)        |  false (X)  |
| Raise support ticket                                    |  false (X)  |            true           |           true          |     true    |
| View & resolve support tickets                          |     true    |         false (X)         |        false (X)        |  false (X)  |
| View own activity                                       |     true    |            true           |           true          |     true    |

### 4.10 Core RBAC Architectural Principles (Non-Negotiable)

These are the immutable pillars governing the entire system:

| Principle                    | Implementation Requirement                                                                                                                                                                                                                                                                                                               |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Least Privilege**          | Every role is granted only the minimum permissions required. No implicit escalation at any layer.                                                                                                                                                                                                                                        |
| **Explicit Authorization**   | No account auto-activates. Every entity requires a deliberate approval action from the role above it in the hierarchy.                                                                                                                                                                                                                   |
| **Tenant Isolation**         | No cross-org reads or writes. Every query touching user, document, or billing data is predicated on `org_id`.                                                                                                                                                                                                                            |
| **Delegated Administration** | Super Admin delegates Org governance to Editor. Editor delegates member management to Teacher. Each level is autonomous within its scope.                                                                                                                                                                                                |
| **Managed Audit Trail**      | Every authorization, activation, suspension, and data access event is appended to an audit log. Logs are append-only during normal operation. Only the Super Admin can manage audit log deletion (with archival to Google Drive and CSV export before deletion). Automated deletion schedules can be configured in Super Admin settings. |
| **Scope-Bounded Visibility** | Metrics, logs, and user lists are visible only within your own scope. An Editor cannot see another Editor's Org. A Teacher cannot see another Org's Teachers.                                                                                                                                                                            |
| **Status-Gated Access**      | A `pending` account cannot perform any actions regardless of role assignment. The `status` field is checked at middleware layer before any permission resolution.                                                                                                                                                                        |

### 4.11 Document Pool Visibility & Permissions

All documents within an Organisation belong to a **shared document pool** that is visible — with varying levels of access — to every authorised member of that Org. The principle is: the pool is collectively visible, but CRUD permissions depend on who created the document and your role.

#### 4.11.1 Visibility Rules by Creator

| Document Creator                       | Editor (assigned to Org)                                                 | Teacher (Org owner)                                | Sub-member (authorised)                                               |
| -------------------------------------- | ------------------------------------------------------------------------ | -------------------------------------------------- | --------------------------------------------------------------------- |
| **Teacher-created**                    | View-only until Teacher submits for pipeline; then full pipeline control | Full CRUD (owner)                                  | Read + Edit (if Teacher grants permission); **cannot delete**         |
| **Sub-member-created**                 | Full CRUD (including delete)                                             | Full CRUD (including delete)                       | Full CRUD on own documents; Read-only on other Sub-members' documents |
| **Editor-created** (for Teacher's Org) | Full CRUD (owner)                                                        | View-only; badge displayed: _"Created by: Editor"_ | View-only; badge displayed: _"Created by: Editor"_                    |

#### 4.11.2 Attribution & Badges

* Every document card/row displays the creator's name and role: `Created by: [Name] ([Role])`.
* Editor-created documents carry a distinct **"Created by: Editor"** badge visible to Teacher and Sub-members. This ensures transparency — Teacher always knows which documents were created on their behalf.
* Sub-member-created documents show `Created by: [Sub-member Name]` visible to Teacher and Editor.
* The `created_by_id` and `created_by_role` fields in the `documents` table enforce this attribution at the data layer (see Section 29 [#documents](../database/schema-overview.md#documents "mention")).

#### 4.11.3 Document Comments (Dispute Resolution)

Every document in the pool supports a **simple linear comment thread** for communication and dispute resolution between roles.

* **Who can comment**: Editor, Teacher, and Sub-member — any authorised member of the Org can add a comment to any document visible to them.
* **Thread structure**: Chronological (newest at bottom), flat (no nesting/replies-to). Each comment records: author name, role badge, timestamp, and message body.
* **Use cases**: Flagging errors, requesting revisions, clarifying content, recording editorial decisions, resolving disputes about document content.
* **Visibility**: All comments on a document are visible to all roles that can view that document. Comments are append-only for non-admin roles — once posted, a comment cannot be edited or deleted by the author. Only Super Admin can delete comments (via audit action).
* **Schema**: See `document_comments` table in Section 29.

#### 4.11.4 Audit Logging

All document pool actions — create, edit, delete, status change, comment addition — are recorded in `audit_logs` with the acting user's ID, role, timestamp, and action type. This provides a complete chain-of-custody for every document change.
