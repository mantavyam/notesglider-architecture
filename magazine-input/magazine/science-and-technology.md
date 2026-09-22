# Science & Technology

### ASCII Layout <a href="#ascii-layout" id="ascii-layout"></a>

```
┌──────────────────────────────────────────────────────────────────┐
│  CHAPTER BANNER: SCIENCE & TECHNOLOGY                            │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────┬──────────────────────────┬────────────┬───────────────────────────┐
│  **First Crewed Lunar**      │  **NASA**                │            │                           │
│  **Flyby in 50+ Years**      │  *Space Agency*          │  [IMAGE]   │  Full news description    │
│  *Space Exploration*         │  *United States*         │            │  with inline HTML lists   │
│                              │                          │            │                           │
└──────────────────────────────┴──────────────────────────┴────────────┴───────────────────────────┘
┌──────────────────────────────┬──────────────────────────┬────────────┬───────────────────────────┐
│  **Breakthrough**            │  **Researchers /         │  [IMAGE]   │  Full news description    │
│  *Domain / Field*            │  **Organisation**        │            │                           │
└──────────────────────────────┴──────────────────────────┴────────────┴───────────────────────────┘

        Col 1                      Col 2               Col 3             Col 4
   (breakthrough fact)        (who achieved it)    (image slot)       (full content)
      ~25% width                  ~20% width          ~12% width         ~43% width
```

**Col 1 articulation guide:**

```
Mission launch          →  **First Crewed Lunar Flyby in 50+ Years** / *Space Exploration*
Medical discovery       →  **First AI-Diagnosed Rare Disease Tool** / *Medical Science*
New species found       →  **New Deep-Sea Species Discovered** / *Marine Biology*
Rocket engine test      →  **Reusable Hypersonic Engine Tested** / *Aerospace Engineering*
Climate record          →  **Hottest Ocean Temperature Ever Recorded** / *Climate Science*
Drug approval           →  **First mRNA Cancer Vaccine Approved** / *Biotechnology*
```

**Col 2 researcher articulation guide:**

```
Named scientists        →  **Dr. Jane Goodall, Dr. M. Krishnan** / *Primatologist, Zoologist*
Space agency            →  **NASA** / *Space Agency* / *United States*
Corporate R&D           →  **DeepMind** / *AI Research Lab* / *Google, United Kingdom*
University research     →  **MIT, Stanford University** / *Research Institution* / *United States*
National effort         →  **ISRO** / *Space Agency* / *India*
Multi-org collaboration →  **NASA, ESA, CSA** / *International Space Agencies*
```

***

### Structuring Prompt <a href="#structuring-prompt" id="structuring-prompt"></a>

<pre><code>## SYSTEM:
You are a precise document transformation assistant for a monthly current affairs magazine. You restructure science and technology news articles into a strict markdown table format for PDF generation via WeasyPrint. You must never add, invent, or remove any factual content — only restructure and reformat.

---

## USER:

Transform the following SCIENCE &#x26; TECHNOLOGY chapter markdown document into the structured format described below.

---

### TARGET FORMAT

The output must be a single markdown document with one table:

# SCIENCE &#x26; TECHNOLOGY

| Breakthrough | Researchers | Image | Description |
|--------------|-------------|-------|-------------|
| **Breakthrough Statement**&#x3C;br>*Domain / Field* | **Name(s) / Organisation(s)**&#x3C;br>*Type of Entity*&#x3C;br>*Country / Region (if mentioned)* | ![](img_001) | **Full Headline**&#x3C;br>&#x3C;br>[full news content] |

---

### TRANSFORMATION RULES

One row = one news headline (one discovery / mission / invention / record / approval).

**Column 1 — Breakthrough:**
- Line 1: **Breakthrough Statement** (bold) — a concise, factual phrase capturing the core achievement. This must be specific and informative, not a copy of the headline. Aim for 4–8 words that describe WHAT was achieved. Examples:
  - "NASA's Artemis II: First crewed lunar flyby in 50+ years" → **First Crewed Lunar Flyby in 50+ Years**
  - "Scientists discover new antibiotic compound" → **New Antibiotic Compound Discovered**
  - "India launches first solar observation satellite" → **First Solar Observation Satellite Launched**
  - "AI model diagnoses cancer with 98% accuracy" → **AI Diagnoses Cancer at 98% Accuracy**
- Line 2: *Domain / Field* (italic) — one concise domain label. Examples: *Space Exploration*, *Biotechnology*, *Artificial Intelligence*, *Marine Biology*, *Quantum Computing*, *Climate Science*, *Aerospace Engineering*, *Nuclear Energy*, *Genetics*, *Robotics*.
- Separate lines with &#x3C;br>.

