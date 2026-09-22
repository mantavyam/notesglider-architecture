---
icon: signal-stream
---

# Publication Streams

## SECTION 9A — PUBLICATION STREAMS (MULTI-STREAM DOCUMENT ARCHITECTURE)

### 9A.1 Overview

An Organisation can operate **multiple independent publication streams**, each representing a distinct editorial pipeline with its own set of document types. This enables a single Organisation to manage parallel content verticals (e.g. "Current Affairs" and "Science Outlook") under one tenant, each with independent aggregation schedules, custom naming, and separate billing tracking.

Every Organisation must have **at least one publication**. The system creates a default publication named `CURRENT-AFFAIRS` on Organisation creation.

### 9A.2 Default Publication

| Property       | Value                                                                                                                                                  |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Name           | `CURRENT-AFFAIRS`                                                                                                                                      |
| Created by     | System (auto-created on Organisation registration)                                                                                                     |
| Deletable      | No — at least one publication must exist at all times                                                                                                  |
| Document types | All standard types as defined in Section 9.1 (Newsletter, Mindmap, Compilation, Magazine, Quarterly Collection, Bi-Annual Compendium, Annual Yearbook) |

The default publication uses the standard trigger schedules defined in Sections 9.2, 9.3, 9.3A, 9.3B, and 9.3C.

### 9A.3 Custom Publications

#### Who Can Create

* **Teacher (Organisation Owner)**: Can create additional publications from Organisation Settings → Publication Management → "Create New Publication."
* **Editor**: Can request creation of a new publication on behalf of the Teacher. This request requires explicit approval from the Teacher, who in turn requires approval from the Super Admin. Once approved, access is passed down the chain: Super Admin → Teacher → Editor. Both Teacher and Editor can then manage the publication's document types and configuration.
* **Sub-member**: View-only access to publication settings. Cannot create or configure publications.

#### Publication Approval Flow

```
Editor requests new publication
  → Teacher receives approval request
    → Teacher approves and forwards to Super Admin
      → Super Admin approves
        → Publication is created, access granted to Teacher + Editor
```

If the Teacher creates the publication directly:

```
Teacher creates new publication
  → Super Admin receives approval request
    → Super Admin approves
      → Publication is activated
      → Mapped Editor automatically gains managed access
```

#### Custom Publication Properties

| Property                  | Description                                                                                                                                                      |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Publication Name          | Unique within the Organisation. e.g. `SCIENCE-OUTLOOK`, `HISTORY-DIGEST`                                                                                         |
| Atomic Document Type      | **At least one atomic (single-date) document type is mandatory** — this is the fundamental unit from which all aggregations are built. e.g. `NEWSLETTER-SCIENCE` |
| Aggregated Document Types | Optional. Custom aggregation types with configurable ranges. e.g. `WEEKLY-SCIENCE-COMPILATION`, `MONTHLY-SCIENCE-MAGAZINE`                                       |
| Trigger Schedules         | Each publication has its own **independent** aggregation trigger schedule, configurable per document type                                                        |

### 9A.4 Custom Document Types Within a Publication

#### Atomic Document Types (Mandatory — at least one)

* The fundamental single-date unit of content. Equivalent to "Newsletter" in the default publication.
* Used as the building block for all aggregated document types in the same publication.
* Each atomic document type gets a **custom name** that must be **unique across all publications within the same Organisation** (cross-publication uniqueness enforced).
* Example: A publication `SCIENCE-OUTLOOK` could have an atomic type `DAILY-SCIENCE-BRIEF`.

#### Aggregated Document Types (Optional)

Custom aggregated documents can be configured with:

| Property          | Description                                                                                                                                                                     |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Custom Name       | Must be unique across all publications in the same Organisation. e.g. `BI-WEEKLY-SCIENCE-REVIEW`                                                                                |
| Aggregation Range | Custom configurable range: Weekly (7 days), Bi-Weekly (14 days), Monthly (calendar month), Quarterly (3 months), Semi-Annual (6 months), Annual (12 months), or Custom (N days) |
| Auto Trigger      | Configurable trigger date and time for automatic aggregation. Same format as Sections 9.2–9.3C. Each publication's triggers are independent.                                    |
| Manual Trigger    | Editor can always manually trigger aggregation at any time, regardless of auto-trigger settings                                                                                 |
| Source            | Aggregation pulls only from atomic document types within the **same publication** — never cross-publication                                                                     |

#### Naming Uniqueness Constraint

**Document type names must be unique across ALL publications within the same Organisation.** This prevents ambiguity in billing, Kanban boards, and document references.

* Valid: `CURRENT-AFFAIRS` has `NEWSLETTER`; `SCIENCE-OUTLOOK` has `NEWSLETTER-SCIENCE`
* Invalid: `CURRENT-AFFAIRS` has `NEWSLETTER`; `SCIENCE-OUTLOOK` also has `NEWSLETTER` ← **REJECTED — duplicate name**

The backend must enforce this at the database level with a unique constraint on `(org_id, document_type_name)` across all publications.

### 9A.5 Mindmap Restrictions for Publications

* Mindmaps can **only** be generated from **atomic** (single-date) document types.
* Aggregated documents (Compilations, Magazines, Quarterly Collections, etc.) are **never** sent to the Mindmap pipeline.
* This applies to both default and custom publications.
* The Editor selects a specific single-date document from any publication and initiates the Mindmap pipeline from that document.

