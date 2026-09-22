# Appointments

### ASCII Layout <a href="#ascii-layout" id="ascii-layout"></a>

```
┌──────────────────────────────────────────────────────────────────┐
│  CHAPTER BANNER: APPOINTMENTS                                    │
└──────────────────────────────────────────────────────────────────┘

┌────────────┬──────────────────────────────────────┬──────────────────────────────┐
│            │  **Stuti Pradhan**                   │                              │
│  [IMAGE]   │  *Delegate*                          │  Full news description       │
│            │  *World Youth Parliament*            │  with inline HTML lists      │
├────────────┼──────────────────────────────────────┼──────────────────────────────┤
│            │  **Lt Gen Pushpendra Pal Singh**     │                              │
│  [IMAGE]   │  *GOC-in-C, Western Command*         │  **Major Appointments in     │
│            │  *Indian Army*                       │  Defence Sector** (headline) │
│            │  ─────────────────────────────       │                              │
│            │  **Lt Gen Dhiraj Seth**              │  All persons described in    │
│            │  *Vice Chief of the Army Staff*      │  full with their bullet      │
│            │  *Indian Army*                       │  content, structured         │
│            │  ─────────────────────────────       │  clearly under each name     │
│            │  **Lt Gen Sandeep Jain**             │  as a sub-section            │
│            │  *GOC-in-C, Southern Command*        │                              │
│            │  *Indian Army*                       │                              │
└────────────┴──────────────────────────────────────┴──────────────────────────────┘

   Col 1              Col 2                              Col 3
 (image slot)   (person(s) name + designation)        (full content)
  ~12% width          ~28% width                        ~60% width
```

**Single vs. Multi-appointment Col 2 pattern:**

```
textSingle person  →  **Name**
                  *Designation*
                  *Organization*

Multi-person   →  **Name 1**
                  *Designation 1*
                  *Organization 1*
                  ───────────────   ← <hr> visual divider between persons
                  **Name 2**
                  *Designation 2*
                  *Organization 2*
```

***

### Structuring Prompt <a href="#structuring-prompt" id="structuring-prompt"></a>

````
## SYSTEM:
You are a precise document transformation assistant for a monthly current affairs
magazine. You restructure appointment news articles into a strict markdown table
format for PDF generation via WeasyPrint. You must never add, invent, or remove
any factual content — only restructure and reformat.

---

USER:

Transform the following APPOINTMENTS chapter markdown document into the structured
format described below.

---

### TARGET FORMAT

The output must be a single markdown document with one table:

# APPOINTMENTS

| Image | Person | Description |
|-------|--------|-------------|
| ![](img_001) | **Full Name**<br>*Designation*<br>*Organization* | **Full Headline**<br><br>[full news content] |

---

### STEP 1 — IDENTIFY APPOINTMENT TYPE PER NEWS ITEM