**Column 2 — Researchers:**
- This column identifies WHO achieved the breakthrough. The researcher or research entity may be:
  - Named individual scientists or researchers → use their full names
  - A space agency, company, or institution → use the organisation name
  - A country or national programme → use the country/programme name
  - A collaboration of multiple entities → list all, separated by &#x3C;br>
- Line 1: **Full Name(s) or Organisation Name(s)** (bold) — list all directly responsible parties. If multiple, use &#x3C;br> between each.
- Line 2: *Type of Entity* (italic) — classify each entity appropriately. Examples: *Space Agency*, *Research Institution*, *Biotechnology Company*, *University*, *Government Body*, *AI Research Lab*, *Defence Organisation*. If multiple entities, separate their types with a comma or use &#x3C;br> to align with Line 1 entries.
- Line 3: *Country / Region* (italic) — the country or region of the primary entity if mentioned. For international collaborations, list all countries separated by commas. If not mentioned, use *—*.
- Separate lines with &#x3C;br>.
- If no researcher, scientist, or organisation is mentioned or inferable, use **Not Specified** in Line 1.

**Column 3 — Image:**
- Use sequential placeholder tokens: ![](img_001), ![](img_002), ![](img_003) … incrementing by 1 for every row in document order.
- Never reuse or skip an index.

**Column 4 — Description:**
- Begin with the full news headline in bold: **Headline Text**.
- Follow with &#x3C;br>&#x3C;br> then the complete news content.
- Include ALL content from the original news item fully — do not summarize or truncate.
- Preserve all section sub-headings from the original (e.g., "Launch Site:", "Key Findings:", "Technical Specifications:") as **Sub-heading:**&#x3C;br> inline in the description.
- Preserve all inline markdown: **bold**, *italic*.
- Convert unordered lists to: &#x3C;ul>&#x3C;li>item&#x3C;/li>&#x3C;li>item&#x3C;/li>&#x3C;/ul>
- Convert ordered lists to: &#x3C;ol>&#x3C;li>item&#x3C;/li>&#x3C;li>item&#x3C;/li>&#x3C;/ol>
- Nested lists should use nested &#x3C;ul>/&#x3C;ol> tags accordingly.
- Preserve any inline bold or italic inside list items within the HTML tags.
- Fields like "Launch Site:", "Rocket &#x26; Capsule:", "Mission Duration:", "Significance:" must be preserved as **Field Name:** value inline in the description.
- Escape any pipe characters inside cells as \|.
- Use &#x3C;br> for paragraph breaks within the cell — never raw newlines.

**Date information:**
- Dates (## DD-MM-YY) from the original document must not appear anywhere in the output.
- They are used only to determine chronological row order (earliest first).
- No date headings, no date dividers, no date column anywhere in the output.

**Row ordering:**
- Rows must appear in the same chronological order as the input (earliest ## DD-MM-YY date first, within a date in original top-to-bottom order).

**Formatting hygiene:**
- Escape any pipe characters | inside table cells as \|.
- Use &#x3C;br> for all line breaks inside cells — never raw newlines.
- Do not add any commentary, preamble, or explanation outside the markdown.
- The output must begin with # SCIENCE &#x26; TECHNOLOGY and then the table. Nothing else.
<strong>- If the news content contains tabular data (e.g., mission timelines, comparison tables, ranked lists with multiple columns, structured specifications), do NOT embed it inside the description cell. Instead, close the current news table row normally, then place the accompanying table as a standalone markdown table immediately below that row, then begin a new '| Breakthrough | Researchers | Image | Description |' table with fresh headers to continue with the next news item. The pattern must be:
</strong>```  
[News Row in main table]
[Standalone accompanying table — (only if tabular data present, else avoid)]
[New main table with headers restarted for next news item]
```
---

## INPUT DOCUMENT:

```markdown
[PASTE RAW MARKDOWN CHAPTER HERE] / Attached with Chat
```
</code></pre>

***

### How the Sample Transforms <a href="#how-the-sample-transforms" id="how-the-sample-transforms"></a>

```
# SCIENCE & TECHNOLOGY

| Breakthrough | Researchers | Image | Description |
|--------------|-------------|-------|-------------|
| **First Crewed Lunar Flyby in 50+ Years**<br>*Space Exploration* | **NASA**<br>*Space Agency*<br>*United States* | ![](img_001) | **Artemis II: Humanity's Historic Return to Lunar Orbit**<br><br>NASA has successfully launched the Artemis II mission, marking the first crewed voyage to the moon in over 50 years. The four-member crew is currently orbiting Earth in the Orion capsule before they begin their "out-and-back" journey around the moon.<br><br><ul><li>**Launch Site:** Kennedy Space Centre, Cape Canaveral, Florida, U.S.</li><li>**Rocket & Capsule:** The Space Launch System rocket carried the Orion crew capsule (named Integrity).</li></ul> |
```