### 9A.6 Publication Management Access Matrix

| Capability                                              | Super Admin | Editor                              | Teacher                                                     | Sub-member       |
| ------------------------------------------------------- | ----------- | ----------------------------------- | ----------------------------------------------------------- | ---------------- |
| View all publications (system-wide)                     | true        | false (X)                           | false (X)                                                   | false (X)        |
| Approve new publication requests                        | true        | false (X)                           | false (X)                                                   | false (X)        |
| Suspend/block any publication or specific document type | true        | false (X)                           | false (X)                                                   | false (X)        |
| Create new publication (direct)                         | false (X)   | false (X)                           | true (requires SA approval)                                 | false (X)        |
| Request new publication                                 | false (X)   | true (requires Teacher+SA approval) | N/A                                                         | false (X)        |
| Configure publication (doc types, triggers, names)      | false (X)   | true (managed access)               | true (full access)                                          | false (X)        |
| Add atomic document types                               | false (X)   | true                                | true                                                        | false (X)        |
| Add aggregated document types                           | false (X)   | true                                | true                                                        | false (X)        |
| View publication settings                               | true        | true                                | true                                                        | true (view-only) |
| Delete publication                                      | false (X)   | false (X)                           | true (requires SA approval, cannot delete last publication) | false (X)        |

### 9A.7 Duplicate Document Prevention

The application must enforce strict **duplicate document prevention** at both backend and UI levels.

#### Backend Logic

Before allowing document creation, the backend validates:

1. **Same publication + same document type + same date** → **REJECT**. A document of the same type for the same date under the same publication already exists.
2. **Same document type name across publications** → Already prevented by the naming uniqueness constraint (Section 9A.4).

#### Duplicate Prevention Flow

```
User initiates document creation:
  Step 1: CHOOSE PUBLICATION TYPE   → e.g. 'CURRENT-AFFAIRS'
  Step 2: CHOOSE DOCUMENT TYPE      → e.g. 'NEWSLETTER'
  Step 3: CHOOSE DATE               → e.g. '01-01-2026'
  
Backend check: SELECT COUNT(*) FROM documents 
  WHERE org_id = ? AND publication_id = ? 
  AND doc_type_name = ? AND document_date = ?

  IF count > 0:
    → [DUPLICATE DENIED] 
    → UI warning: "A [NEWSLETTER] for [01-01-2026] already exists under 
       [CURRENT-AFFAIRS]. You cannot create duplicate documents. 
       To make changes, open the existing document and create a new version."
    → CTA button: "Open Existing Document"
    
  IF count = 0:
    → Proceed with document creation
```

#### UI Indicators

* When the user selects a date in the document creation flow, the system performs a real-time check and renders availability:
  * **Green indicator**: "No existing document — ready to create"
  * **Red indicator**: "Document already exists — \[View Existing]"
* The calendar date picker should visually mark dates that already have documents for the selected publication + document type (e.g. dot indicator on occupied dates).

#### Version Control Alternative

When a duplicate is detected, the user is directed to the existing document where they can:

* View the current version
* Create a new version (Section 24.4 — Named Version system applies)
* Documents support full version history with semantic versioning (`1.0.0`, `1.1.0`, etc.)

### 9A.8 Impact on Entry Point Cards

#### Teacher Dashboard Cards (Section 8.1.1)

When multiple publications exist, the Teacher's Entry Point Cards are **grouped by publication**:

* A publication selector/tab appears above the cards
* Selecting a publication shows the cards for that publication's document types
* The default view shows cards for `CURRENT-AFFAIRS` (default publication)
* Custom publications show cards matching their configured document types

#### Editor Dashboard Cards (Section 8.2.0)

The Editor's Entry Point Cards show **aggregate counts across all publications and all assigned orgs**:

* Each standard card (Newsletter, Compilation, etc.) aggregates counts across all publications and all assigned orgs
* A publication filter dropdown allows filtering to a specific publication
* The Mindmap card remains Editor-exclusive and aggregates across all publications

### 9A.9 Impact on Billing

* All document types across all publications are **billable** following the same rules as Section 23.
* The invoice groups documents **by publication**, then by document type within each publication.
* The `is_billable` flag applies per-document regardless of which publication it belongs to.
* The retrospective document logic (Section 23.3A) applies identically across all publications.

### 9A.10 Impact on Google Drive Folder Structure

Publications extend the Google Drive folder structure (Section 25.2):

```
Teacher's Drive Root/
  └── Notesglider/
      └── [Organisation Name]/
          └── [Academic Year]/
              ├── CURRENT-AFFAIRS/
              │   ├── Newsletters/
              │   ├── Compilations/
              │   ├── Magazines/
              │   ├── Quarterly/
              │   ├── BiAnnual/
              │   ├── AnnualYearbook/
              │   └── Mindmaps/
              ├── SCIENCE-OUTLOOK/
              │   ├── Daily-Science-Brief/
              │   ├── Weekly-Science-Compilation/
              │   ├── Monthly-Science-Magazine/
              │   └── Mindmaps/
              └── [Other Publications]/
                  └── ...
```