Before building the table, classify each news headline (### level) as:

- SINGLE APPOINTMENT: the news covers exactly one person being appointed.
- MULTI APPOINTMENT: the news covers two or more persons being appointed
  under the same headline (e.g., "Major Appointments in Defence Sector").

This classification determines how Column 2 is structured.

---

### STEP 2 — EXTRACT PERSON DATA

For each person in every news item, extract:

**A. Full Name:**
- The complete name of the appointed person as stated in the source.
- Use the most complete form available (e.g., "Lieutenant General Pushpendra Pal Singh",
  not "Lt Gen Singh").

**B. Designation:**
- The NEW role/post the person has been appointed to.
- Use the exact title as stated. If a short form and long form both exist, use the
  long form with short form in parentheses where helpful.
  Example: General Officer Commanding-in-Chief (GOC-in-C)
- If no designation is explicitly stated, infer the most accurate label from context.
- If uninferable, use *—*.

**C. Organization / Body:**
- The ministry, command, institution, department, country, or international body
  the person now serves under or represents.
- Example: Indian Army — Western Command, Rajya Sabha, World Youth Parliament.
- If not mentioned, use *—*.

---

### COLUMN-BY-COLUMN RULES

**Column 1 — Image:**
- Use sequential placeholder tokens: ![](img_001), ![](img_002), ![](img_003) …
  incrementing by 1 for every ROW (one row = one news headline) in document order.
- Never reuse or skip an index.
- One image token per row regardless of how many persons are in that news item.

**Column 2 — Person:**

FOR SINGLE APPOINTMENT rows:
- Line 1: **Full Name** (bold)
- Line 2: *Designation* (italic)
- Line 3: *Organization* (italic)
- Separate lines with <br>.

FOR MULTI APPOINTMENT rows:
- List ALL persons from that news headline in Column 2, in the exact order they appear
  in the source.
- For each person use:
  **Full Name**<br>*Designation*<br>*Organization*
- Separate each person block from the next using: <br><hr><br>
  This creates a clear visual divider between persons within the same cell.
- Do NOT create separate rows for each person in a multi-appointment news —
  all persons stay in a single row, in Column 2, under the same news image.

**Column 3 — Description:**
- Begin with the full news headline in bold: **Headline Text**.
- Follow with <br><br> then the complete news content.
- Include ALL content from the original news item fully — do not summarize or truncate.
- FOR MULTI APPOINTMENT news: preserve each person's sub-section clearly.
  Each named sub-heading (e.g., "Lieutenant General Pushpendra Pal Singh") must appear
  as **Person Name**<br> followed by their bullet content as a <ul> list, then a <br>
  before the next person's sub-section begins. This ensures the description mirrors
  the structure of Column 2 for visual coherence.
- Preserve all inline markdown: **bold**, *italic*.
- Convert unordered lists to: <ul><li>item</li><li>item</li></ul>
- Convert ordered lists to: <ol><li>item</li><li>item</li></ol>
- Nested lists should use nested <ul>/<ol> tags accordingly.
- Preserve any inline bold or italic inside list items within the HTML tags.
- Fields like "Previous Position:", "Preceded By:", "Previous Role:", "Succeeds:"
  must be preserved as **Field Name:** value inline in the description.
- Escape any pipe characters inside cells as \|.
- Use <br> for paragraph breaks within the cell — never raw newlines.

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
- The output must begin with # APPOINTMENTS and then the table. Nothing else.

---

## INPUT DOCUMENT:

```markdown
[PASTE RAW MARKDOWN CHAPTER HERE] / Attached with Chat
```
````

***

### How the Sample Transforms <a href="#how-the-sample-transforms" id="how-the-sample-transforms"></a>

```
# APPOINTMENTS

| Image | Person | Description |
|-------|--------|-------------|
| ![](img_001) | **Stuti Pradhan**<br>*Delegate*<br>*World Youth Parliament* | **Stuti Pradhan to Represent India at World Youth Parliament**<br><br><ul><li>Stuti Pradhan from Sikkim has been chosen as a delegate for the international World Youth Parliament.</li><li>This platform gathers young leaders from various nations to address global challenges and create policy solutions.</li></ul> |
| ![](img_002) | **Lieutenant General Pushpendra Pal Singh**<br>*General Officer Commanding-in-Chief (GOC-in-C)*<br>*Indian Army — Western Command*<br><hr><br>**Lt Gen Dhiraj Seth**<br>*Vice Chief of the Army Staff*<br>*Indian Army*<br><hr><br>**Lieutenant General Sandeep Jain**<br>*General Officer Commanding-in-Chief (GOC-in-C)*<br>*Indian Army — Southern Command* | **Major Appointments in Defence Sector**<br><br>**Lieutenant General Pushpendra Pal Singh**<br><ul><li>Lieutenant General Pushpendra Pal Singh has officially taken over as the GOC-in-C of the Indian Army's Western Command.</li><li>He succeeds Lt Gen Manoj Kumar Katiyar, who retired on March 31, 2026.</li><li>**Previous Position:** Vice Chief of the Army Staff.</li></ul><br>**Lt Gen Dhiraj Seth**<br><ul><li>Lt Gen Dhiraj Seth has officially taken charge as the Vice Chief of the Army Staff.</li><li>**Preceded By:** Lt Gen Pushpendra Pal Singh, who moved to head the Western Command at Chandimandir.</li><li>**Previous Role:** General Officer Commanding-in-Chief (GOC-in-C), Southern Command (based in Pune).</li></ul><br>**Lieutenant General Sandeep Jain**<br><ul><li>Lieutenant General Sandeep Jain officially assumed the role of GOC-in-C of the Indian Army's Southern Command.</li></ul> |
```
