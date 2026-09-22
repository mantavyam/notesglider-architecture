# GDP Forecasts

### ASCII Layout <a href="#ascii-layout" id="ascii-layout"></a>

```
┌──────────────────────────────────────────────────────────────────┐
│  CHAPTER BANNER: GDP FORECASTS                                   │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────┬──────────────────────────┬───────────────────────────┐
│  **6.6%**                │  **World Bank**          │                           │
│  *GDP Growth Forecast*   │  *International Body*    │  Full news description    │
│  *FY 2025–26*            │                          │  with inline HTML lists   │
│                          │                          │                           │
└──────────────────────────┴──────────────────────────┴───────────────────────────┘
┌──────────────────────────┬──────────────────────────┬───────────────────────────┐
│  **6th Largest**         │  **IMF**                 │  Full news description    │
│  *Economy Rank*          │  *April 2026 Outlook*    │  with inline HTML lists   │
│  *$4.15 Trillion*        │                          │                           │
└──────────────────────────┴──────────────────────────┴───────────────────────────┘

        Col 1                      Col 2                     Col 3
  (key numeric figure)       (forecasting entity)          (full content)
      ~22% width                  ~23% width                 ~55% width
```

**Col 1 pattern — flexible to data type:**

```
Forecast/Projection  →  **6.6%** / *GDP Growth Forecast* / *FY 2025–26*
Ranking news         →  **6th Largest** / *Economy Rank* / *$4.15 Trillion*
Index / Score        →  **102.4** / *Ease of Doing Business* / *2026*
Deficit / Surplus    →  **-3.2%** / *Fiscal Deficit* / *FY 2026*
```

***

### Structuring Prompt <a href="#structuring-prompt" id="structuring-prompt"></a>

<pre><code>## SYSTEM:
You are a precise document transformation assistant for a monthly current affairs magazine. You restructure GDP forecasts and economic outlook news articles into a strict markdown table format for PDF generation via WeasyPrint. You must never add, invent, or remove any factual content — only restructure and reformat.

---

<strong>## USER:
</strong>
Transform the following GDP FORECASTS chapter markdown document into the structured format described below.

---

### TARGET FORMAT

The output must be a single markdown document with one table:

# GDP FORECASTS

| Figure | Forecasting Entity | Description |
|--------|--------------------|-------------|
| **Key Numeric Figure**&#x3C;br>*Parameter Label*&#x3C;br>*Period / Year (if available)* | **Organization / Agency Name**&#x3C;br>*Report / Publication Name (if mentioned)*&#x3C;br>*Category (e.g., International Body, Rating Agency, Think Tank)* | **Full Headline**&#x3C;br>&#x3C;br>[full news content] |

---

### TRANSFORMATION RULES

One row = one news headline (one forecast / outlook / ranking / economic data release).

**Column 1 — Key Numeric Figure:**
- Line 1: **The single most prominent numeric figure from the news** (bold) — this is the headline number that best represents the news item. Examples: **6.6%**, **6th Largest**, **$4.15 Trillion**, **102.4**, **-3.2%**.
- Line 2: *Parameter Label* (italic) — a concise label describing what the figure represents. Examples: *GDP Growth Forecast*, *Economy Rank*, *Fiscal Deficit*, *Inflation Rate*, *Growth Projection*.
- Line 3: *Period / Financial Year / Edition* (italic) — e.g., *FY 2025–26*, *Q3 2026*, *April 2026*. If not mentioned, use *—*.
- Separate lines with &#x3C;br>.
- If no numeric figure is clearly identifiable, use **—** on Line 1 and a descriptive label on Line 2.

**Column 2 — Forecasting Entity:**
- Line 1: **Full Organization / Agency / Entity Name** (bold) — e.g., **World Bank**, **International Monetary Fund**, **RBI**, **Moody's**, **ADB**.
- Line 2: *Report / Publication Name* (italic) — e.g., *World Economic Outlook*, *Global Economic Prospects*, *Monetary Policy Report*. If not mentioned, use *—*.
- Line 3: *Category* (italic) — classify the entity as one of: *International Body*, *Rating Agency*, *Central Bank*, *Think Tank*, *Government Body*, *Research Institution*. Infer from context.
- Separate lines with &#x3C;br>.

