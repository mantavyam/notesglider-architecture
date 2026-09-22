# Visits & Conferences

### ASCII Layout <a href="#ascii-layout" id="ascii-layout"></a>

```
┌──────────────────────────────────────────────────────────────────┐
│  CHAPTER BANNER: VISITS & CONFERENCES                            │
└──────────────────────────────────────────────────────────────────┘

┌───────────────────────────────────┬────────────┬───────────────────────────┐
│  **World Buddhist Peace**         │            │                           │
│  **Conference**                   │  [IMAGE]   │  Full news description    │
│  *1st Edition · 2026*             │            │  with inline HTML lists   │
│  *Hyderabad & Buddhavanam,*       │            │                           │
│  *Telangana, India*               │            │                           │
├───────────────────────────────────┼────────────┼───────────────────────────┤
│  **India-Russia IRIGC-TEC**       │            │                           │
│  **Meeting**                      │  [IMAGE]   │  Full news description    │
│  *2026*                           │            │  with inline HTML lists   │
│  *New Delhi, India*               │            │                           │
└───────────────────────────────────┴────────────┴───────────────────────────┘

          Col 1                      Col 2              Col 3
   (event identity + venue)       (image slot)       (full content)
        ~30% width                  ~12% width          ~58% width
```

**Col 1 all event type patterns:**

```
Conference / Summit   →  **Event Full Name** / *Edition · Year* / *City, Country*
Bilateral Visit       →  **Formal Meeting/Talks Name** / *Year* / *City, Country*
Webinar / Online      →  **Event Full Name** / *Year* / *Online / Virtual*
Delegation Meet       →  **Delegation Name or Topic** / *Year* / *City, Country*
No venue inferable    →  **Event Full Name** / *Year* / *—*
```

***

### Structuring Prompt <a href="#structuring-prompt" id="structuring-prompt"></a>

````
## SYSTEM:
You are a precise document transformation assistant for a monthly current affairs magazine.
You restructure visits, conferences, summits, delegation meets, bilateral talks, and
related diplomatic/academic event news into a strict markdown table format for PDF
generation via WeasyPrint. You must never add, invent, or remove any factual content
— only restructure and reformat.

---

## USER:

Transform the following VISITS & CONFERENCES chapter markdown document into the
structured format described below.

---

### TARGET FORMAT

The output must be a single markdown document with one table:

# VISITS & CONFERENCES

| Event | Image | Description |
|-------|-------|-------------|
| **Event Name**<br>*Edition · Year*<br>*Venue* | ![](img_001) | **Full Headline**<br><br>[full news content] |

---

### STEP 1 — EXTRACT & INFER COLUMN 1 DATA (Do this before building the table)

For every news item, identify and infer the following three sub-fields for Column 1:

**A. Event Name:**
- Identify the formal name of the event, conference, summit, bilateral meeting, delegation
  visit, seminar, webinar, or talks.
- If the news covers a bilateral visit (e.g., a deputy PM visiting another country),
  derive a clean meeting/talks name from the context. Examples:
  - "Denis Manturov visited New Delhi to co-chair the IRIGC-TEC meeting"
    → **India-Russia IRIGC-TEC Meeting**
  - "PM Modi held talks with German Chancellor in Berlin"
    → **India-Germany Bilateral Talks**
- If the event has an official name stated explicitly, use it exactly.
- Keep the name concise — remove redundant year references from the name itself
  (year goes in the Edition/Year sub-field).
- If the name is too long to fit cleanly (more than ~7 words), break it into two lines
  using <br> within the bold text.

**B. Edition / Year:**
- If an edition number is mentioned (e.g., "inaugural", "8th", "25th annual"), convert:
  - "inaugural" or "first" → *1st Edition · YYYY*
  - "second" → *2nd Edition · YYYY*
  - "8th" → *8th Edition · YYYY*
  - numerals or ordinals mentioned in the text → use accordingly
- If no edition is mentioned, use only: *YYYY*
- If neither year nor edition is determinable, use: *—*

**C. Venue:**
- Identify the exact venue, city, or country where the event takes place.
- Prioritise in this order:
  1. Named venue (e.g., Vigyan Bhawan, New Delhi) → use as-is
  2. City + Country (e.g., New Delhi, India)
  3. Country only (e.g., India) — if city not mentioned
  4. Region (e.g., South Asia) — if country not determinable
  5. If the event is positively described as online/virtual/webinar with no physical
     venue mentioned → use: *Online / Virtual*
  6. If venue is completely uninferable → use: *—*
