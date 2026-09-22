# Defence Exercise

Good — I have enough context to verify the sample data accuracy and build the prompt confidently. Here's the full deliverable:

***

### ASCII Layout <a href="#ascii-layout" id="ascii-layout"></a>

```
──────────────────────────────────────────────────────────────────┐
│  CHAPTER BANNER: DEFENCE EXERCISE                                │
└──────────────────────────────────────────────────────────────────┘

┌───────────────────────────┬────────────┬──────────────────────────────┬───────────────────────────┐
│  **Exercise Name**        │            │  **Location / Region**       │                           │
│  *Expedition Name*        │  [IMAGE]   │  *Venue (if mentioned)*      │  Full news description    │
│  *Edition · Year*         │            │  **Nations:** A, B, C, D...  │  with inline HTML lists   │
│                           │            │                              │                           │
└───────────────────────────┴────────────┴──────────────────────────────┴───────────────────────────┘
┌───────────────────────────┬────────────┬──────────────────────────────┬───────────────────────────┐
│  **Exercise Name**        │  [IMAGE]   │  **Location / Region**       │  Full news description    │
└───────────────────────────┴────────────┴──────────────────────────────┴───────────────────────────┘

        Col 1                  Col 2               Col 3                       Col 4
  (exercise identity)       (image slot)     (location + nations)           (full content)
      ~22% width              ~12% width           ~21% width                 ~45% width
```

**Nation listing pattern in Col 3:**

```
**Location / Region**
*Specific Venue (if any)*
──────────────────────
**Nations:** Bangladesh, France,
Indonesia, Kenya, Maldives,
Mauritius, Myanmar, Seychelles,
Singapore, Sri Lanka, Tanzania,
Timor-Leste
```

***

### Structuring Prompt <a href="#structuring-prompt" id="structuring-prompt"></a>

<pre><code>## SYSTEM:
You are a precise document transformation assistant for a monthly current affairs magazine. You restructure defence exercise and expedition news articles into a strict markdown table format for PDF generation via WeasyPrint. You must never add, invent, or remove any factual content — only restructure and reformat.

---

<strong>## USER:
</strong>
Transform the following DEFENCE EXERCISE chapter markdown document into the structured format described below.

---

### TARGET FORMAT

The output must be a single markdown document with one table:

# DEFENCE EXERCISE

| Exercise | Image | Location &#x26; Nations | Description |
|----------|-------|--------------------|-------------|
| **Exercise / Expedition Name**&#x3C;br>*Edition · Year (if available)* | ![](img_001) | **Location / Region**&#x3C;br>*Specific Venue (if mentioned)*&#x3C;br>**Nations:** Nation1, Nation2, Nation3… | **Full Headline**&#x3C;br>&#x3C;br>[full news content] |

---

### TRANSFORMATION RULES

One row = one news headline (one exercise / expedition / drill / operation).

**Column 1 — Exercise Identity:**
- Line 1: **Exercise / Expedition Name** (bold) — the exact official name of the exercise or expedition.
- Line 2: *Edition · Year* (italic) — e.g., *3rd Edition · 2026* or just *2026* if only year is available. If neither edition nor year is explicitly mentioned, use *—*.
- Separate lines with &#x3C;br>.
- If no name is identifiable, use **—**.

**Column 2 — Image:**
- Use sequential placeholder tokens: ![](img_001), ![](img_002), ![](img_003) … incrementing by 1 for every row in document order.
- Never reuse or skip an index.

**Column 3 — Location &#x26; Nations:**
- Line 1: **Location / Region** (bold) — the country, sea region, or geographic area where the exercise is held (e.g., **Indian Ocean Region**, **Rajasthan**, **South China Sea**).
- Line 2: *Specific Venue* (italic) — the named venue, base, or city if explicitly mentioned (e.g., *Maritime Warfare Centre, Kochi*). Omit this line if no specific venue is mentioned.
- Line 3: **Nations:** followed by a comma-separated list of ALL participating nations exactly as named in the source — do not omit, merge, or paraphrase any nation name. If nations are not mentioned, use **Nations:** *—*.
- Separate sections with &#x3C;br>.
- If the host nation is identified, list it first followed by the rest in the order they appear in the source.

**Column 4 — Description:**
- Begin with the full news headline in bold: **Headline Text**.
- Follow with &#x3C;br>&#x3C;br> then the complete news content.
- Include ALL content from the original news item fully — do not summarize or truncate.
- Preserve all section sub-headings from the original (e.g., "Host and Venue:", "India's Leadership Role:") as **Sub-heading:**&#x3C;br> inline in the description.
- Preserve all inline markdown: **bold**, *italic*.
- Convert unordered lists to: &#x3C;ul>&#x3C;li>item&#x3C;/li>&#x3C;li>item&#x3C;/li>&#x3C;/ul>
- Convert ordered lists to: &#x3C;ol>&#x3C;li>item&#x3C;/li>&#x3C;li>item&#x3C;/li>&#x3C;/ol>
- Nested lists should use nested &#x3C;ul>/&#x3C;ol> tags accordingly.
- Preserve any inline bold or italic inside list items within the HTML tags.
- (if present) Fields like "Objective:", "Host:", "Frequency:", "Significance:" must be preserved as **Field Name:** value inline in the description.
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
- The output must begin with # DEFENCE EXERCISE and then the table. Nothing else.

---

## INPUT DOCUMENT:
[PASTE RAW MARKDOWN CHAPTER HERE / Document Attached with Chat]
</code></pre>

***

### How the Sample Transforms <a href="#how-the-sample-transforms" id="how-the-sample-transforms"></a>

```
# DEFENCE EXERCISE

| Exercise | Image | Location & Nations | Description |
|----------|-------|--------------------|-------------|
| **IONS Maritime Exercise (IMEX)**<br>*2026* | ![](img_001) | **Indian Ocean Region**<br>*Maritime Warfare Centre, Southern Naval Command, Kochi*<br>**Nations:** India, Bangladesh, France, Indonesia, Kenya, Maldives, Mauritius, Myanmar, Seychelles, Singapore, Sri Lanka, Tanzania, Timor-Leste | **IONS Maritime Exercise (IMEX) 2026: Strengthening Indian Ocean Security**<br><br>The Indian Navy hosted the Table Top Exercise of the Indian Ocean Naval Symposium at the Southern Naval Command in Kochi.<br><br>**Host and Venue:**<br><ul><li>Hosted by the Indian Navy.</li><li>Held at the Maritime Warfare Centre, Southern Naval Command, Kochi.</li></ul>**India's Leadership Role:**<br><ul><li>India has assumed the IONS Chairmanship for the 2026–2028 cycle.</li><li>This chairmanship comes after a gap of sixteen years.</li></ul>**Participating Nations:**<br><ul><li>Representatives from 12 countries attended: Bangladesh, France, Indonesia, Kenya, Maldives, Mauritius, Myanmar, Seychelles, Singapore, Sri Lanka, Tanzania, and Timor-Leste.</li><li>International officers from IOS SAGAR and the Indian Navy also participated.</li></ul> |
```
