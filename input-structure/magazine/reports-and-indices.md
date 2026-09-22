# Reports & Indices

### ASCII Layout

```
+--------------------------------------------------------------------------+
|  Note: 25 mm First Page of the Chapter, Subsequent Page - 15mm           |
|--------------------------------------------------------------------------|
|                                                                          |
|                           [ MAIN PAGE TITLE ]                            |
|                                                                          |
|--------------------------------------------------------------------------|
| [IMAGE PLACEHOLDER:  :                                                   |
|   TILTED RECTANGLE   :     [ NEWS 1 HEADLINE - BOLD ].                   |
|  (3D PERSPECTIVE) ]  :     |— Content Placeholder Block                  |
|                      :     |— • Detail Content Line                      |
|                      :                                                   |
|--------------------------------------------------------------------------|
| +----------------------------------------------------------------------+ |
| |                      [ TABLE NAME HERE - BOLD TITLE ROW ]            | |
| |----------------------------------------------------------------------| |
| | [Column 1 Data] | [Column 2 Data] | [Column 3 Data]                  | |               
| | [Column 1 Data] | [Column 2 Data] | [Column 3 Data]                  | |               
| | [Column 1 Data] | [Column 2 Data] | [Column 3 Data]                  | |               
| | [Column 1 Data] | [Column 2 Data] | [Column 3 Data]                  | |               
| +----------------------------------------------------------------------+ |
|--------------------------------------------------------------------------|
| [IMAGE PLACEHOLDER:  :                                                   |
|   TILTED RECTANGLE   :     [ NEWS 2 HEADLINE - BOLD ].                   |
|  (3D PERSPECTIVE) ]  :     |— Content Placeholder Block                  |
|                      :     |— • Detail Content Line                      |
|                      :                                                   |
|--------------------------------------------------------------------------|
|                        Footer Height = 15mm                              |
+--------------------------------------------------------------------------+
```

### ASCII Layout (Secondary) <a href="#ascii-layout" id="ascii-layout"></a>

```
text┌──────────────────────────────────────────────────────────────────┐
│  CHAPTER BANNER: REPORTS & INDICES                               │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────┬──────────────────┬──────────────────┬───────────────────────────┐
│  **Global Gender Gap**   │  *2025*          │  **World**       │                           │
│  **Report**              │                  │  **Economic**    │  Full news description    │
│                          │                  │  **Forum (WEF)** │  with inline HTML lists   │
├──────────────────────────┼──────────────────┼──────────────────┼───────────────────────────┤
│  **Human Development**   │  *2024–25*       │  **UNDP**        │  Full news description    │
│  **Index**               │                  │                  │                           │
└──────────────────────────┴──────────────────┴──────────────────┴───────────────────────────┘
         ↓ (if accompanied by a table)
┌──────────────────────────────────────────────────────────────────┐
│  | Rank | Country | Score | ... |                               │
│  |------|---------|-------|     |                               │
│  | 1    | Iceland | 0.935 |    |                               │
└──────────────────────────────────────────────────────────────────┘

     Col 1               Col 2             Col 3              Col 4
 (report/index name)   (edition/year)   (org name)         (full content)
    ~25% width           ~12% width       ~18% width          ~45% width
```

***

### Structuring Prompt <a href="#structuring-prompt" id="structuring-prompt"></a>

````
## SYSTEM:
You are a precise document transformation assistant for a monthly current affairs
magazine. You restructure reports and indices news articles into a strict markdown
format for PDF generation via WeasyPrint. You must never add, invent, remove,
summarize, or paraphrase any factual content — only restructure and reformat.

---

## USER:

Transform the following REPORTS & INDICES chapter markdown document into the
structured format described below.

---

### TARGET FORMAT

The output is a sequence of news blocks. Each news block follows this exact pattern:

[4-column news table row]
[standalone accompanying table — ONLY if tabular data is present in that news item]

This pattern repeats for every news item. Each news block's table is independent —
close the table after each news row, place the accompanying table immediately after
if applicable, then open a fresh table with headers for the next news item.

Structure of each news block:

| Report / Index | Edition | Organisation | Description |
|----------------|---------|--------------|-------------|
| **Report or Index Name** | *Edition or Year* | **Organisation Name** | **Full Headline**<br><br>[full news content] |

Followed immediately by (only if tabular data is present):

| Col Header 1 | Col Header 2 | ... |
|--------------|--------------|-----|
| data         | data         | ... |

