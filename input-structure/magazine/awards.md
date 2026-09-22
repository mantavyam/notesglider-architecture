# Awards

### ASCII Layout <a href="#ascii-layout" id="ascii-layout"></a>

```
┌──────────────────────────────────────────────────────────────────┐
│  CHAPTER BANNER: AWARDS & RECOGNITION                           │
└──────────────────────────────────────────────────────────────────┘

┌─────────────────────────┬────────────┬──────────────────────────┐
│  **Award Title**        │            │                          │
│  *Awarding Org*         │  [IMAGE]   │  Full news description   │
│  *8th Edition · 2025*   │            │  with **bold**, *italic* │
│                         │            │  bullet points, etc.     │
└─────────────────────────┴────────────┴──────────────────────────┘
         ↓  (if the news has an accompanying table)
┌──────────────────────────────────────────────────────────────────┐
│  | Category | Winner | Prize Money | ...                        │
│  |----------|--------|-------------|                             │
│  | Player of Year (Men) | Hardik Singh | ₹20 Lakh |            │
│  | Player of Year (Women) | Navneet Kaur | ₹20 Lakh |         │
│  ...                                                             │
└──────────────────────────────────────────────────────────────────┘

┌─────────────────────────┬────────────┬──────────────────────────┐
│  **Next Award Title**   │            │                          │
│  *Awarding Org*         │  [IMAGE]   │  Next news description   │
│  *—*                    │            │                          │
└─────────────────────────┴────────────┴──────────────────────────┘

       Col 1                 Col 2              Col 3
  (award identity)        (image slot)       (full content)
     ~30% width             ~15% width          ~55% width
```

**Pattern:**

```
[News Table Row]
[Accompanying Data Table — only if present]
[News Table Row]
[Accompanying Data Table — only if present]
...
```

***

### Structuring Prompt <a href="#structuring-prompt" id="structuring-prompt"></a>

````
## SYSTEM:
You are a precise document transformation assistant for a monthly current affairs
magazine. You restructure awards and recognition news articles into a strict markdown
format for PDF generation via WeasyPrint. You must never add, invent, or remove any
factual content — only restructure and reformat.

---

## USER:

Transform the following AWARDS & RECOGNITION chapter markdown document into the
structured format described below.

---

### TARGET FORMAT

The output is a sequence of news blocks. Each news block follows this exact pattern:

[3-column news table row]
[standalone accompanying table — ONLY if tabular data is present in that news item]

This pattern repeats for every news item. Each news block's table is independent —
close the table after each row, place the accompanying table if applicable, then open
a fresh table with headers for the next news item.

Structure of each news block:

| Award | Image | Description |
|-------|-------|-------------|
| **Award Title**<br>*Awarding Organization*<br>*Edition · Year* | ![](img_001) | **Full Headline**<br><br>[news intro content] |

Followed immediately by (if applicable):

| Column A | Column B | ... |
|----------|----------|-----|
| data     | data     | ... |

Then a new table begins for the next news item.

The full output must begin with:
# AWARDS & RECOGNITION

---

### STEP 1 — EXTRACT & INFER COLUMN 1 DATA (Do this before building the table)

For every news item, identify and infer the following three sub-fields for Column 1:

**A. Award Title:**
- The exact name of the award, award ceremony, or recognition event.
- If the news covers a full award ceremony (e.g., "Hockey India 8th Annual Awards"),
  use the ceremony name as the title, keeping it concise.
- If the news covers a single specific award given to one person, use the award name.
- Remove redundant year references from the title — year goes in Edition/Year field.
- If no clear award title is identifiable, use **—**.

**B. Awarding Organization:**
- The body, institution, ministry, government, or association conferring the award.
- Use the exact name as stated. If only an abbreviated form is given, expand it if
  the full form is well-known (e.g., BCCI → Board of Control for Cricket in India).
- If not mentioned, use *—*.

**C. Edition / Year:**
- If an edition number is mentioned, convert to ordinal format:
  "inaugural" or "first" → *1st Edition · YYYY*
  "8th Annual" → *8th Edition · YYYY*
  numerals/ordinals stated in text → use accordingly with year
- If no edition is mentioned but year is available, use only: *YYYY*
- If neither is available, use: *—*

---

### STEP 2 — ASSESS FOR ACCOMPANYING TABLE

Before writing each news block, assess whether the news content contains structured
tabular data such as:
- Multiple award category winners (category + recipient pairs)
- Ranked or numbered lists of recipients
- Prize money, scores, or marks per recipient
- Any list of comparable entities with two or more attributes each

If YES → the introductory/contextual paragraph(s) go in Column 3 of the news table
row, and all structured winner/category data moves to the standalone accompanying
table placed immediately after the news table row.

If NO → all content goes fully into Column 3. Do not create an accompanying table.

---

### COLUMN-BY-COLUMN RULES

