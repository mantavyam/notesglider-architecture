# Compendium

## SECTION 15B — BI-ANNUAL COMPENDIUM WORKFLOW

### 15B.1 What a Bi-Annual Compendium Is

A Bi-Annual Compendium is a single aggregated document covering 6 months of an academic year. Each academic year produces exactly two Compendiums: Compendium-1 (first half) and Compendium-2 (second half).

### 15B.2 Trigger Mechanism

#### Automatic (Cron-Based)

* Google Cloud Scheduler triggers a backend job at the configured time (default: 6:10 PM IST, GMT +5:30). Trigger date is configurable; **default is 1 week after the bi-annual period ends**.
* For a January–December academic year:
  * Compendium-1 (Jan–Jun): triggers on July 7
  * Compendium-2 (Jul–Dec): triggers on January 7 of the next year
* Supports multilingual aggregation in parallel.

#### Manual (Editor On-Demand Override)

* Same on-demand creation capability as Compilation.
* Editor defines a custom 6-month date range.

### 15B.3 Bi-Annual Compendium Creation Pipeline

1. **Aggregation**: Backend collects all approved documents within the 6-month range (Newsletters, Compilations, Magazines, Quarterly Collections).
2. **Structure Assembly**: Backend builds the Compendium document structure.
3. **Image Folder**: `IMG-C{n}-AY{YYYY}/CATEGORY1/`, etc.
4. **Editor Review**: Document enters Editor's queue.
5. **PDF Generation**: WeasyPrint generates Compendium PDF using themed template.
6. **Drive Upload**: Final PDF uploaded to: `YYYY/BIANNUAL/FINAL/COMPENDIUM-{n}-AY{YYYY}.pdf`.

### 15B.4 Bi-Annual Compendium File Naming Convention

| Asset            | Naming Pattern                                                 |
| ---------------- | -------------------------------------------------------------- |
| RAW document     | `COMPENDIUM-{n}-AY{YYYY}.extension`                            |
| RAW HTML         | `COMPENDIUM-{n}-AY{YYYY}.html`                                 |
| RAW image folder | `IMG-C{n}-AY{YYYY}/CATEGORY1/`, `IMG-C{n}-AY{YYYY}/CATEGORYn/` |
| Final PDF        | `COMPENDIUM-{n}-AY{YYYY}.pdf`                                  |
