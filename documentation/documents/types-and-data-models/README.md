---
icon: folder-tree
---

# Types & Data Models

{% hint style="info" %}
A Good Example for creating themed templates for dynamically generating the PDF from the structured source input is provided in this Github repository from the official weasyprint organisation, please use it for reference.

**Some Notable Mentions Include:**

[https://github.com/CourtBouillon/weasyprint-samples/tree/main/report](https://github.com/CourtBouillon/weasyprint-samples/tree/main/report)

[https://github.com/CourtBouillon/weasyprint-samples/tree/main/book](https://github.com/CourtBouillon/weasyprint-samples/tree/main/book)

[https://github.com/CourtBouillon/weasyprint-samples/tree/main/invoice](https://github.com/CourtBouillon/weasyprint-samples/tree/main/invoice)
{% endhint %}

{% embed url="https://github.com/CourtBouillon/weasyprint-samples/tree/main" %}

## SECTION 9 — DOCUMENT TYPES & DATA MODELS

### 9.1 Document Type Overview

| Frequency | Scope          | Document Name            | Creator                                                                            |
| --------- | -------------- | ------------------------ | ---------------------------------------------------------------------------------- |
| Daily     | Single date    | **Newsletter**           | Teacher (canvas editor)                                                            |
| Daily     | Single date    | **Mindmap**              | Editor (from Newsletter assets)                                                    |
| Weekly    | 7-day range    | **Compilation**          | System (auto) or Editor (manual)                                                   |
| Monthly   | Calendar month | **Magazine**             | System (auto) or Editor (manual)                                                   |
| Quarterly | 3-month range  | **Quarterly Collection** | System (auto) or Editor (manual)                                                   |
| Bi-Annual | 6-month range  | **Bi-Annual Compendium** | System (auto) or Editor (manual)                                                   |
| Annual    | 12-month range | **Annual Yearbook**      | System (auto) or Editor (manual)                                                   |
| On-demand | Variable range | **Category Extraction**  | Editor (manual cherry-pick of specific news categories from any aggregation range) |

### 9.2 Compilation Trigger Schedule

Compilation documents aggregate Newsletter data for the following weekly date ranges within each calendar month. **Trigger time**: configurable by the Editor in their settings for the respective Organisation. **Default trigger time**: 6:00 PM IST (GMT +5:30). The trigger runs at the end of the day to include documents produced on the trigger date itself.

| Trigger Date       | Date Range Covered  | Default Trigger Time |
| ------------------ | ------------------- | -------------------- |
| 7th of month       | 1st — 7th           | 6:00 PM IST          |
| 14th of month      | 8th — 14th          | 6:00 PM IST          |
| 21st of month      | 15th — 21st         | 6:00 PM IST          |
| Last date of month | 22nd — End of month | 6:00 PM IST          |

### 9.3 Magazine Trigger Schedule

**Trigger time**: configurable by the Editor. **Default**: 6:00 PM IST (GMT +5:30).

| Trigger Date       | Data Range Covered                                      | Default Trigger Time |
| ------------------ | ------------------------------------------------------- | -------------------- |
| Last date of month | Full calendar month (all 4 weekly Compilation archives) | 6:00 PM IST          |

### 9.3A Quarterly Collection Trigger Schedule

Quarterly Collections aggregate 3 months of data per academic year quarter. Trigger is configurable in Editor settings. **Default**: 1 week after the quarter ends.

| Quarter | Calendar Months (for Jan–Dec Academic Year) | Default Trigger Date                 | Default Trigger Time |
| ------- | ------------------------------------------- | ------------------------------------ | -------------------- |
| Q1      | Months 1–3 of Academic Year                 | 7th of 4th Calendar Month of AY      | 6:00 PM IST          |
| Q2      | Months 4–6 of Academic Year                 | 7th of 7th Calendar Month of AY      | 6:00 PM IST          |
| Q3      | Months 7–9 of Academic Year                 | 7th of 10th Calendar Month of AY     | 6:10 PM IST          |
| Q4      | Months 10–12 of Academic Year               | 7th of 1st Calendar Month of next AY | 6:00 PM IST          |

> **Note**: For a January–December academic year, Q4 trigger fires on January 7 of the next calendar year.

### 9.3B Bi-Annual Compendium Trigger Schedule

Bi-Annual Compendiums aggregate 6 months of data. Trigger is configurable. **Default**: 1 week after the bi-annual period ends.

| Compendium   | Calendar Months (for Jan–Dec Academic Year) | Default Trigger Date                 | Default Trigger Time |
| ------------ | ------------------------------------------- | ------------------------------------ | -------------------- |
| Compendium-1 | Months 1–6 of Academic Year                 | 7th of 7th Calendar Month of AY      | 6:10 PM IST          |
| Compendium-2 | Months 7–12 of Academic Year                | 7th of 1st Calendar Month of next AY | 6:10 PM IST          |

> **Note**: For a January–December academic year, Compendium-2 trigger fires on January 7 of the next calendar year.

### 9.3C Annual Yearbook Trigger Schedule

The Annual Yearbook aggregates the complete 12 months of an academic year. Trigger is configurable. **Default**: 1 week after the annual period ends.

| Document        | Data Range                  | Default Trigger Date                 | Default Trigger Time |
| --------------- | --------------------------- | ------------------------------------ | -------------------- |
| Annual Yearbook | Full 12-month Academic Year | 7th of 1st Calendar Month of next AY | 6:20 PM IST          |

> **Intentional co-scheduling**: For a January–December academic year, Q4 Collection (6:00 PM), Compendium-2 (6:10 PM), and Annual Yearbook (6:20 PM) all trigger on January 7 of the next year, processing independently as separate parallel documents queued one after another with 10-minute intervals between each.

### 9.3D Category Extraction

Category Extraction is a standalone document type that cherry-picks specific news categories from individual newsletters over any aggregation range (Week, Month, Quarter, Bi-Annual, Annual).

* **Creator**: Editor (manual)
* **UI mechanism**: Checkbox interface allowing selection of specific news categories (e.g. "SCIENCE & TECHNOLOGY") across the aggregation range
* **Document naming**: `extraction-{CATEGORY-NAME}-{AGGREGATION-DOC-NAME}`
* **Storage**: Uses the same folder structure as the aggregation type from which it was extracted
* **Kanban visibility**: Appears in Kanban board as its own entity
* **Billing flag**: Configurable boolean flag `count_in_billing` (default: `false`). Generally set to `false` because these documents are primarily used for proofreading and cross-checking purposes of an individual category within the overall final aggregated document.
* **Drive storage**: Saved as its own entity in the path used by the source aggregation type
* **Use case example**: An exclusive "SCIENCE & TECHNOLOGY" category extracted over a monthly aggregation range could be used for cross-checking purposes to verify the integrity of the actual final Magazine document
* **Multi-lingual support**: Category Extractions support parallel multilingual generation as configured by Teacher or Editor

### 9.4 Newsletter Raw Data Structure (Markdown Hierarchy)

The Newsletter document follows this strict heading hierarchy when represented as markdown. This hierarchy governs both the canvas editor display and the PDF template rendering:

```
# DD-MM-YY                    ← H1: Document Date
# CURRENT AFFAIRS             ← H1: Section Header
| Q  | A  |                   ← Table: 2 columns, minimum 6 rows (Q&A pairs)
|----|----|
| Q1 | A1 |

# OUTLINE                     ← H1: Section Header
- Headline 1                  ← Bullet list of all news headlines in this Newsletter
- Headline 2
───────────────────           ← Visual separator

# NEWS CATEGORY               ← H1: Category name (e.g. SCIENCE & TECHNOLOGY)
## News Headline              ← H2: Individual news item headline
### DD-MM-YY                  ← H3: Date of the news item
Content paragraphs...         ← Body text, bullet points, tables (optional)
```

### 9.5 Compilation & Magazine Raw Data Structure

Compilation documents are aggregated from multiple Newsletter archives. The structure is as follows:

```
# WK{n}/M{n}-Compilation-DD(start)-DD(end)-MMM-YY-RAW
│
├── # INDEX
│   ├── CATEGORY-1 (from all Newsletters in the period)
│   ├── CATEGORY-2
│   └── CATEGORY-N
│
├── # HIGHLIGHTS OF THIS WEEK/MONTH
│   └── (2 randomly selected news headlines from each Newsletter in range)
│
├── # WEEKLY/MONTHLY CURRENT AFFAIRS
│   ├── ## DATE-1
│   │   └── Q&A Table (from that date's Newsletter)
│   ├── ## DATE-2
│   │   └── Q&A Table
│   └── ## DATE-N
│       └── Q&A Table
│
├── # OUTLINE
│   ├── ## DATE-1
│   │   └── Bullet list of all headlines from that date
│   ├── ## DATE-2
│   │   └── Bullet list of headlines
│   └── ## DATE-N
│       └── Bullet list of headlines
│
└── # WEEKLY/MONTHLY COMPILATION
    ├── # CATEGORY-1
    │   ├── ## DATE-1
    │   │   ├── ### NEWS-HEADLINE
    │   │   │   ├── Bullet points / paragraphs (news content)
    │   │   │   └── Optional related data table
    │   │   └── ### NEWS-HEADLINE
    │   │       └── ...
    │   ├── ## DATE-2
    │   │   └── ...
    │   └── ## DATE-N
    │       └── ...
    ├── # CATEGORY-2
    │   └── ...
    └── # CATEGORY-N
        └── ...
```

### 9.6 Document Status State Machine

Every document in the system has a `doc-status` field that follows this state machine:

```
draft ──► review ──► approved ──► published ──► archived
  ▲                      │
  │                      ▼
  └──── (revision) ◄── delivered
```

| Status      | Description                                     | Who Sets It                    |
| ----------- | ----------------------------------------------- | ------------------------------ |
| `draft`     | Teacher is actively creating/editing            | System (on creation)           |
| `review`    | Submitted to Editor queue, locked               | System (on Teacher submission) |
| `approved`  | Editor has reviewed and approved raw document   | Editor                         |
| `published` | PDF has been generated and delivered to Teacher | System (on delivery)           |
| `archived`  | Document moved to historical archive            | System or Editor               |