**Column 3 — Description:**
- Begin with the full news headline in bold: **Headline Text**.
- Follow with &#x3C;br>&#x3C;br> then the complete news content.
- Include ALL content from the original news item fully — do not summarize or truncate.
- Preserve all section sub-headings from the original (e.g., "Revised Growth Projections:", "Current Global Rankings:") as **Sub-heading:**&#x3C;br> inline in the description.
- Preserve all inline markdown: **bold**, *italic*.
- Convert unordered lists to: &#x3C;ul>&#x3C;li>item&#x3C;/li>&#x3C;li>item&#x3C;/li>&#x3C;/ul>
- Convert ordered lists to: &#x3C;ol>&#x3C;li>item&#x3C;/li>&#x3C;li>item&#x3C;/li>&#x3C;/ol>
- Nested lists should use nested &#x3C;ul>/&#x3C;ol> tags accordingly.
- Preserve any inline bold or italic inside list items within the HTML tags.
- Numeric data presented as bullet lists (e.g., country rankings with GDP values, year-wise projections) must be fully preserved as &#x3C;ul> or &#x3C;ol> — do not convert to prose.
- (if present) Fields like "Previous Forecast:", "Revised Estimate:", "Key Driver:", "Context:" must be preserved as **Field Name:** value inline in the description.
- Escape any pipe characters inside cells as \|.
- Use &#x3C;br> for paragraph breaks within the cell — never raw newlines.

**Date information:**
- Dates (## DD-MM-YY) from the original document must not appear anywhere in the output.
- They are used only to determine chronological row order (earliest first).
- No date headings, no date dividers, no date column anywhere in the output.

**Row ordering:**
- Rows must appear in the same chronological order as the input (earliest date first, within a date in original top-to-bottom order).

**Formatting hygiene:**
- Escape any pipe characters | inside table cells as \|.
- Use &#x3C;br> for all line breaks inside cells — never raw newlines.
- Do not add any commentary, preamble, or explanation outside the markdown.
- The output must begin with # GDP FORECASTS and then the table. Nothing else.

---

## INPUT DOCUMENT:

```markdown
[PASTE RAW MARKDOWN CHAPTER HERE] / Attached With Chat
```
</code></pre>

***

### How the Sample Transforms <a href="#how-the-sample-transforms" id="how-the-sample-transforms"></a>

```
# GDP FORECASTS

| Figure | Forecasting Entity | Description |
|--------|--------------------|-------------|
| **6.6%**<br>*GDP Growth Forecast*<br>*FY 2025–26* | **World Bank**<br>*—*<br>*International Body* | **World Bank Hikes India's GDP Growth Forecast to 6.6%**<br><br>The World Bank has increased India's growth projection for the current financial year due to strong local demand and new international trade deals. India is officially recognized as the main driver of economic growth across the South Asian region.<br><br>**Revised Growth Projections:**<br><ul><li>**Current FY (Forecast):** Raised from 6.3% to 6.6%.</li><li>**FY 2025 (Estimated):** Growth accelerated to 7.1%.</li><li>**FY 2026 (Estimated):** Projected to reach 7.6%.</li></ul> |
| **6th Largest**<br>*Economy Rank*<br>*April 2026* | **International Monetary Fund (IMF)**<br>*April 2026 World Economic Outlook*<br>*International Body* | **India Slips to 6th Largest Economy: IMF April 2026 Outlook**<br><br>According to the International Monetary Fund April 2026 data, India has become the 6th largest economy in the world, slipping from its previous position. While India's domestic growth remains robust, external financial factors and currency fluctuations have allowed the United Kingdom to move ahead.<br><br>**Current Global Rankings (IMF April 2026):**<br><ul><li>**United States:** $32.3 trillion</li><li>**China:** $20.85 trillion</li><li>**Germany:** $5.45 trillion</li><li>**Japan:** $4.38 trillion</li><li>**United Kingdom:** $4.26 trillion</li><li>**India:** $4.15 trillion</li></ul> |
```