**Column 1 — Award Identity:**
- Line 1: **Award Title** (bold) — from Step 1A.
- Line 2: *Awarding Organization* (italic) — from Step 1B.
- Line 3: *Edition · Year* or *Year* (italic) — from Step 1C.
- Separate all lines with <br>.

**Column 2 — Image:**
- Use sequential placeholder tokens: ![](img_001), ![](img_002), ![](img_003) …
  incrementing by 1 for every news row in document order.
- Never reuse or skip an index.
- One image token per news row — not per winner or per category.

**Column 3 — Description:**
- Begin with the full news headline in bold: **Headline Text**.
- Follow with <br><br> then the news content.
- If an accompanying table will follow (Step 2 = YES): include only the
  introductory/contextual paragraph(s) here — the sentences that set the scene,
  describe the event, its purpose, venue, or significance. Do not include the
  winner lists or category data in Column 3 in this case.
- If no accompanying table (Step 2 = NO): include ALL content from the original
  news item fully here — do not summarize or truncate.
- Preserve all inline markdown: **bold**, *italic*.
- Convert unordered lists to: <ul><li>item</li><li>item</li></ul>
- Convert ordered lists to: <ol><li>item</li><li>item</li></ol>
- Nested lists should use nested <ul>/<ol> tags accordingly.
- Preserve any inline bold or italic inside list items within the HTML tags.
- Fields like "Venue:", "Theme:", "Hosted By:", "Significance:" must be preserved
  as **Field Name:** value inline in the description.
- Escape any pipe characters inside cells as \|.
- Use <br> for paragraph breaks within the cell — never raw newlines.

---

### ACCOMPANYING TABLE RULES (only when Step 2 = YES)

- Place the accompanying table immediately after the closing row of that news block's
  main table, before the next news block begins.
- Infer appropriate column headers from the content. Do not use a fixed schema —
  only include columns for data that is actually present.
  Typical headers for this chapter (adapt as needed):
  | Category | Award Name | Recipient | Prize Money |
- Use standard markdown table format wherever possible.
- If the original data contains merged cell logic (e.g., a group header spanning
  multiple rows), use an HTML <table> with colspan / rowspan instead of markdown
  table syntax for that accompanying table only.
- Do not create an accompanying table for news with a single winner or purely
  prose content.

---

**Date information:**
- Dates (## DD-MM-YY) from the original document must not appear anywhere in output.
- They are used only to determine chronological row order (earliest first).
- No date headings, no date dividers, no date column anywhere in the output.

**Row ordering:**
- Rows must appear in the same chronological order as the input
  (earliest ## DD-MM-YY date first, within a date in original top-to-bottom order).

**Formatting hygiene:**
- Escape any pipe characters | inside table cells as \|.
- Use <br> for all line breaks inside cells — never raw newlines.
- Do not add any commentary, preamble, or explanation outside the markdown.
- The output must begin with # AWARDS & RECOGNITION followed immediately by the
  first news block. Nothing else before it.

---

## INPUT DOCUMENT:

```markdown
[PASTE RAW MARKDOWN CHAPTER HERE]
```
````

***

### How the Sample Transforms <a href="#how-the-sample-transforms" id="how-the-sample-transforms"></a>

```
# AWARDS & RECOGNITION

| Award | Image | Description |
|-------|-------|-------------|
| **Hockey India Annual Awards**<br>*Hockey India*<br>*8th Edition · 2025* | ![](img_001) | **Hockey India 8th Annual Awards 2025: Key Winners Announced**<br><br>The 8th Hockey India Annual Awards 2025 were held in New Delhi. The ceremony celebrated the outstanding achievements of Indian hockey players and contributors during the 2025 season. |

| Category | Award Name | Recipient | Prize Money |
|----------|------------|-----------|-------------|
| Player of the Year (Men) | Balbir Singh Sr. Award | Hardik Singh | ₹20 Lakh |
| Player of the Year (Women) | Balbir Singh Sr. Award | Navneet Kaur | ₹20 Lakh |
| Lifetime Achievement | Major Dhyan Chand Award | Zafar Iqbal | ₹25 Lakh |
| Best Member Unit | — | Hockey Jharkhand | ₹2.5 Lakh |
| Goalkeeper of the Year | Baljit Singh Award | Bichu Devi Kharibam | ₹5 Lakh |
| Defender of the Year | Pargat Singh Award | Sanjay | ₹5 Lakh |
| Midfielder of the Year | Ajit Pal Singh Award | Sumit | ₹5 Lakh |
| Forward of the Year | Dhanraj Pillay Award | Sukhjeet Singh | ₹5 Lakh |

| Award | Image | Description |
|-------|-------|-------------|
| **[Next Award Title]**<br>*[Org]*<br>*[Year]* | ![](img_002) | **[Next Headline]**<br><br>[next news content] |
```
