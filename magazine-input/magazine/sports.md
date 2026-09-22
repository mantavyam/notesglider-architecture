# Sports

### ASCII Layout <a href="#ascii-layout" id="ascii-layout"></a>

```asciidoc
┌──────────────────────────────────────────────────────────────────┐
│  CHAPTER BANNER: SPORTS NEWS                                     │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────┬───────────────────────────────────────┬──────────────────┐
│                  │                                       │                  │
│  [SPORT LOGO]    │  Full news description                │  [NEWS IMAGE]    │
│  Cricket /       │  with inline HTML lists               │                  │
│  Badminton /     │                                       │  ![](img_001)    │
│  Tennis etc.     │                                       │                  │
│  (shared per     │                                       │                  │
│   sport group)   │                                       │                  │
├──────────────────┼───────────────────────────────────────┼──────────────────┤
│  [SPORT LOGO]    │  Next news description                │  [NEWS IMAGE]    │
│  (same sport =   │                                       │  ![](img_002)    │
│   same logo)     │                                       │                  │
└──────────────────┴───────────────────────────────────────┴──────────────────┘

      Col 1                    Col 2                       Col 3
  (sport logo img)         (full content)             (news image slot)
    ~15% width               ~65% width                  ~20% width
```

**Dual image logic:**

```
Col 1 — Sport logo placeholder:  ![sport](logo_cricket)   ← sport-type key, not sequential
Col 3 — News image placeholder:  ![](img_001)             ← sequential per news row
```

***

### PROMPT 1 — Pre-Processing: Group & Sort by Sport <a href="#prompt-1--pre-processing-group--sort-by-sport" id="prompt-1--pre-processing-group--sort-by-sport"></a>

```markdown
## SYSTEM:
You are a sports news categorisation assistant for a current affairs magazine. Your task is to read a list of sports news items and group them by sport type, infer sport types where missing, and produce a clean re-ordered document ready for table transformation.

---

## USER:

Read the following SPORTS NEWS chapter markdown document and perform the following steps:

---

### STEP 1 — IDENTIFY SPORT TYPE FOR EVERY NEWS ITEM

For each news headline (### level heading) in the document:
- If the sport type is explicitly stated in the headline or sub-heading, use it.
- If the sport type is NOT explicitly mentioned, infer it from the content or search it on the web (player names, terminology, equipment, competition names, governing bodies, etc.).
- If the sport type genuinely cannot be determined, mark it as UNCATEGORIZED.
- For news covering events, tournaments, or policies that span multiple sports, mark it as GENERAL SPORTS NEWS.

---

### STEP 2 — GROUP AND REORDER

Reorder all news items by grouping them under their sport type. Within each sport group, preserve the original chronological order (by ## DD-MM-YY date, earliest first).

Output format for grouped news:

## SPORT TYPE NAME
(all original markdown content of news item 1 preserved exactly)

## SPORT TYPE NAME
(all original markdown content of news item 2 preserved exactly — same sport, same ## heading)

Continue pattern for all groups.

Place groups in the following priority order:
1. Sports with the most news items appear first.
2. For equal counts, use alphabetical order of sport name.
3. GENERAL SPORTS NEWS group → placed at the very end before UNCATEGORIZED.
4. UNCATEGORIZED → placed absolutely last.

---

### STEP 3 — OUTPUT

Produce the full reordered markdown document:
- Begin with: # SPORTS NEWS
- Each sport group starts with: ## SPORT TYPE (e.g., ## BADMINTON, ## CRICKET)
- remove the date information, it is not required, in place of that use the ## heading level 2 for mentioning the sports category name. 
- Under each sport group, include all original news items (### headlines + full content) exactly as in the input — do not alter, summarize, or reformat any content.
- At the end, include: ## GENERAL SPORTS NEWS (if any), then ## UNCATEGORIZED (if any).
- Do not add any commentary or explanation outside the markdown.

---

## INPUT DOCUMENT:
[attached with chat]
---
begin by first mapping the sport to each headline
```

***

### PROMPT 2 — Transformation: Convert to 3-Column Table <a href="#prompt-2--transformation-convert-to-3-column-table" id="prompt-2--transformation-convert-to-3-column-table"></a>

````markdown
## SYSTEM:
You are a precise document transformation assistant for a monthly current affairs magazine. You restructure sports news articles into a strict markdown table format for PDF generation via WeasyPrint. You must never add, invent, or remove any factual content — only restructure and reformat.

---

USER:

Transform the following SPORTS NEWS chapter markdown document (which has already been pre-grouped by sport type) into the structured table format described below.

---

### TARGET FORMAT

The output is a sequence of sport-group blocks. Each sport group produces its own independent table.