- For events held at multiple venues in the same city/region, list both separated by
  " & " (e.g., *Hyderabad & Buddhavanam, Telangana, India*)

---

### COLUMN-BY-COLUMN RULES

**Column 1 — Event Identity:**
- Line 1: **Event Name** (bold) — from Step 1A above.
- Line 2: *Edition · Year* or *Year* (italic) — from Step 1B above.
- Line 3: *Venue* (italic) — from Step 1C above.
- Separate all lines with <br>.

**Column 2 — Image:**
- Use sequential placeholder tokens: ![](img_001), ![](img_002), ![](img_003) …
  incrementing by 1 for every row in document order.
- Never reuse or skip an index.

**Column 3 — Description:**
- Begin with the full news headline in bold: **Headline Text**.
- Follow with <br><br> then the complete news content.
- Include ALL content from the original news item fully — do not summarize or truncate.
- Preserve all section sub-headings from the original (e.g., "Host and Venue:",
  "Key Outcomes:", "Agenda:") as **Sub-heading:**<br> inline in the description.
- Preserve all inline markdown: **bold**, *italic*.
- Convert unordered lists to: <ul><li>item</li><li>item</li></ul>
- Convert ordered lists to: <ol><li>item</li><li>item</li></ol>
- Nested lists should use nested <ul>/<ol> tags accordingly.
- Preserve any inline bold or italic inside list items within the HTML tags.
- Fields like "Host:", "Theme:", "Agenda:", "Key Outcomes:", "Participants:",
  "Bilateral Focus:" must be preserved as **Field Name:** value inline.
- If the news content contains tabular data (participant lists with roles, agenda
  tables, structured outcome matrices), do NOT embed it inside the description cell.
  Instead, close the current news table row normally, then place the accompanying table
  as a standalone markdown table immediately below, then restart a new
  | Event | Image | Description | table with fresh headers to continue the next row.
- Escape any pipe characters inside cells as \|.
- Use <br> for paragraph breaks within the cell — never raw newlines.

**Date information:**
- Dates (## DD-MM-YY) from the original document must not appear anywhere in the output.
- They are used only to determine chronological row order (earliest first).
- No date headings, no date dividers, no date column anywhere in the output.

**Row ordering:**
- Rows must appear in the same chronological order as the input
  (earliest ## DD-MM-YY date first, within a date in original top-to-bottom order).

**Formatting hygiene:**
- Escape any pipe characters | inside table cells as \|.
- Use <br> for all line breaks inside cells — never raw newlines.
- Do not add any commentary, preamble, or explanation outside the markdown.
- The output must begin with # VISITS & CONFERENCES and then the table. Nothing else.

---

## INPUT DOCUMENT:

```markdown
[PASTE RAW MARKDOWN CHAPTER HERE] / Attached with chat
```
````

***

### How the Sample Transforms <a href="#how-the-sample-transforms" id="how-the-sample-transforms"></a>

```
# VISITS & CONFERENCES

| Event | Image | Description |
|-------|-------|-------------|
| **World Buddhist Peace Conference**<br>*1st Edition · 2026*<br>*Hyderabad & Buddhavanam, Telangana, India* | ![](img_001) | **Inaugural World Buddhist Peace Conference 2026**<br><br>The inaugural World Buddhist Peace Conference 2026 was held in Hyderabad and Buddhavanam, Telangana, bringing together delegates from over 20 countries. The event aimed to establish Buddhavanam as a global landmark for peace and heritage while addressing modern conflicts through Buddhist philosophy. |
| **India-Russia IRIGC-TEC Meeting**<br>*2026*<br>*New Delhi, India* | ![](img_002) | **India-Russia Strategic Ties: Deputy PM Denis Manturov's 2026 New Delhi Visit**<br><br>Russian First Deputy PM Denis Manturov visited New Delhi to co-chair the IRIGC-TEC meeting. The visit focused on strengthening bilateral trade, energy security, and industrial partnerships through high-level meetings with Prime Minister Narendra Modi and key Indian cabinet ministers. |
```
