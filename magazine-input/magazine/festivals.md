# Festivals

Sample content verified against sources. Here's the full deliverable:travelmedia+2

***

### ASCII Layout <a href="#ascii-layout" id="ascii-layout"></a>

```
┌──────────────────────────────────────────────────────────────────┐
│  CHAPTER BANNER: FESTIVALS                                       │
└──────────────────────────────────────────────────────────────────┘

┌────────────┬──────────────────────────────────┬───────────────────────────┐
│            │  **Ayodhya Parv**                │                           │
│  [IMAGE]   │  *3rd Edition · 2026*            │  Full news description    │
│            │                                  │  with inline HTML lists   │
└────────────┴──────────────────────────────────┴───────────────────────────┘

┌────────────┬──────────────────────────────────┬───────────────────────────┐
│            │  **Baisakhi, Bohag Bihu /        │                           │
│  [IMAGE]   │  Rongali Bihu, Vishu**           │  Full news description    │
│            │  *2026*                          │  with inline HTML lists   │
└────────────┴──────────────────────────────────┴───────────────────────────┘

   Col 1               Col 2                           Col 3
 (image slot)   (festival name(s) + edition)        (full content)
  ~12% width           ~28% width                    ~60% width
```

**Multi-festival name pattern in Col 2:**

```
Single festival  →  **Ayodhya Parv** / *3rd Edition · 2026*
Multi-festival   →  **Baisakhi, Bohag Bihu / Rongali Bihu, Vishu** / *2026*
```

***

### Structuring Prompt <a href="#structuring-prompt" id="structuring-prompt"></a>

````
## SYSTEM:
You are a precise document transformation assistant for a monthly current affairs magazine. You restructure festival and celebration news articles into a strict markdown table format for PDF generation via WeasyPrint. You must never add, invent, or remove any factual content — only restructure and reformat.

---

## USER:

Transform the following FESTIVALS chapter markdown document into the structured format described below.

---

### TARGET FORMAT

The output must be a single markdown document with one table:

# FESTIVALS

| Image | Festival | Description |
|-------|----------|-------------|
| ![](img_001) | **Festival Name(s)**<br>*Edition · Year (if available)* | **Full Headline**<br><br>[full news content] |

---

### TRANSFORMATION RULES

One row = one news headline (one festival event / one celebration news item).

**Column 1 — Image:**
- Use sequential placeholder tokens: ![](img_001), ![](img_002), ![](img_003) … incrementing by 1 for every row in document order.
- Never reuse or skip an index.

**Column 2 — Festival:**
- Line 1: **Festival Name(s)** (bold) — the exact name(s) of the festival(s) as mentioned in the news.
  - If the news covers a single festival, use its name: **Ayodhya Parv**
  - If the news covers multiple festivals (e.g., a greetings / occasion round-up), list ALL festival names separated by a comma and space: **Baisakhi, Bohag Bihu / Rongali Bihu, Vishu**
  - Preserve alternate names using a slash (e.g., Bohag Bihu / Rongali Bihu) exactly as they appear in the source.
  - Do not truncate or paraphrase festival names.
- Line 2: *Edition · Year* (italic) — e.g., *3rd Edition · 2026* or just *2026* if only the year is identifiable. If neither is available, use *—*.
- Separate lines with <br>.

**Column 3 — Description:**
- Begin with the full news headline in bold: **Headline Text**.
- Follow with <br><br> then the complete news content.
- Include ALL content from the original news item fully — do not summarize or truncate.
- Preserve all section sub-headings from the original (e.g., "Baisakhi (Punjab & Haryana):", "Bohag Bihu / Rongali Bihu (Assam):") as **Sub-heading:**<br> inline in the description.
- Preserve all inline markdown: **bold**, *italic*.
- Convert unordered lists to: <ul><li>item</li><li>item</li></ul>
- Convert ordered lists to: <ol><li>item</li><li>item</li></ol>
- Nested lists should use nested <ul>/<ol> tags accordingly.
- Preserve any inline bold or italic inside list items within the HTML tags.
- (if present) Fields like "Duration:", "Organizers:", "Venue:", "Significance:" must be preserved as **Field Name:** value inline in the description.
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
- The output must begin with # FESTIVALS and then the table. Nothing else.

---

## INPUT DOCUMENT:

```markdown
[PASTE RAW MARKDOWN CHAPTER HERE] / Attached with Chat
```
````

***

### How the Sample Transforms <a href="#how-the-sample-transforms" id="how-the-sample-transforms"></a>

```
# FESTIVALS

| Image | Festival | Description |
|-------|----------|-------------|
| ![](img_001) | **Ayodhya Parv**<br>*3rd Edition · 2026* | **Ayodhya Parv 2026: Cultural Festival in New Delhi**<br><br>The Ayodhya Parv 2026 is the third edition of this annual cultural festival, organized at the Indira Gandhi National Centre for the Arts, New Delhi. The event serves as a platform for various Indian states to exhibit their historical and cultural ties to the legacy of Lord Ram.<br><br>**Duration:** 3 Days (April 3 to April 5, 2026).<br>**Organizers:** Organized by Ayodhya Nyas in collaboration with IGNCA.<br><br>The first edition of Ayodhya Parv was held in early 2020 at the Indira Gandhi National Centre for the Arts (IGNCA), New Delhi. It was launched during a pivotal time when the construction of the Ram Temple in Ayodhya was gaining momentum, serving as a cultural curtain-raiser for the city's transformation. |
| ![](img_002) | **Baisakhi, Bohag Bihu / Rongali Bihu, Vishu**<br>*2026* | **President and VP Extend Greetings for Harvest & New Year Festivals**<br><br>President Droupadi Murmu and Vice President C. P. Radhakrishnan extended heartfelt greetings to citizens in India and abroad on the occasion of various harvest festivals and traditional New Years (April 14–15, 2026).<br><br>**Baisakhi (Punjab & Haryana):**<br><ul><li>Celebrated as a prominent harvest festival in Northern India.</li><li>It is a major Sikh harvest festival marking the ripening of Rabi crops.</li><li>Historically, it commemorates the foundation of the Khalsa Panth by Guru Gobind Singh in 1699.</li></ul>**Bohag Bihu / Rongali Bihu (Assam):**<br><ul><li>Marks the beginning of the Assamese New Year.</li><li>Symbolizes hope, renewal, and the joy of the new season.</li><li>It is a time for seeding the land and celebrating the end of the harvest season.</li></ul>**Vishu (Kerala):**<br><ul><li>Marks the Malayali New Year and the astronomical spring equinox.</li><li>It is celebrated to pray for a prosperous year ahead, characterized by the "first sight" (Vishukkani — viewing auspicious items like fruits/flowers first thing in the morning).</li></ul> |
```
