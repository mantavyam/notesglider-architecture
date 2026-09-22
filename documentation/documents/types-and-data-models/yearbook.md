# Yearbook

## SECTION 15C — ANNUAL YEARBOOK WORKFLOW

### 15C.1 What an Annual Yearbook Is

The Annual Yearbook is a single comprehensive document aggregating the complete 12 months of an academic year.

### 15C.2 Trigger Mechanism

#### Automatic (Cron-Based)

* Google Cloud Scheduler triggers a backend job at the configured time (default: 6:20 PM IST, GMT +5:30). Trigger date is configurable; **default is 1 week after the academic year ends**.
* For a January–December academic year: triggers on January 7 of the next year.
* **Co-scheduling note**: Q4 Collection (6:00 PM), Compendium-2 (6:10 PM), and Annual Yearbook (6:20 PM) all trigger on January 7, processing independently as separate parallel documents queued sequentially with 10-minute intervals.
* Supports multilingual aggregation in parallel.

#### Manual (Editor On-Demand Override)

* Editor defines a custom 12-month date range or selects the academic year.

### 15C.3 Annual Yearbook Creation Pipeline

1. **Aggregation**: Backend collects all approved documents across the complete academic year.
2. **Structure Assembly**: Backend builds the Yearbook document structure from the full year's data.
3. **Image Folder**: `IMG-YEARBOOK-AY{YYYY}/CATEGORY1/`, etc.
4. **Editor Review**: Document enters Editor's queue.
5. **PDF Generation**: WeasyPrint generates Yearbook PDF using themed template.
6. **Drive Upload**: Final PDF uploaded to: `YYYY/ANNUAL/FINAL/YEARBOOK-AY{YYYY}.pdf`.

### 15C.4 Annual Yearbook File Naming Convention

| Asset            | Naming Pattern                                                         |
| ---------------- | ---------------------------------------------------------------------- |
| RAW document     | `YEARBOOK-AY{YYYY}.extension`                                          |
| RAW HTML         | `YEARBOOK-AY{YYYY}.html`                                               |
| RAW image folder | `IMG-YEARBOOK-AY{YYYY}/CATEGORY1/`, `IMG-YEARBOOK-AY{YYYY}/CATEGORYn/` |
| Final PDF        | `YEARBOOK-AY{YYYY}.pdf`                                                |