Then a new table begins for the next news item.

The full output must begin with:
# REPORTS & INDICES

---

### STEP 1 — EXTRACT & INFER COLUMN DATA (Do this before building the table)

For every news item (### headline), identify and infer the following:

**A. Report / Index Name:**
- The exact official name of the report, index, ranking, or survey being discussed.
- Examples: Global Gender Gap Report, Human Development Index,
  World Happiness Report, Ease of Doing Business Index, SIPRI Arms Report.
- Remove year or edition references from the name itself — those go in Column 2.
- If the news discusses a country's performance on a well-known index without
  naming it explicitly, infer the standard name from context.
- If two or more reports are discussed in a single headline, list both names
  separated by <br> within the cell.
- If no report/index name is identifiable, use **—**.

**B. Edition / Year:**
- Extract the edition number, year, or reporting period of the report/index.
- Format as:
  Named edition  → *2025 Edition* or *2024–25* or *Vol. 12 · 2025*
  Year only      → *2025*
  Quarterly      → *Q1 2026*
  If not mentioned → *—*
- Do not include the word "Report" or "Index" here — only the edition/year value.

**C. Organisation Name:**
- The body, institution, agency, or entity that published or released the report/index.
- Use the full official name. If an abbreviation is used in the source, expand it
  on first use with abbreviation in parentheses.
  Examples: World Economic Forum (WEF), United Nations Development Programme (UNDP),
  Stockholm International Peace Research Institute (SIPRI),
  International Monetary Fund (IMF), Reserve Bank of India (RBI).
- If multiple organisations jointly published, list all separated by <br>.
- If not mentioned but well-known from the report name, infer it.
- If genuinely unidentifiable, use *—*.

---

### STEP 2 — ASSESS FOR ACCOMPANYING TABLE

Before writing each news block, assess whether the news content contains accompanied structured
tabular data. If YES → the introductory/contextual paragraph(s) and key findings go in Column 4
of the news table row, and all structured ranking/data rows move to the standalone
accompanying table placed immediately after the news table row spanning full page width as a standalone table.

If NO → all content goes fully into Column 4. Do not create an accompanying table by yourself if not already present.

---

### COLUMN-BY-COLUMN RULES

**Column 1 — Report / Index Name:**
- **Report or Index Name** (bold) — from Step 1A.
- If the name is longer than ~5 words, break across two lines using <br> within
  the bold text for better visual fit.
- No additional lines needed in this column.

**Column 2 — Edition / Year:**
- *Edition or Year* (italic) — from Step 1B.
- Single line only.

**Column 3 — Organisation:**
- **Organisation Name** (bold) — from Step 1C.
- If multiple organisations, separate with <br>.
- If the abbreviation is well-known, you may use it as Line 1 bold and expand
  in italic on Line 2: **WEF**<br>*World Economic Forum*

**Column 4 — Description:**
- Begin with the full news headline in bold: **Full Headline Text**.
- Follow with <br><br> then the complete news content.
- If an accompanying table will follow (Step 2 = YES): include all narrative
  content — introductory sentences, context, key findings stated in prose,
  India's rank or score if mentioned, significance statements. Do NOT include
  the structured ranking rows or data tables in this column in that case.
- If no accompanying table (Step 2 = NO): include ALL content from the original
  news item fully — do not summarize, omit, or truncate anything.
- Preserve all inline markdown: **bold**, *italic*, __underline__ → <u>underline</u>.
- Convert unordered lists to: <ul><li>item</li><li>item</li></ul>
- Convert ordered lists to: <ol><li>item</li><li>item</li></ol>
- Nested lists should use nested <ul>/<ol> tags accordingly.
- Preserve any inline bold, italic, or underline inside list items within HTML tags.
- Fields like "India's Rank:", "Key Finding:", "Methodology:", "Base Year:",
  "Coverage:", "Score:" must be preserved as **Field Name:** value inline.
- Escape any pipe characters inside cells as \|.
- Use <br> for paragraph breaks within the cell — never raw newlines.

---

### ACCOMPANYING TABLE RULES (only when Step 2 = YES)

- Place the accompanying table immediately after the closing row of that news
  block's main table, before the next news block's table begins.
- Infer appropriate column headers from the content. Only include columns for
  data that is actually present in the source. Do not invent columns.
  Typical headers (adapt as needed):
  | Rank | Country / Entity | Score / Value | Change |
- Use standard markdown table format wherever the data is straightforward.
- If the original data contains merged cell logic (e.g., a category header
  spanning multiple sub-rows), use an HTML <table> with colspan / rowspan
  for that accompanying table only.
- Preserve ALL rows of data from the source — do not truncate ranking lists.
- Do not create an accompanying table for news with only prose content or a
  single data point.

---

**Date information:**
- Dates (## DD-MM-YY) from the original document must not appear anywhere
  in the output. They indicate only when the news was captured in the editor.
- Use them solely to determine chronological row order (earliest first).
- No date headings, no date dividers, no date column anywhere in the output.

**Row ordering:**
- Rows must appear in the same chronological order as the input
  (earliest ## DD-MM-YY date first, within a date in original top-to-bottom order).

**Formatting hygiene:**
- Escape any pipe characters | inside table cells as \|.
- Use <br> for all line breaks inside cells — never raw newlines.
- Do not add any commentary, preamble, or explanation outside the markdown.
- The output must begin with # REPORTS & INDICES followed immediately by the
  first news block. Nothing else before it.

---

## INPUT DOCUMENT:

```markdown
[PASTE RAW MARKDOWN CHAPTER HERE]
```
````

***

### How a Typical News Item Transforms <a href="#how-a-typical-news-item-transforms" id="how-a-typical-news-item-transforms"></a>

**Single data point (no accompanying table):**

```
# REPORTS & INDICES

| Report / Index | Edition | Organisation | Description |
|----------------|---------|--------------|-------------|
| **World Happiness Report** | *2025* | **UN Sustainable<br>Development Solutions Network** | **Finland Tops World Happiness Report 2025 for 8th Consecutive Year**<br><br>Finland has retained its position as the happiest country in the world for the eighth consecutive year according to the World Happiness Report 2025.<br><br><ul><li>**India's Rank:** 126 out of 147 countries.</li><li>**Methodology:** Based on self-reported life evaluations averaged over 2022–2024.</li><li>**Key Finding:** Social support and freedom to make life choices remain the top drivers of happiness globally.</li></ul> |
```

**With accompanying ranking table:**

```
| Report / Index | Edition | Organisation | Description |
|----------------|---------|--------------|-------------|
| **Global Gender Gap<br>Report** | *2025* | **WEF**<br>*World Economic Forum* | **India Ranks 129th in Global Gender Gap Report 2025**<br><br>The World Economic Forum released the Global Gender Gap Report 2025. India has been ranked 129th out of 156 countries, slipping 3 positions from its previous rank of 126.<br><br>**Key Finding:** India's lowest scores are in Economic Participation and Health sub-indices. |

| Rank | Country | Score |
|------|---------|-------|
| 1 | Iceland | 0.935 |
| 2 | Finland | 0.912 |
| 3 | Norway | 0.911 |
| 129 | India | 0.625 |

| Report / Index | Edition | Organisation | Description |
|----------------|---------|--------------|-------------|
| **[Next Report Name]** | *[Year]* | **[Org]** | **[Next Headline]**<br><br>[next content] |
```

### Report Image Frame

```
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Report Mockup</title>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800;900&display=swap" rel="stylesheet">
<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  /* ── PARAMETERISED VALUES ── */
  --PUBLISHER_NAME:      "THE SOUFAN CENTER";
  --REPORT_LABEL:        "GLOBAL SECURITY FORUM 2021";
  --REPORT_TITLE_LINE1:  "OVERVIEW";
  --REPORT_TITLE_LINE2:  "& FINDINGS";
  --REPORT_TITLE_LINE3:  "REPORT";
  --ORG_NAME_LARGE:      "GLOBAL SECURITY FORUM";

  /* ── THEME COLORS ── */
  --cover-bg:   #e8e8e8;
  --accent:     #1b2a4a;
  --dot-color:  rgba(27,42,74,0.13);
}

body {
  min-height: 100vh;
  background: #ffffff;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Inter', sans-serif;
  padding: 80px 40px;
}

/* ══════════════════════════════════════
   SCENE CONTAINER
══════════════════════════════════════ */
.scene {
  position: relative;
  width: 680px;
  height: 620px;
  filter: drop-shadow(0 40px 60px rgba(0,0,0,0.18)) drop-shadow(0 10px 20px rgba(0,0,0,0.10));
}

/* ══════════════════════════════════════
   SHARED REPORT COVER STYLES
══════════════════════════════════════ */
.report {
  position: absolute;
  width: 360px;
  height: 480px;
  background: var(--cover-bg);
  overflow: hidden;
  border-radius: 1px 4px 4px 1px;
  /* Thin binding crease on left */
  border-left: 3px solid rgba(0,0,0,0.12);
  box-shadow:
    inset 4px 0 8px rgba(0,0,0,0.06),
    inset -1px 0 3px rgba(0,0,0,0.04);
}

/* Dot-matrix background pattern (top-right quadrant) */
.report::before {
  content: '';
  position: absolute;
  top: 0; right: 0;
  width: 65%; height: 65%;
  background-image: radial-gradient(circle, var(--dot-color) 1.2px, transparent 1.2px);
  background-size: 10px 10px;
  /* Fade out toward center */
  -webkit-mask-image: radial-gradient(ellipse at top right, black 20%, transparent 75%);
  mask-image: radial-gradient(ellipse at top right, black 20%, transparent 75%);
  pointer-events: none;
}

/* Left binding shadow crease line */
.report::after {
  content: '';
  position: absolute;
  top: 0; left: 10px;
  width: 1px; height: 100%;
  background: linear-gradient(to bottom, rgba(0,0,0,0.08), rgba(0,0,0,0.14), rgba(0,0,0,0.06));
  pointer-events: none;
}

.report-inner {
  position: relative;
  z-index: 2;
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 22px 22px 16px 26px;
}

/* ── ORG LOGO BLOCK (top right) ── */
.org-logo-block {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  align-self: flex-end;
  margin-bottom: 10px;
}

.org-logo-icon {
  width: 32px;
  height: 32px;
  flex-shrink: 0;
}

/* Globe SVG icon */
.org-logo-text {
  display: flex;
  flex-direction: column;
  line-height: 1.1;
}
.org-logo-text span {
  font-size: 8.5px;
  font-weight: 800;
  letter-spacing: .12em;
  text-transform: uppercase;
  color: var(--accent);
}

/* ── REPORT LABEL (e.g. "GLOBAL SECURITY FORUM 2021") ── */
.report-label {
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: .15em;
  text-transform: uppercase;
  color: var(--accent);
  margin-bottom: 6px;
  margin-top: auto;
}

/* ── MAIN TITLE BLOCK ── */
.report-title {
  font-size: 58px;
  font-weight: 900;
  line-height: 0.92;
  letter-spacing: -.01em;
  text-transform: uppercase;
  color: var(--accent);
  margin-bottom: 28px;
}

/* ── BOTTOM INFO BLOCK ── */
.report-footer {
  margin-top: auto;
  display: flex;
  flex-direction: column;
  gap: 2px;
  border-top: 1px solid rgba(27,42,74,0.2);
  padding-top: 8px;
}

.footer-publisher {
  font-size: 7px;
  font-weight: 600;
  letter-spacing: .22em;
  text-transform: uppercase;
  color: rgba(27,42,74,0.5);
  margin-bottom: 3px;
}

.footer-org {
  font-size: 10.5px;
  font-weight: 800;
  letter-spacing: .08em;
  text-transform: uppercase;
  color: var(--accent);
}

.footer-subtitle {
  font-size: 9px;
  font-weight: 400;
  font-style: italic;
  color: var(--accent);
  line-height: 1.45;
}

.footer-meta {
  font-size: 7px;
  font-weight: 700;
  letter-spacing: .12em;
  text-transform: uppercase;
  color: rgba(27,42,74,0.6);
  margin-top: 4px;
}

.footer-url {
  font-size: 6.5px;
  letter-spacing: .10em;
  color: rgba(27,42,74,0.45);
  margin-top: auto;
  padding-top: 6px;
  text-align: center;
}

/* ══════════════════════════════════════
   REAR REPORT — tilted, behind, left
══════════════════════════════════════ */
.report-rear {
  transform-origin: center bottom;
  transform:
    translate(-30px, 40px)
    rotate(-8deg);
  z-index: 1;
  /* Slightly lighter/more washed to read as "behind" */
  filter: brightness(0.97);
}

/* ══════════════════════════════════════
   FRONT REPORT — slight tilt, right, in front
══════════════════════════════════════ */
.report-front {
  transform-origin: center bottom;
  transform:
    translate(0px, 20px)
    rotate(-1deg);
  z-index: 2;
}

/* ══════════════════════════════════════
   GROUND SHADOW
══════════════════════════════════════ */
.scene-shadow {
  position: absolute;
  bottom: -20px;
  left: 50%;
  transform: translateX(-100%);
  width: 500px;
  height: 70px;
  background: radial-gradient(ellipse at 50% 30%,
    rgba(0,0,0,0.22) 0%,
    rgba(0,0,0,0.08) 50%,
    transparent 80%);
  filter: blur(14px);
  pointer-events: none;
  z-index: 0;
}
</style>
</head>
<body>

<div class="scene">
  <div class="scene-shadow"></div>

  <!-- ══ REAR REPORT ══ -->
  <div class="report report-rear">
    <div class="report-inner">

      <div class="org-logo-block">
        <!-- Globe SVG icon -->
        <svg class="org-logo-icon" viewBox="0 0 40 40" fill="none" xmlns="http://www.w3.org/2000/svg">
          <circle cx="20" cy="20" r="18" stroke="#1b2a4a" stroke-width="1.5" fill="none"/>
          <ellipse cx="20" cy="20" rx="9" ry="18" stroke="#1b2a4a" stroke-width="1" fill="none"/>
          <ellipse cx="20" cy="20" rx="18" ry="7" stroke="#1b2a4a" stroke-width="1" fill="none"/>
          <line x1="2" y1="20" x2="38" y2="20" stroke="#1b2a4a" stroke-width="1"/>
          <line x1="20" y1="2" x2="20" y2="38" stroke="#1b2a4a" stroke-width="1"/>
          <!-- dot matrix overlay on globe -->
          <circle cx="20" cy="20" r="18" fill="url(#dots)" opacity="0.4"/>
          <defs>
            <pattern id="dots" x="0" y="0" width="4" height="4" patternUnits="userSpaceOnUse">
              <circle cx="1" cy="1" r="0.6" fill="#1b2a4a"/>
            </pattern>
          </defs>
        </svg>
        <div class="org-logo-text">
          <span>GLOBAL</span>
          <span>SECURITY</span>
          <span>FORUM</span>
        </div>
      </div>

      <div class="report-label">GLOBAL SECURITY FORUM</div>
      <div class="report-title">OVERVIEW<br>&amp; FINDINGS<br>REPORT</div>

      <div class="report-footer">
        <div class="footer-publisher">THE SOUFAN CENTER</div>
        <div class="footer-org">GLOBAL SECURITY FORUM</div>
      </div>

    </div>
  </div>

  <!-- ══ FRONT REPORT ══ -->
  <div class="report report-front">
    <div class="report-inner">

      <div class="org-logo-block">
        <svg class="org-logo-icon" viewBox="0 0 40 40" fill="none" xmlns="http://www.w3.org/2000/svg">
          <circle cx="20" cy="20" r="18" stroke="#1b2a4a" stroke-width="1.5" fill="none"/>
          <ellipse cx="20" cy="20" rx="9" ry="18" stroke="#1b2a4a" stroke-width="1" fill="none"/>
          <ellipse cx="20" cy="20" rx="18" ry="7" stroke="#1b2a4a" stroke-width="1" fill="none"/>
          <line x1="2" y1="20" x2="38" y2="20" stroke="#1b2a4a" stroke-width="1"/>
          <line x1="20" y1="2" x2="20" y2="38" stroke="#1b2a4a" stroke-width="1"/>
          <circle cx="20" cy="20" r="18" fill="url(#dots2)" opacity="0.4"/>
          <defs>
            <pattern id="dots2" x="0" y="0" width="4" height="4" patternUnits="userSpaceOnUse">
              <circle cx="1" cy="1" r="0.6" fill="#1b2a4a"/>
            </pattern>
          </defs>
        </svg>
        <div class="org-logo-text">
          <span>GLOBAL</span>
          <span>SECURITY</span>
          <span>FORUM</span>
        </div>
      </div>

      <div class="report-label">GLOBAL SECURITY FORUM 2021</div>
      <div class="report-title">OVERVIEW<br>&amp; FINDINGS<br>REPORT</div>

      <div class="report-footer">
        <div class="footer-publisher">THE SOUFAN CENTER</div>
        <div class="footer-org">GLOBAL SECURITY FORUM</div>
      </div>

    </div>
  </div>

</div>

</body>
</html>
```
