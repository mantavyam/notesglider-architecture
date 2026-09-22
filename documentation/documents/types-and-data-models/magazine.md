# Magazine

## SECTION 15 — MAGAZINE DOCUMENT WORKFLOW

### 15.1 What a Magazine Is

A Magazine is a single aggregated document covering a full calendar month of data. It is assembled from all four weekly Compilation archives for that month.

### 15.2 Trigger Mechanism

#### Automatic (Cron-Based)

* Google Cloud Scheduler triggers a backend job on the **last day of each calendar month** at the configured trigger time (default: 6:00 PM IST, GMT +5:30). The trigger time is configurable by the Editor.
* Aggregates all four Compilation documents for that month.
* Supports multilingual aggregation in parallel as configured by the Teacher or triggered by the Editor.

#### Manual (Editor On-Demand Override)

* Same on-demand creation capability as Compilation (see Section 14.2).
* Editor selects specific Compilation documents or defines a custom date range.

### 15.3 Magazine Creation Pipeline

1. **Aggregation**: Backend collects all Compilation documents for the month.
2. **Structure Assembly**: Backend builds the Magazine document structure from Compilation data.
3. **Image Folder**: A single image folder with sub-folders per news category: `IMG-MMMYY/CATEGORY1/`, etc.
4. **Editor Review**: Magazine document enters Editor's queue.
5. **PDF Generation**: WeasyPrint generates Magazine PDF using themed magazine template.
6. **Proofreading Check**: Editor completes verification.
7. **Drive Upload**: Final PDF uploaded to: `YYYY/MMMYY/MONTHLY/FINAL/MAGAZINE-MMMYY.pdf`.

### 15.4 Magazine File Naming Convention

| Asset            | Naming Pattern                                 |
| ---------------- | ---------------------------------------------- |
| RAW document     | `MAGAZINE-MMMYY.extension`                     |
| RAW image folder | `IMG-MMMYY/CATEGORY1/`, `IMG-MMMYY/CATEGORYn/` |
| Image files      | `n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension`  |
| Final PDF        | `MAGAZINE-MMMYY.pdf`                           |
