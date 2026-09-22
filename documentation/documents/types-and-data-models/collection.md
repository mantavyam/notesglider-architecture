# Collection

## SECTION 15A — QUARTERLY COLLECTION WORKFLOW

### 15A.1 What a Quarterly Collection Is

A Quarterly Collection is a single aggregated document covering one academic year quarter (3 months) of data. It aggregates all Compilation and Magazine documents within the quarter's date range.

### 15A.2 Trigger Mechanism

#### Automatic (Cron-Based)

* Google Cloud Scheduler triggers a backend job at the configured time (default: 6:00 PM IST, GMT +5:30). Trigger date is configurable by the Editor; **default is 1 week after the quarter ends**.
* For a January–December academic year:
  * Q1 (Jan–Mar): triggers on April 7
  * Q2 (Apr–Jun): triggers on July 7
  * Q3 (Jul–Sep): triggers on October 7
  * Q4 (Oct–Dec): triggers on January 7 of the next year
* Each Teacher's academic year configuration is evaluated independently.
* Supports multilingual aggregation in parallel.

#### Manual (Editor On-Demand Override)

* Same on-demand creation capability as Compilation (see Section 14.2).
* Editor selects specific documents or defines a custom 3-month date range.

### 15A.3 Quarterly Collection Creation Pipeline

1. **Aggregation**: Backend collects all approved Newsletters, Compilations, and Magazines within the quarter's date range.
2. **Structure Assembly**: Backend builds the Quarterly Collection document structure from aggregated data.
3. **Image Folder**: A single image folder with sub-folders per news category: `IMG-Q{n}-AY{YYYY}/CATEGORY1/`, etc.
4. **Editor Review**: Quarterly Collection document enters Editor's queue.
5. **PDF Generation**: WeasyPrint generates Quarterly Collection PDF using themed template.
6. **Proofreading Check**: Editor completes verification.
7. **Drive Upload**: Final PDF uploaded to: `YYYY/QUARTERLY/FINAL/Q{n}-COLLECTION-AY{YYYY}.pdf`.

### 15A.4 Quarterly Collection File Naming Convention

| Asset            | Naming Pattern                                                 |
| ---------------- | -------------------------------------------------------------- |
| RAW document     | `COLLECTION-Q{n}-AY{YYYY}.extension`                           |
| RAW HTML         | `COLLECTION-Q{n}-AY{YYYY}.html`                                |
| RAW image folder | `IMG-Q{n}-AY{YYYY}/CATEGORY1/`, `IMG-Q{n}-AY{YYYY}/CATEGORYn/` |
| Final PDF        | `Q{n}-COLLECTION-AY{YYYY}.pdf`                                 |