For each sport group (## SPORT TYPE), output:

**Sport group header:**
## SPORT TYPE

**Followed immediately by its table:**

| Sport | Description | Image |
|-------|-------------|-------|
| ![sport](logo_SPORTTYPE) | **Full Headline**<br><br>[full news content] | ![](img_001) |

The pattern repeats for every sport group. Each group gets a fresh table with its own headers.

---

### TRANSFORMATION RULES

**Sport Group Header:**
- Preserve the ## SPORT TYPE heading exactly as it appears in the pre-grouped input (e.g., ## BADMINTON, ## CRICKET).
- This heading appears once per group, directly before that group's table.

**Column 1 — Sport Logo:**
- Use a sport-type-keyed placeholder in this format: ![sport](logo_SPORTTYPE)
- The SPORTTYPE key must be lowercase, no spaces, no special characters. Examples:
  - BADMINTON → ![sport](logo_badminton)
  - CRICKET → ![sport](logo_cricket)
  - TABLE TENNIS → ![sport](logo_tabletennis)
  - PARA SWIMMING → ![sport](logo_paraswimming)
- This same placeholder is repeated identically for EVERY row within that sport group — it represents a shared sport logo icon, not a per-news image.
- For GENERAL SPORTS NEWS group, use: ![sport](logo_generalsports)
- For UNCATEGORIZED group, use: ![sport](logo_unknown)

**Column 2 — Description:**
- Begin with the full news headline in bold: **Headline Text**.
- Follow with <br><br> then the complete news content.
- Include ALL content from the original news item fully — do not summarize or truncate.
- Preserve all section sub-headings from the original as **Sub-heading:**<br> inline.
- Preserve all inline markdown: **bold**, *italic*.
- Convert unordered lists to: <ul><li>item</li><li>item</li></ul>
- Convert ordered lists to: <ol><li>item</li><li>item</li></ol>
- Nested lists should use nested <ul>/<ol> tags accordingly.
- Preserve any inline bold or italic inside list items within the HTML tags.
- (if present) Fields like "Reason:", "Final Competition Status:", "Record:", "Achievement:" must be preserved as **Field Name:** value inline.
- If the news content contains tabular data (rankings, match scores, statistics tables), do NOT embed it inside the description cell. Instead, close the current news table row normally, then place the accompanying table as a standalone markdown table immediately below, then restart a new | Sport \| Description \| Image | table with fresh headers to continue the next row in that same sport group.
- Escape any pipe characters inside cells as \|.
- Use <br> for paragraph breaks within the cell — never raw newlines.

**Column 3 — News Image:**
- Use sequential placeholder tokens across the ENTIRE document (not resetting per sport group): ![](img_001), ![](img_002), ![](img_003) …
- The counter increments by 1 for every news row in the document, in the order they appear after grouping.
- Never reuse or skip an index.

**Date information:**
- Dates (## DD-MM-YY) from the original document must not appear anywhere in the output.
- No date headings, no date dividers, no date column anywhere in the output.

**Formatting hygiene:**
- Escape any pipe characters | inside table cells as \|.
- Use <br> for all line breaks inside cells — never raw newlines.
- Do not add any commentary, preamble, or explanation outside the markdown.
- The output must begin with # SPORTS NEWS followed immediately by the first sport group block. Nothing else before it.
- If the news content contains tabular data, do NOT embed it inside the description cell. Instead, close the current news table row normally, then place the accompanying table as a standalone markdown table immediately below that row, then begin a new '| Sport | Description | Image |' table with fresh headers to continue with the next news item. The pattern must be:
```  
[News Row in main table]
[Standalone accompanying table — (only if tabular data present, else avoid)]
[New main table with headers restarted for next news item]
```
---

## INPUT DOCUMENT:

```markdown
[PASTE PRE-GROUPED MARKDOWN FROM PROMPT 1 OUTPUT HERE]
```
````

***

### How the Sample Transforms <a href="#how-the-sample-transforms" id="how-the-sample-transforms"></a>

After Prompt 1 groups the content, Prompt 2 produces:

```
# SPORTS NEWS

## BADMINTON

| Sport | Description | Image |
|-------|-------------|-------|
| ![sport](logo_badminton) | **Badminton Legend Carolina Marin Announces Retirement**<br><br>Spanish badminton icon and former World No. 1 Carolina Marin has officially announced her retirement from professional badminton at the age of 32.<br><br><ul><li>**Reason:** Forced to take the decision due to a recurring knee injury sustained during the Paris 2024 Olympics (where she tore her ACL for the third time).</li><li>**Final Competition Status:** She has withdrawn from the upcoming European Championships in her hometown, Huelva, Spain.</li></ul> | ![](img_001) |
```
