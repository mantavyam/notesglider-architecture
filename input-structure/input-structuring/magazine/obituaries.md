# Obituaries

### ASCII Layout <a href="#ascii-layout" id="ascii-layout"></a>

```
┌──────────────────────────────────────────────────────────────────┐
│  CHAPTER BANNER: OBITUARIES                                      │
└──────────────────────────────────────────────────────────────────┘

┌────────────┬──────────────────────────────┬───────────────────────────────────┐
│            │  **Jim Whittaker**           │                                   │
│  [IMAGE]   │  *Mountaineer*               │  Full news description            │
│            │  *1929 – 2026*               │  with inline HTML lists           │
│            │                              │                                   │
└────────────┴──────────────────────────────┴───────────────────────────────────┘
┌────────────┬──────────────────────────────┬───────────────────────────────────┐
│  [IMAGE]   │  **Person Name**             │  Full news description            │
│            │  *Generalised Profession*    │                                   │
│            │  *YYYY – YYYY*               │                                   │
└────────────┴──────────────────────────────┴───────────────────────────────────┘

   Col 1              Col 2                          Col 3
 (image slot)   (name + profession + lifespan)     (full content)
  ~12% width          ~23% width                    ~65% width
```

**Col 2 profession generalisation examples:**

```
textFirst American to summit Everest, CEO of REI  →  Mountaineer
District Collector, IAS Officer               →  Civil Servant
Playback Singer, Film Music Composer          →  Musician
Test Cricketer, Cricket Coach                 →  Cricketer
Supreme Court Judge, Chief Justice            →  Jurist
Research Scientist, ISRO Rocket Engineer      →  Scientist
Kannada Film Actor, Theatre Artist            →  Actor
```

***

### Structuring Prompt <a href="#structuring-prompt" id="structuring-prompt"></a>

<pre><code>## SYSTEM:
You are a precise document transformation assistant for a monthly current affairs magazine. You restructure obituary news articles into a strict markdown table format for PDF generation via WeasyPrint. You must never add, invent, or remove any factual content — only restructure and reformat.

---

<strong>## USER:
</strong>
Transform the following OBITUARIES chapter markdown document into the structured format described below.

---

### TARGET FORMAT

The output must be a single markdown document with one table:

# OBITUARIES

| Image | Person | Description |
|-------|--------|-------------|
| ![](img_001) | **Full Name**&#x3C;br>*Generalised Profession*&#x3C;br>*Birth Year – Death Year* | **Full Headline**&#x3C;br>&#x3C;br>[full news content] |

---

### TRANSFORMATION RULES

One row = one news headline (one obituary / one person).

**Column 1 — Image:**
- Use sequential placeholder tokens: ![](img_001), ![](img_002), ![](img_003) … incrementing by 1 for every row in document order.
- Never reuse or skip an index.

**Column 2 — Person:**
- Line 1: **Full Name** (bold) — the exact full name of the deceased as mentioned in the news.
- Line 2: *Generalised Profession* (italic) — a single, concise, universally recognisable profession label that best reflects the domain in which the person made their name. Use the broadest accurate label, not a job title. Examples:
  - "First American to summit Everest, former CEO of REI" → *Mountaineer*
  - "Playback singer, music composer" → *Musician*
  - "Test cricketer, BCCI selector" → *Cricketer*
  - "Supreme Court judge" → *Jurist*
  - "IAS officer, District Collector" → *Civil Servant*
  - "Kannada film actor, theatre artist" → *Actor*
  - "Research scientist at ISRO" → *Scientist*
  - "Field Marshal, Army Chief" → *Military Officer*
  - "Governor, Chief Minister" → *Politician*
  - "Novelist, poet" → *Writer*
  - Use only one label. If two domains are equally defining (e.g., scientist-politician), choose the one most prominently associated with their legacy as described in the news.
- Line 3 (inferenced lifespan, skip if not given in data): *Birth Year – Death Year* (italic) — extract both years (if mentioned). If birth year is not mentioned, use *? – Death Year*. If death year is not mentioned but can be inferred from the date context, use it. If neither is available, use *—*.
- Separate lines with &#x3C;br>.

**Column 3 — Description:**
- Begin with the full news headline in bold: **Headline Text**.
- Follow with &#x3C;br>&#x3C;br> then the complete news content.
- Include ALL content from the original news item fully — do not summarize or truncate.
- Preserve all inline markdown: **bold**, *italic*.
- Convert unordered lists to: &#x3C;ul>&#x3C;li>item&#x3C;/li>&#x3C;li>item&#x3C;/li>&#x3C;/ul>
- Convert ordered lists to: &#x3C;ol>&#x3C;li>item&#x3C;/li>&#x3C;li>item&#x3C;/li>&#x3C;/ol>
- Nested lists should use nested &#x3C;ul>/&#x3C;ol> tags accordingly.
- Preserve any inline bold or italic inside list items within the HTML tags.
- (if present) Fields like "The Milestone:", "Notable Work:", "Survived By:", "Awards:" must be preserved as **Field Name:** value inline in the description.
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
- The output must begin with # OBITUARIES and then the table. Nothing else.

---

## INPUT DOCUMENT:

```markdown
[PASTE RAW MARKDOWN CHAPTER HERE] / Attached with Chat
```
</code></pre>

***

### How the Sample Transforms <a href="#how-the-sample-transforms" id="how-the-sample-transforms"></a>

```
text# OBITUARIES

| Image | Person | Description |
|-------|--------|-------------|
| ![](img_001) | **Jim Whittaker**<br>*Mountaineer*<br>*1929 – 2026* | **First American to Summit Everest Jim Whittaker Passed Away**<br><br>Jim Whittaker, the pioneering mountaineer who became the first American to reach the summit of Mount Everest, has passed away. A legendary figure in the outdoor world, he was also the first full-time employee and former CEO of Recreational Equipment, Inc.<br><br><ul><li>**The Milestone:** On May 1, 1963, Jim Whittaker became the first American to summit Mount Everest (8,848 meters).</li><li>He reached the top alongside Sherpa Nawang Gombu.</li></ul> |
```
