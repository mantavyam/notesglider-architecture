# Compilation

## SECTION 14 — COMPILATION DOCUMENT WORKFLOW

### 14.1 What a Compilation Is

A Compilation is a single aggregated document covering one week's worth of Newsletter data. It includes:

* An index of all news categories covered during the week.
* Highlighted headlines randomly selected from each day's Newsletter.
* The Current Affairs Q\&A tables from each date in the range.
* The full outline (all news headlines) from each date.
* The complete news content organised by category, then by date within each category.

### 14.2 Trigger Mechanism

#### Automatic (Cron-Based)

* Google Cloud Scheduler triggers a backend job on the following dates each month at the configured trigger time (default: 6:00 PM IST, GMT +5:30). The trigger time is configurable by the Editor in their settings per Organisation.
  * **7th** → aggregates Newsletters from 1st–7th (inclusive of documents produced on the 7th before trigger time)
  * **14th** → aggregates Newsletters from 8th–14th
  * **21st** → aggregates Newsletters from 15th–21st
  * **Last day of month** → aggregates Newsletters from 22nd–end of month
* All Compilation documents support multilingual aggregation in parallel as configured by the Teacher's requirements or as manually triggered by the Editor during the processing queue.

#### Manual (Editor On-Demand Override)

* Editor can initiate a Compilation creation at any time from their RBAC dashboard.
* Manual creation UI provides two selection modes:
  * **Cherry-pick mode**: Editor selects specific Newsletter documents individually.
  * **Date range mode**: Editor defines a custom start and end date — all Newsletters in that range are included.
* Manual creation overrides the standard weekly date boundaries.
* Manual Compilations follow the same pipeline and output format as auto-triggered ones.

### 14.3 Newsletter Eligibility Rules

| Scenario        | Behaviour                                                                                                                                                                                                                        |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Default filter  | Only Newsletters with `doc-status = approved` are included automatically.                                                                                                                                                        |
| Editor override | Editor can include Newsletters of any status. A dismissible warning dialog appears: _"The following documents have not been approved. Including them may affect output quality."_ with the list of non-approved document titles. |

### 14.4 Compilation Creation Pipeline

1. **Aggregation**: Backend collects all eligible Newsletters within the date range.
2. **Structure Assembly**: Backend builds the Compilation document structure (index, highlights, current affairs tables, outlines, full content).
3. **Highlight Selection**: System randomly selects 2 news headlines from each date's Newsletter for the "Highlights" section.
4. **Document Creation**: A single Compilation document is created in the database.
5. **Image Folder**: A single image folder with sub-folders per news category is created: `WKn-MMMYY/CATEGORY1/`, `WKn-MMMYY/CATEGORYn/`, etc.
6. **Editor Review**: Compilation document enters Editor's queue (same RBAC pipeline as Newsletter).
7. **PDF Generation**: Editor reviews and approves → WeasyPrint generates the Compilation PDF using the themed compilation template.
8. **Proofreading Check**: Editor completes proofreading verification.
9. **Drive Upload**: Editor completes process by uploading final Compilation PDF to: `YYYY/MMMYY/WEEKLY/FINAL/WKn-COMPILATION-MMMYY.pdf`.

### 14.5 Compilation File Naming Convention

| Asset            | Naming Pattern                                         |
| ---------------- | ------------------------------------------------------ |
| RAW document     | `COMPILATION-WK1-MMMYY.extension`                      |
| RAW image folder | `IMG-WK1-MMMYY/CATEGORY1/`, `IMG-WK1-MMMYY/CATEGORYn/` |
| Image files      | `n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension`          |
| Final PDF        | `WK1-COMPILATION-MMMYY.pdf`                            |

Replace `WK1` with `WK2`, `WK3`, `WK4` per the week serial number.
