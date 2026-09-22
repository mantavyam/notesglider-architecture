# Important Days

### ASCII Layout <a href="#ascii-layout" id="ascii-layout"></a>

```
text┌──────────────────────────────────────────────────────────────────┐
│  CHAPTER BANNER: IMPORTANT DAYS                                  │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────┬──────────────────────────────┬───────────────────────────┐
│  01                  │                              │                           │
│  Apr                 │  Utkal Divas                 │  Full news description    │
│                      │  (Odisha Foundation Day)     │  with inline HTML lists   │
├──────────────────────┼──────────────────────────────┼───────────────────────────┤
│  04–10               │                              │                           │
│  May                 │  Nationwide Fire Safety Week │  Full news description    │
│                      │                              │  with inline HTML lists   │
└──────────────────────┴──────────────────────────────┴───────────────────────────┘

      Col 1                    Col 2                       Col 3
 (date reference)         (day / week name)             (full content)
   ~15% width               ~30% width                   ~55% width
```

**Col 1 format decision tree:**

```
textSingle day         → DD (e.g., 01) + MMM
Date range         → DD1–DD2 + MMM
Week ordinal       → 1st/2nd/3rd/Last Week + MMM
Month-wide         → MMM (e.g., January)
Quarter            → Q1/Q2/Q3/Q4 + YYYY
Half year          → 1st Half / 2nd Half + YYYY
Full year (past)   → YYYY (position first)
Full year (current/future) → YYYY (position last)
```

**Col 1 all possible patterns:**

```
textSingle day      →   01          │  Week ordinal  →  2nd Week
                    Apr         │                   Mar
─────────────────────────────── │ ───────────────────────────
Date range      →   04–10       │  Month only    →  January
                    May         │
─────────────────────────────── │ ───────────────────────────
Combined weeks  →   3rd–4th     │  Quarter       →  Q1
                    Week, Jun   │                   2026
─────────────────────────────── │ ───────────────────────────
Half year       →   1st Half    │  Full year     →  2025
                    2026        │  (past → first position)
─────────────────────────────── │ ───────────────────────────
                                │  Full year     →  2026
                                │  (current/future → last)
```

**Repeated-date suppression rule:**

```
text│  01        │  Utkal Divas           │  ... │  ← date shown
│  Apr       │                        │      │
├────────────┼────────────────────────┼──────┤
│            │  April Fool's Day      │  ... │  ← same date,
│            │                        │      │    cell left blank
```

***

### Structuring Prompt <a href="#structuring-prompt" id="structuring-prompt"></a>

````
## SYSTEM:
You are a precise document transformation assistant for a monthly current affairs magazine. You restructure "Important Days" chapter news articles into a strict markdown table format for PDF generation via WeasyPrint. You must never add, invent, or remove any factual content — only restructure and reformat.

---

## USER:

Transform the following IMPORTANT DAYS chapter markdown document into the structured format described below.

---

### TARGET FORMAT

The output must be a single markdown document with one table:

# IMPORTANT DAYS

| Date | Day / Week | Description |
|------|------------|-------------|
| **DD**<br>MMM | **Celebration Name**<br>*Sub-title or alternate name (if any)* | **Full Headline**<br><br>[full news content] |

---

### STEP 1 — DATE IDENTIFICATION (Do this before building the table)

For every news item, identify the actual date of the celebration/observance. Follow these rules strictly:

1. Look for the date INSIDE the news headline or description body (e.g., "On April 1, 2026", "observed from 4th to 10th May", "every second Sunday of March").
2. Do NOT use the ## DD-MM-YY markdown heading as the celebration date. That heading only indicates when the news was captured in the editor and has no relation to the actual celebration date.
3. If no date is explicitly stated in the news content, perform search on the internet by web search tool and gather knowledge to identify the standard/official date of that day/observance.
4. Classify every news item into one of these date types:
   - SINGLE DAY: a fixed calendar date → e.g., April 1
   - DATE RANGE: a span of dates within a month → e.g., 4th–10th May
   - WEEK ORDINAL: a relative week → e.g., First Sunday of May, Last Week of March
   - COMBINED WEEKS: spanning two or more named weeks → e.g., 3rd–4th Week of June
   - MONTH ONLY: the entire month is the observance period → e.g., January declared as X Month
   - QUARTER: a 3-month block → e.g., Q1 2026 (Jan–Mar)
   - HALF YEAR: 6-month block → e.g., 1st Half 2026
   - FULL YEAR (PAST): the observance year is before the current year → e.g., 2025
   - FULL YEAR (CURRENT / FUTURE): the observance year is the current or a future year → e.g., 2026, 2027

---

### STEP 2 — ASCENDING DATE ORDERING (Do this before building the table)

Sort all news rows in ascending date order using these rules:

1. FULL YEAR (PAST) rows → place at the very top (before all other rows), earliest year first.
2. Then chronologically: SINGLE DAY / DATE RANGE / WEEK ORDINAL / COMBINED WEEKS, sorted by earliest start date within the year.
3. For WEEK ORDINALS (e.g., "First Sunday of May"), resolve to an approximate date for ordering purposes (do not display the resolved date, use it only for ordering).
4. MONTH ONLY rows → insert before the first SINGLE DAY / DATE RANGE row of that same month. If all rows are from the same month and a MONTH ONLY row exists for a different earlier month, place it first.
5. QUARTER rows → insert at the start of that quarter's first month position.
6. HALF YEAR rows → insert at the start of that half-year's first month position.
7. FULL YEAR (CURRENT / FUTURE) rows → place at the very end, earliest year first.
8. If multiple news items share the exact same date classification and date, preserve their original relative order from the input.

---

### STEP 3 — BUILD THE TABLE

**Column 1 — Date Reference:**

Format based on date type:
- SINGLE DAY → **DD** (bold, large) on line 1, plain MMM on line 2. e.g., **01**<br>Apr
- DATE RANGE → **DD1–DD2** (bold) on line 1, plain MMM on line 2. e.g., **04–10**<br>May. If range spans two months: **DD1 MMM1–**<br>**DD2 MMM2**.
- WEEK ORDINAL → **1st / 2nd / 3rd / Last** (bold) on line 1, **Week** on line 2, plain MMM on line 3. e.g., **1st**<br>**Week**<br>May
- COMBINED WEEKS → **Nth–Mth** (bold) on line 1, **Week** on line 2, plain MMM on line 3.
- MONTH ONLY → **MMM** (bold, full month name) on line 1 only.
- QUARTER → **Q1 / Q2 / Q3 / Q4** (bold) on line 1, plain YYYY on line 2.
- HALF YEAR → **1st Half / 2nd Half** (bold) on line 1, plain YYYY on line 2.
- FULL YEAR → **YYYY** (bold) on line 1 only.
- Separate lines with <br>.

**IMPORTANT — Repeated Date Suppression:**
If two or more consecutive rows share the exact same date reference (same DD/range/week/month/year AND same MMM), leave Column 1 EMPTY (blank cell) for all rows after the first occurrence. Never repeat the same date value in consecutive rows.

**Column 2 — Event Name:**
- Line 1: **Official Name of the Day / Week / Month** (bold) — the exact celebration name as used in the news.
- Line 2: *Sub-title or alternate name* (italic) — only if explicitly present in the source (e.g., *Odisha Foundation Day*, *Rongali Bihu*). Omit if not present.
- Separate lines with <br>.

**Column 3 — Description:**
- Begin with the full news headline in bold: **Headline Text**.
- Follow with <br><br> then the complete news content.
- Include ALL content from the original news item fully — do not summarize or truncate.
- Preserve all section sub-headings from the original as **Sub-heading:**<br> inline.
- Preserve all inline markdown: **bold**, *italic*.
- Convert unordered lists to: <ul><li>item</li><li>item</li></ul>
- Convert ordered lists to: <ol><li>item</li><li>item</li></ol>
- Nested lists should use nested <ul>/<ol> tags accordingly.
- Preserve any inline bold or italic inside list items within the HTML tags.
- (if present) Fields like "Theme:", "Duration:", "Aim:", "Organizer:", "Significance:" must be preserved as **Field Name:** value inline.
- Escape any pipe characters inside cells as \|.
- Use <br> for paragraph breaks within cells — never raw newlines.

**Formatting hygiene:**
- Escape any pipe characters | inside table cells as \|.
- Use <br> for all line breaks inside cells — never raw newlines.
- Do not add any commentary, preamble, or explanation outside the markdown.
- The output must begin with # IMPORTANT DAYS and then the table. Nothing else.

---

## INPUT DOCUMENT:

```markdown
[PASTE RAW MARKDOWN CHAPTER HERE] / Attached with Chat
```
````

***

### How the Sample Transforms <a href="#how-the-sample-transforms" id="how-the-sample-transforms"></a>

```
# IMPORTANT DAYS

| Date | Event | Description |
|------|------------|-------------|
| **01**<br>Apr | **Utkal Divas**<br>*Odisha Foundation Day* | **Utkal Divas 2026: The 90th Foundation Day of Odisha**<br><br>On April 1, 2026, the state of Odisha celebrates its formation day, known as Utkal Divas. This day commemorates the establishment of Odisha as a separate province in 1936, honoring the leaders who fought for the preservation of the Odia language and cultural identity. |
| **04–10**<br>May | **Nationwide Fire Safety Week** | **MoHFW to Observe Nationwide Fire Safety Week 2026**<br><br>The Ministry of Health and Family Welfare is set to observe Fire Safety Week to strengthen fire prevention and preparedness in healthcare facilities.<br><br>**Aim:** To promote fire safety awareness, improve compliance with safety norms, and foster a culture of safety among healthcare workers, students, and the general community.<br><br>**Event Details and Theme:**<br><ul><li>**Duration:** 4th to 10th of May 2026.</li><li>**Theme:** "Safe School, Safe Hospital and Fire Safety Aware Society – Together for Fire Prevention."</li></ul> |
```
