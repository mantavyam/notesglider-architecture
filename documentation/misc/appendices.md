---
icon: asterisk
---

# Appendices

## APPENDIX A — IMAGE NAMING CONVENTION REFERENCE

```
n-HEADLINE[:10]-CATEGORY-DD-MM-YY.extension
```

| Token           | Meaning                                                                                | Example         |
| --------------- | -------------------------------------------------------------------------------------- | --------------- |
| `n`             | Order index of image (1, 2, 3...)                                                      | `1`             |
| `HEADLINE[:10]` | First 10 characters of headline (strip special chars, replace spaces with underscores) | `IndialaunchAI` |
| `CATEGORY`      | Category name (uppercase, hyphens for spaces)                                          | `SCIENCE-TECH`  |
| `DD-MM-YY`      | Document date                                                                          | `01-03-26`      |
| `.extension`    | File extension                                                                         | `.jpg`          |

**Full example**: `1-IndiaLaunc-SCIENCE-TECH-01-03-26.jpg`

***

## APPENDIX B — DOCUMENT ID FORMATS

| Document Type        | Format                                              | Example                                                   |
| -------------------- | --------------------------------------------------- | --------------------------------------------------------- |
| Newsletter           | `NL-YYYYMMDD-{4-digit seq}`                         | `NL-20260301-0001`                                        |
| Compilation          | `CM-WKn-MMMYY-{4-digit seq}`                        | `CM-WK1-MAR26-0001`                                       |
| Magazine             | `MG-MMMYY-{4-digit seq}`                            | `MG-MAR26-0001`                                           |
| Mindmap              | `MM-YYYYMMDD-{4-digit seq}`                         | `MM-20260301-0001`                                        |
| Quarterly Collection | `QC-Qn-YYYY-{4-digit seq}`                          | `QC-Q1-2026-0001`                                         |
| Bi-Annual Compendium | `BA-Hn-YYYY-{4-digit seq}`                          | `BA-H1-2026-0001`                                         |
| Annual Yearbook      | `YB-YYYY-{4-digit seq}`                             | `YB-2026-0001`                                            |
| Category Extraction  | `extraction-{CATEGORY-NAME}-{AGGREGATION-DOC-NAME}` | `extraction-SCIENCE_AND_TECHNOLOGY-COMPILATION-WK1-MAR26` |

***

## APPENDIX C — FILE NAMING CONVENTIONS SUMMARY

| File                            | Pattern                                             | Example                                                        |
| ------------------------------- | --------------------------------------------------- | -------------------------------------------------------------- |
| Daily Newsletter RAW (MD)       | `NEWSLETTER-DD-MM-YY.md`                            | `NEWSLETTER-01-03-26.md`                                       |
| Daily Newsletter RAW (HTML)     | `NEWSLETTER-DD-MM-YY.html`                          | `NEWSLETTER-01-03-26.html`                                     |
| Daily Newsletter PDF            | `NEWSLETTER-DD-MM-YY.pdf`                           | `NEWSLETTER-01-03-26.pdf`                                      |
| Daily Mindmap PDF               | `MINDMAP-DD-MM-YY.pdf`                              | `MINDMAP-01-03-26.pdf`                                         |
| Weekly Compilation RAW (MD)     | `COMPILATION-WK1-MMMYY.md`                          | `COMPILATION-WK1-MAR26.md`                                     |
| Weekly Compilation RAW (HTML)   | `COMPILATION-WK1-MMMYY.html`                        | `COMPILATION-WK1-MAR26.html`                                   |
| Weekly Compilation PDF          | `WK1-COMPILATION-MMMYY.pdf`                         | `WK1-COMPILATION-MAR26.pdf`                                    |
| Monthly Magazine RAW (MD)       | `MAGAZINE-MMMYY.md`                                 | `MAGAZINE-MAR26.md`                                            |
| Monthly Magazine RAW (HTML)     | `MAGAZINE-MMMYY.html`                               | `MAGAZINE-MAR26.html`                                          |
| Monthly Magazine PDF            | `MAGAZINE-MMMYY.pdf`                                | `MAGAZINE-MAR26.pdf`                                           |
| Quarterly Collection RAW (MD)   | `COLLECTION-Qn-YYYY.md`                             | `COLLECTION-Q1-2026.md`                                        |
| Quarterly Collection RAW (HTML) | `COLLECTION-Qn-YYYY.html`                           | `COLLECTION-Q1-2026.html`                                      |
| Quarterly Collection PDF        | `COLLECTION-Qn-YYYY.pdf`                            | `COLLECTION-Q1-2026.pdf`                                       |
| Bi-Annual Compendium RAW (MD)   | `COMPENDIUM-Hn-YYYY.md`                             | `COMPENDIUM-H1-2026.md`                                        |
| Bi-Annual Compendium RAW (HTML) | `COMPENDIUM-Hn-YYYY.html`                           | `COMPENDIUM-H1-2026.html`                                      |
| Bi-Annual Compendium PDF        | `COMPENDIUM-Hn-YYYY.pdf`                            | `COMPENDIUM-H1-2026.pdf`                                       |
| Annual Yearbook RAW (MD)        | `YEARBOOK-YYYY.md`                                  | `YEARBOOK-2026.md`                                             |
| Annual Yearbook RAW (HTML)      | `YEARBOOK-YYYY.html`                                | `YEARBOOK-2026.html`                                           |
| Annual Yearbook PDF             | `YEARBOOK-YYYY.pdf`                                 | `YEARBOOK-2026.pdf`                                            |
| Category Extraction RAW (MD)    | `extraction-{CATEGORY}-{AGGREGATION-DOC-NAME}.md`   | `extraction-SCIENCE_AND_TECHNOLOGY-COMPILATION-WK1-MAR26.md`   |
| Category Extraction RAW (HTML)  | `extraction-{CATEGORY}-{AGGREGATION-DOC-NAME}.html` | `extraction-SCIENCE_AND_TECHNOLOGY-COMPILATION-WK1-MAR26.html` |
| Category Extraction PDF         | `extraction-{CATEGORY}-{AGGREGATION-DOC-NAME}.pdf`  | `extraction-SCIENCE_AND_TECHNOLOGY-COMPILATION-WK1-MAR26.pdf`  |
| Year-end image archive          | `IMAGES-YYYY.zip`                                   | `IMAGES-2026.zip`                                              |

