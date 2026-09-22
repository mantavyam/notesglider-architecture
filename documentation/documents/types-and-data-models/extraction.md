# Extraction

## SECTION 15D — CATEGORY EXTRACTION WORKFLOW

### 15D.1 What a Category Extraction Is

A Category Extraction is a standalone document that isolates a specific cherry-picked set of news categories from individual newsletters over any aggregation range (Week, Month, Quarter, Bi-Annual, Annual). It produces a focused, category-specific output from the larger aggregated data.

### 15D.2 Creation Flow (Editor-Only)

1. **Select Aggregation Type**: Editor selects the source aggregation type (Weekly Compilation, Monthly Magazine, Quarterly Collection, Bi-Annual Compendium, or Annual Yearbook) and the specific period.
2. **Category Selection UI**: A checkbox-based interface displays all news categories that exist across the newsletters in the selected aggregation range.
3. **Cherry-Pick Categories**: Editor selects one or more categories. All news items falling under the selected categories across the aggregation range are included.
4. **Document Generation**: Backend assembles the extraction document containing only the selected categories.
5. **Billing Flag**: Editor sets the `count_in_billing` flag (default: `false`). When `false`, this document is not counted towards the Teacher's billable page count.
6. **Editor Review & PDF Generation**: Standard pipeline — Editor reviews, approves, WeasyPrint generates PDF.
7. **Storage**: The extraction document is saved as its own entity in the same folder structure used by the source aggregation type.

### 15D.3 Category Extraction File Naming Convention

| Asset     | Naming Pattern                                                |
| --------- | ------------------------------------------------------------- |
| Document  | `extraction-{CATEGORY-NAME}-{AGGREGATION-DOC-NAME}.extension` |
| HTML      | `extraction-{CATEGORY-NAME}-{AGGREGATION-DOC-NAME}.html`      |
| Final PDF | `extraction-{CATEGORY-NAME}-{AGGREGATION-DOC-NAME}.pdf`       |

**Example**: `extraction-SCIENCE-AND-TECHNOLOGY-WK1-COMPILATION-MAR26.pdf`
