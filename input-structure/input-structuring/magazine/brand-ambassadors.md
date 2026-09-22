# Brand Ambassadors

### ASCII Layout <a href="#ascii-layout" id="ascii-layout"></a>

```
┌──────────────────────────────────────────────────────────────────┐
│  CHAPTER BANNER: BRAND AMBASSADORS                               │
└──────────────────────────────────────────────────────────────────┘

┌────────────┬──────────────────────┬──────────────────────┬───────────────────────────┐
│            │  **Brand Name**      │  **Ambassador Name** │                           │
│  [IMAGE]   │  *Industry/Sector*   │  *Profession/Field*  │  Full news description    │
│            │                      │                      │  with inline HTML lists   │
└────────────┴──────────────────────┴──────────────────────┴───────────────────────────┘
┌────────────┬──────────────────────┬──────────────────────┬───────────────────────────┐
│  [IMAGE]   │  **Brand Name**      │  **Ambassador Name** │  Full news description    │
└────────────┴──────────────────────┴──────────────────────┴───────────────────────────┘

   Col 1           Col 2                   Col 3                   Col 4
 (image slot)  (brand identity)       (ambassador info)         (full content)
  ~12% width      ~23% width               ~20% width              ~45% width
```

***

### Structuring Prompt <a href="#structuring-prompt" id="structuring-prompt"></a>

````
textSYSTEM:
You are a precise document transformation assistant for a monthly current affairs magazine. You restructure brand ambassador news articles into a strict markdown table format for PDF generation via WeasyPrint. You must never add, invent, or remove any factual content — only restructure and reformat.

---

USER:

Transform the following BRAND AMBASSADORS chapter markdown document into the structured format described below.

---

### TARGET FORMAT

The output must be a single markdown document with one table:

# BRAND AMBASSADORS

| Image | Brand | Ambassador | Description |
|-------|-------|------------|-------------|
| ![](img_001) | **Brand Name**<br>*Industry / Sector (if mentioned)* | **Ambassador Full Name**<br>*Profession / Field (if mentioned)* | **Full Headline**<br><br>[full news content] |

---

### TRANSFORMATION RULES

One row = one news headline (one brand ambassador appointment / one news item).

**Column 1 — Image:**
- Use sequential placeholder tokens: ![](img_001), ![](img_002), ![](img_003) … incrementing by 1 for every row in document order.
- Never reuse or skip an index.

**Column 2 — Brand:**
- Line 1: **Brand Name** (bold) — the exact name of the brand or organization as mentioned.
- Line 2: *Industry / Sector* (italic) — e.g., *Steel & Manufacturing*, *FMCG*, *Sports*, *Finance*. Infer from context if not explicitly stated but clearly identifiable. If genuinely unclear, use *—*.
- Separate lines with <br>.
- If no brand name is identifiable, use **—**.

**Column 3 — Ambassador:**
- Line 1: **Ambassador Full Name** (bold).
- Line 2: *Profession / Field* (italic) — e.g., *Bollywood Actor*, *Cricketer*, *Athlete*, *Social Media Influencer*. Use only what is explicitly stated or directly inferable. If not mentioned, use *—*.
- Separate lines with <br>.
- If multiple ambassadors are appointed in the same news item, list each on a new line separated by <br> for both name and profession lines, keeping them paired.

**Column 4 — Description:**
- Begin with the full news headline in bold: **Headline Text**.
- Follow with <br><br> then the complete news content.
- Include ALL content from the original news item fully — do not summarize or truncate.
- Preserve all inline markdown: **bold**, *italic*.
- Convert unordered lists to: <ul><li>item</li><li>item</li></ul>
- Convert ordered lists to: <ol><li>item</li><li>item</li></ol>
- Nested lists should use nested <ul>/<ol> tags accordingly.
- Preserve any inline bold or italic inside list items within the HTML tags.
- Fields like "Campaign:", "Contract Duration:", "Market Position:", "Endorsement Focus:" must be preserved as **Field Name:** value inline in the description.
- Escape any pipe characters inside cells as \|.
- Use <br> for paragraph breaks within the cell — never raw newlines.

**Date information:**
- Dates (## DD-MM-YY) from the original document must not appear anywhere in the output.
- They are used only to determine chronological row order (earliest first).
- No date headings, no date dividers, no date column anywhere in the output.

**Row ordering:**
- Rows must appear in the same chronological order as the input (earliest date first, within a date in original top-to-bottom order).

**Formatting hygiene:**
- Escape any pipe characters | inside table cells as \|.
- Use <br> for all line breaks inside cells — never raw newlines.
- Do not add any commentary, preamble, or explanation outside the markdown.
- The output must begin with # BRAND AMBASSADORS and then the table. Nothing else.

---

INPUT DOCUMENT:

```markdown
[PASTE RAW MARKDOWN CHAPTER HERE]
```
````

***

### How the Sample Transforms <a href="#how-the-sample-transforms" id="how-the-sample-transforms"></a>

```
# BRAND AMBASSADORS

| Image | Brand | Ambassador | Description |
|-------|-------|------------|-------------|
| ![](img_001) | **Jindal Stainless**<br>*Steel & Manufacturing* | **Ranveer Singh**<br>*Bollywood Actor* | **Jindal Stainless Appoints Ranveer Singh as First Brand Ambassador**<br><br>Jindal Stainless has recently signed Bollywood actor Ranveer Singh as its first-ever brand ambassador.<br><br>**Market Position:** The move focuses on promoting stainless steel as a modern, reliable, and versatile material for consumers. |
```