***

## APPENDIX D — AGGREGATION TRIGGER SCHEDULE QUICK REFERENCE

| Document          | Trigger Date                | Default Time | Date Range Covered             |
| ----------------- | --------------------------- | ------------ | ------------------------------ |
| WK1 Compilation   | 7th of month                | 6:00 PM IST  | 1st – 7th                      |
| WK2 Compilation   | 14th of month               | 6:00 PM IST  | 8th – 14th                     |
| WK3 Compilation   | 21st of month               | 6:00 PM IST  | 15th – 21st                    |
| WK4 Compilation   | Last day of month           | 6:00 PM IST  | 22nd – last day                |
| Monthly Magazine  | Last day of month           | 6:00 PM IST  | Full calendar month            |
| Q1 Collection     | 7th of 4th month of AY      | 6:00 PM IST  | Months 1–3 of academic year    |
| Q2 Collection     | 7th of 7th month of AY      | 6:00 PM IST  | Months 4–6 of academic year    |
| Q3 Collection     | 7th of 10th month of AY     | 6:00 PM IST  | Months 7–9 of academic year    |
| Q4 Collection     | 7th of 1st month of next AY | 6:00 PM IST  | Months 10–12 of academic year  |
| Compendium-1 (H1) | 7th of 7th month of AY      | 6:10 PM IST  | Months 1–6 of academic year    |
| Compendium-2 (H2) | 7th of 1st month of next AY | 6:10 PM IST  | Months 7–12 of academic year   |
| Annual Yearbook   | 7th of 1st month of next AY | 6:20 PM IST  | Full academic year (12 months) |

**Notes**:

* AY = Academic Year (configurable per Teacher, default January–December)
* All trigger times are configurable by the Editor in their organisation settings
* When Q4, Compendium-2, and Yearbook co-trigger on the same day (Jan 7 for default AY): Q4 fires at 6:00 PM, Compendium-2 at 6:10 PM, Yearbook at 6:20 PM — each as a separate parallel document queued sequentially
* Trigger time is always after the end of the last day of the covered period to ensure inclusion of all documents created on that final day

***

## APPENDIX E — BILLING QUICK REFERENCE

<table><thead><tr><th width="230.31640625">Item</th><th>Value</th></tr></thead><tbody><tr><td>Monthly rate</td><td>$1.00 per PDF page</td></tr><tr><td>Semi-annual rate</td><td>$0.95 per PDF page (discount: $0.05/page — requires 6-month PREPAID commitment)</td></tr><tr><td>Annual rate</td><td>$0.90 per PDF page (discount: $0.10/page — requires 12-month PREPAID commitment)</td></tr><tr><td>Billable unit</td><td>1 physical WeasyPrint PDF page</td></tr><tr><td>Billing cycle</td><td>Calendar month (registration date – last day; then 1st – last day)</td></tr><tr><td>Invoice trigger</td><td>5th of next month at 6:00 PM IST (POSTPAID, configurable by Super Admin)</td></tr><tr><td>Grace period</td><td>7 days from invoice due date</td></tr><tr><td>Post-grace block</td><td>PDF pipeline submission disabled; document creation + PPTX remain active (POSTPAID). All functionality blocked (PREPAID).</td></tr><tr><td>PREPAID mode</td><td>Optional — Super Admin sets freeform amount per Teacher. All functionality gatekept until cleared.</td></tr><tr><td>Discount activation</td><td>Pay full 6-month or 12-month PREPAID upfront → discount applies to all POSTPAID invoices during commitment</td></tr><tr><td>Auto-renewal</td><td>Opt-in via Account Settings; requires stored payment details (AES-256 encrypted) + 2FA enabled</td></tr><tr><td>Refund policy</td><td>Super Admin discretion — no automatic refunds. Manual via Billing Dashboard.</td></tr><tr><td>Invoice approval</td><td>Super Admin must review and approve before invoice is sent to Teacher</td></tr><tr><td>Invoice PDF generation</td><td>WeasyPrint (same engine as document PDFs)</td></tr><tr><td>Payment gateway</td><td>Razorpay — Super Admin's master account</td></tr><tr><td>Editor billing access</td><td>Document metrics dashboard only — no pricing, invoicing, or payment data</td></tr><tr><td>Compliance threshold</td><td>Default 75% atomic docs submitted for PDF pipeline per publication stream (configurable per org)</td></tr><tr><td>Compliance grace</td><td>First 2 complete billing cycles — no enforcement, silent monitoring only</td></tr><tr><td>Minimum data retention</td><td>2 months (non-negotiable system floor)</td></tr></tbody></table>

***

{% hint style="success" %}
_End of Notesglider PRD — Final Developer Handoff Edition v4.0.0_ _This document is the single source of truth for all implementation decisions._ _All ambiguities have been resolved through direct product owner confirmation._ _No section of this document should be interpreted loosely or substituted without explicit product owner approval._
{% endhint %}
