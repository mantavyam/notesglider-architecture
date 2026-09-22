# Books

### ASCII Layout <a href="#ascii-layout" id="ascii-layout"></a>

```
┌──────────────────────────────────────────────────────────────────┐
│  CHAPTER BANNER: BOOKS & AUTHORS                                 │
└──────────────────────────────────────────────────────────────────┘

┌────────────┬──────────────────────┬──────────────────┬──────────────────────────┐
│            │  **Book Title**      │  **Author Name** │                          │
│  [IMAGE]   │  *Subtitle if any*   │  *Role/Title*    │  Full news description   │
│            │                      │                  │  with inline HTML lists  │
└────────────┴──────────────────────┴──────────────────┴──────────────────────────┘
┌────────────┬──────────────────────┬──────────────────┬──────────────────────────┐
│  [IMAGE]   │  **Book Title**      │  **Author Name** │  Full news description   │
└────────────┴──────────────────────┴──────────────────┴──────────────────────────┘

   Col 1          Col 2                  Col 3                 Col 4
 (image slot)  (book identity)       (author info)         (full content)
  ~12% width     ~23% width             ~20% width            ~45% width
```

***

### Structuring Prompt <a href="#structuring-prompt" id="structuring-prompt"></a>

```
## SYSTEM:
You are a precise document transformation assistant for a monthly current affairs magazine. You restructure books and authors news articles into a strict markdown table format for PDF generation via WeasyPrint. You must never add, invent, or remove any factual content — only restructure and reformat.

---

## USER:

Transform the following BOOKS & AUTHORS chapter markdown document into the structured format described below.

---

### TARGET FORMAT

The output must be a single markdown document with one table:

# BOOKS & AUTHORS

| Image | Book Title | Author | Description |
|-------|------------|--------|-------------|
| ![](img_001) | **Book Title**<br>*Subtitle (if any)* | **Author Full Name**<br>*Role / Designation (if mentioned)* | **Full Headline**<br><br>[full news content] |

---

### TRANSFORMATION RULES

One row = one news headline (one book release / one news item).

**Column 1 — Image:**
- Use sequential placeholder tokens: ![](img_001), ![](img_002), ![](img_003) … incrementing by 1 for every row in document order.
- Never reuse or skip an index.

**Column 2 — Book Title:**
- Line 1: **Book Title** (bold) — the exact name of the book as mentioned.
- Line 2: *Subtitle* (italic) — only if explicitly mentioned. Otherwise omit this line entirely.
- Separate lines with <br>.
- If no book title is identifiable, use **—**.

**Column 3 — Author:**
- Line 1: **Author Full Name** (bold).
- Line 2: *Role / Designation / Affiliation* (italic) — e.g., *Rajya Sabha MP*, *IAS Officer*, *Journalist*. Use only what is explicitly stated. If not mentioned, skip it.
- Separate lines with <br>.
- If multiple authors exist, list each on a new line separated by <br>.
- Never mention the name of person who released the book in this column, this column shall only contain the name of author / co-author and nobody else's name.

**Column 4 — Description:**
- Begin with the full news headline in bold: **Headline Text**.
- Follow with <br><br> then the complete news content.
- Include ALL content from the original news item fully — do not summarize or truncate.
- Preserve all inline markdown: **bold**, *italic*.
- Convert unordered lists to: <ul><li>item</li><li>item</li></ul>
- Convert ordered lists to: <ol><li>item</li><li>item</li></ol>
- Nested lists should use nested <ul>/<ol> tags accordingly.
- Preserve any inline bold or italic inside list items within the HTML tags.
- Fields like "Publisher:", "ISBN:", "Foreword by:" must be preserved as **Publisher:** value inline in the description.
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
- The output must begin with # BOOKS & AUTHORS and then the table. Nothing else.

---

INPUT DOCUMENT:
[attached with this chat]
```

***

### How the Sample Transforms <a href="#how-the-sample-transforms" id="how-the-sample-transforms"></a>

```
# BOOKS & AUTHORS

| Image | Book Title | Author | Description |
|-------|------------|--------|-------------|
| ![](img_001) | **Tides of Time** | **Sudha Murty**<br>*Rajya Sabha MP* | **VP C.P. Radhakrishnan Releases "Tides of Time" by Sudha Murty**<br><br>Vice President C.P. Radhakrishnan officially released the book Tides of Time, authored by Rajya Sabha MP Sudha Murty.<br><br>The book defines the Parliament as a living embodiment of a vibrant democracy characterized by four pillars: <ul><li>Dialogue</li><li>Debate</li><li>Dissent</li><li>Discussion</li></ul> |
```

### Book Image Frame

```
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Book Showcase — Karuna Style</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,400;0,600;0,700;1,400;1,600&display=swap" rel="stylesheet">
<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
html { -webkit-font-smoothing: antialiased; }

body {
  min-height: 100dvh;
  background: #ffffff;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 60px;
  padding: 80px 40px;
  font-family: 'Cormorant Garamond', Georgia, serif;
}

:root {
  --bw: 270px;
  --bh: 370px;
  --sp: 40px;
  --spine-top:  #e87878;
  --spine-mid:  #d4608a;
  --spine-bot:  #b03060;
  --page-warm:  #f2ede4;
  --page-line:  #d8d2c8;
  --txt-main:   #111111;
  --txt-sub:    #333333;
  --txt-author: #1a1a1a;
}

/* ── SCENE ── */
.scene {
  perspective: 1600px;
  perspective-origin: 55% 40%;
  width: 700px;
  height: 500px;
  position: relative;
}

/* ══════════════════════════
   REAR BOOK (back cover showing)
══════════════════════════ */
.book-rear {
  position: absolute;
  width:  var(--bw);
  height: var(--bh);
  transform-style: preserve-3d;
  /* Rotated ~225deg so back face points toward viewer, shifted left+down */
  transform:
    translate3d(30px, 30px, -60px)
    rotateX(-3deg)
    rotateY(225deg);
}

.book-rear .face-back {
  outline: 1px solid rgba(0,0,0,0.18);
  position: absolute;
  inset: 0;
  background: linear-gradient(155deg,
    #ffffff 0%, #f5f5f5 20%,
    #eeeeee 45%, #e8e8e8 70%, #e0e0e0 100%);
  border-radius: 0 2px 2px 0;
  overflow: hidden;
  backface-visibility: hidden;
  /* face-back is the BACK face, needs 180deg flip */
  transform: rotateY(180deg) translateZ(calc(var(--sp) / 2));
  display: flex;
  flex-direction: column;
  padding: 24px 20px 20px;
  gap: 10px;
}
.book-rear .face-back::after {
  content:'';
  position:absolute; inset:0;
  background: linear-gradient(135deg, rgba(255,255,255,.18) 0%, rgba(255,255,255,0) 55%);
  pointer-events:none;
}
.back-cover-title {
  font-size: 11px; font-weight: 700; letter-spacing: .18em;
  text-transform: uppercase; color: rgba(20,20,20,.85);
  border-bottom: 1px solid rgba(20,20,20,.2); padding-bottom: 6px;
  position: relative; z-index: 2;
}
.back-cover-lorem {
  font-size: 8.5px; line-height: 1.65; color: rgba(40,40,40,.75);
  position: relative; z-index: 2; flex: 1;
}
.back-cover-barcode {
  display: flex; flex-direction: column; align-items: flex-end;
  position: relative; z-index: 2; margin-top: auto;
}
.barcode-lines {
  display: flex; gap: 1.5px; align-items: flex-end; height: 26px;
}
.barcode-lines span {
  width: 2px; background: rgba(40,40,40,.65); display: inline-block;
}
.back-cover-isbn {
  font-size: 6px; font-family: 'Courier New', monospace;
  color: rgba(40,40,40,.6); margin-top: 3px;
}

/* Front face of rear book — hidden from viewer */
.book-rear .face-front {
  outline: 1px solid rgba(0,0,0,0.18);
  position: absolute;
  inset: 0;
  background: #eeeeee;
  transform: translateZ(calc(var(--sp) / 2));
  backface-visibility: hidden;
  border-radius: 2px 0 0 2px;
}

/* Spine of rear book — on RIGHT side since book is rotated 225deg */
.book-rear .face-spine {
  outline: 1px solid rgba(0,0,0,0.18);
  position: absolute;
  width: var(--sp);
  height: var(--bh);
  /* For rear book (rotated 225deg), spine appears on right */
  right: calc(var(--sp) * -1);
  transform-origin: left center;
  transform: rotateY(90deg);
  background: linear-gradient(180deg, #1a1a1a 0%, #111111 50%, #000000 100%);
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}
.book-rear .face-spine::before {
  content:'';
  position:absolute; inset:0;
  background: repeating-linear-gradient(90deg,
    rgba(0,0,0,0) 0px, rgba(0,0,0,0) 10px,
    rgba(0,0,0,.18) 10px, rgba(0,0,0,.18) 11px,
    rgba(255,255,255,.06) 11px, rgba(255,255,255,.06) 12px);
  pointer-events:none;
}
.book-rear .face-spine .spine-title {
  writing-mode: vertical-rl;
  text-orientation: mixed;
  transform: rotate(0deg);
  color: rgba(255,255,255,.92);
  font-size: 11px; font-weight: 600;
  letter-spacing: .14em; text-transform: uppercase;
  white-space: nowrap; overflow: hidden;
  max-height: 88%; padding: 12px 0;
  position: relative; z-index: 1;
}

/* Pages edge — left side of rear book */
.book-rear .face-pages {
  outline: 1px solid rgba(0,0,0,0.18);
  position: absolute;
  width: var(--sp);
  height: var(--bh);
  left: calc(var(--sp) * -1);
  transform-origin: right center;
  transform: rotateY(-90deg);
  background:
    repeating-linear-gradient(90deg,
      transparent 0px, transparent 2px,
      rgba(160,150,135,.30) 2px, rgba(160,150,135,.30) 3px),
    linear-gradient(90deg,
      #d8d2c6 0%, #f0ebe2 25%, #f8f4ed 50%, #f0ebe2 75%, #d8d2c6 100%);
  box-shadow: inset 2px 0 6px rgba(0,0,0,.10), inset -2px 0 4px rgba(0,0,0,.06);
}

.book-rear .face-top {
  outline: 1px solid rgba(0,0,0,0.18);
  position: absolute;
  width: var(--bw);
  height: var(--sp);
  top: 0;
  transform-origin: top center;
  transform: rotateX(-90deg);
  background:
    repeating-linear-gradient(90deg,
      var(--page-warm) 0px, var(--page-warm) 3px,
      var(--page-line) 3px, var(--page-line) 4px),
    linear-gradient(180deg, rgba(255,255,255,.25) 0%, rgba(0,0,0,.06) 100%);
}

.book-rear .face-bottom {
  outline: 1px solid rgba(0,0,0,0.18);
  position: absolute;
  width: var(--bw);
  height: var(--sp);
  bottom: 0;
  transform-origin: bottom center;
  transform: rotateX(90deg);
  background: linear-gradient(90deg, #222222, #3a3a3a, #333333, #3a3a3a, #222222);
}

/* Rear book individual shadow */
.book-rear-shadow {
  position: absolute;
  /* positioned under rear book */
  bottom: 5px;
  left: 10px;
  width: 280px;
  height: 50px;
  background: radial-gradient(ellipse at 40% 50%,
    rgba(0,0,0,.22) 0%, rgba(0,0,0,.08) 55%, rgba(0,0,0,0) 80%);
  filter: blur(10px);
  pointer-events: none;
}

/* ══════════════════════════
   FRONT BOOK (front cover showing)
══════════════════════════ */
.book-front {
  position: absolute;
  width:  var(--bw);
  height: var(--bh);
  transform-style: preserve-3d;
  transform:
    translate3d(165px, 0px, 40px)
    rotateX(-3deg)
    rotateY(-25deg);
  transition: transform .5s cubic-bezier(.16,1,.3,1);
  cursor: pointer;
}
.book-front:hover {
  transform:
    translate3d(165px, -12px, 40px)
    rotateX(-2deg)
    rotateY(-20deg);
}

/* Front face — primary visible face */
.book-front .face-front {
  outline: 1px solid rgba(0,0,0,0.18);
  position: absolute;
  inset: 0;
  /* translateZ by HALF spine width so front face sits flush */
  transform: translateZ(calc(var(--sp) / 2));
  background: linear-gradient(155deg, #ffffff 0%, #f5f5f5 40%, #eeeeee 70%, #e8e8e8 100%);
  /* spine is on LEFT, so left edge has the binding radius */
  border-radius: 0 2px 2px 0;
  overflow: hidden;
  backface-visibility: hidden;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 22px 22px 0;
}
.book-front .face-front::before {
  content:'';
  position:absolute; inset:0;
  background: linear-gradient(140deg,
    rgba(255,255,255,.10) 0%, rgba(255,255,255,.04) 35%, rgba(255,255,255,0) 60%);
  pointer-events:none; z-index:2;
}

.cover-foreword {
  align-self: flex-end;
  font-size: 7.5px; font-style: italic; letter-spacing: .08em;
  color: rgba(80,80,80,.7); line-height: 1.5; text-align: right;
  margin-bottom: 8px; position: relative; z-index: 3;
}
.cover-foreword strong {
  font-style: normal; font-weight: 700; color: rgba(60,60,60,.9);
  text-transform: uppercase; letter-spacing: .12em;
}
.cover-publisher {
  font-size: 8px; letter-spacing: .22em; text-transform: uppercase;
  color: rgba(80,80,80,.7); align-self: flex-end; margin-bottom: 4px;
  position: relative; z-index: 3;
}
.cover-title {
  font-size: 58px; font-weight: 700;
  color: var(--txt-main); letter-spacing: .01em;
  line-height: 0.95; text-align: center; margin-bottom: 2px;
  position: relative; z-index: 3;
  /* Crosshair overlay on title like reference image */
  text-shadow: 0 1px 0 rgba(0,0,0,.08);
}
/* Crosshair lines layered behind title */
.cover-title-wrap {
  position: relative; width: 100%; display: flex;
  align-items: center; justify-content: center;
  flex-direction: column;
}
.cover-title-wrap::before {
  content: '';
  position: absolute;
  width: 90%; height: 1px;
  background: rgba(42,26,26,.12);
  top: 50%; left: 5%;
  z-index: 1;
}
.cover-title-wrap::after {
  content: '';
  position: absolute;
  width: 1px; height: 90%;
  background: rgba(42,26,26,.12);
  left: 50%; top: 5%;
  z-index: 1;
}
.cover-subtitle {
  font-size: 14px; font-weight: 400; font-style: italic;
  color: var(--txt-sub); text-align: center;
  line-height: 1.3; letter-spacing: .02em;
  position: relative; z-index: 3; margin-top: 4px;
}
.cover-icon {
  font-size: 40px; margin: 16px 0 12px; line-height: 1;
  filter: drop-shadow(0 2px 4px rgba(0,0,0,.2));
  position: relative; z-index: 3;
}
.cover-author {
  font-size: 20px; font-weight: 700;
  color: var(--txt-author); letter-spacing: .10em;
  text-transform: uppercase; text-align: center;
  line-height: 1.25; margin-top: auto;
  position: relative; z-index: 3; padding-bottom: 14px;
}
/* Bottom binding strip */
.cover-bind {
  position:absolute; bottom:0; left:0; right:0; height:10px;
  background: linear-gradient(90deg, #222222, #444444, #222222);
  z-index: 4;
}

/* SPINE — left side, anchored RIGHT edge, rotates LEFT */
.book-front .face-spine {
  outline: 1px solid rgba(0,0,0,0.18);
  position: absolute;
  width: var(--sp);
  height: var(--bh);
  /* Spine is on the LEFT of front cover */
  left: calc(var(--sp) * -1);
  /* Pivot from the RIGHT edge of the spine (touching front face) */
  transform-origin: right center;
  transform: rotateY(-90deg);
  background: linear-gradient(180deg, #1a1a1a 0%, #111111 50%, #000000 100%);
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}
.book-front .face-spine::before {
  content:'';
  position:absolute; inset:0;
  background: repeating-linear-gradient(90deg,
    rgba(0,0,0,0) 0px,  rgba(0,0,0,0) 10px,
    rgba(0,0,0,.20) 10px, rgba(0,0,0,.20) 11px,
    rgba(255,255,255,.07) 11px, rgba(255,255,255,.07) 12px);
  pointer-events:none;
}
.book-front .face-spine::after {
  content:'';
  position:absolute; inset:0;
  background: linear-gradient(90deg, rgba(255,255,255,.18) 0%, rgba(255,255,255,0) 50%);
  pointer-events:none;
}
.book-front .face-spine .spine-title {
  writing-mode: vertical-rl;
  text-orientation: mixed;
  transform: rotate(180deg);
  color: rgba(255,255,255,.92);
  font-size: 11px; font-weight: 600;
  letter-spacing: .14em; text-transform: uppercase;
  white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
  max-height: 88%; padding: 12px 0;
  user-select: none; z-index: 1; position: relative;
}

/* Back face of front book — hidden from viewer */
.book-front .face-back {
  outline: 1px solid rgba(0,0,0,0.18);
  position: absolute;
  inset: 0;
  background: linear-gradient(160deg, #e8e8e8, #d8d8d8, #cccccc);
  transform: rotateY(180deg) translateZ(calc(var(--sp) / 2));
  border-radius: 2px 0 0 2px;
  backface-visibility: hidden;
}

/* Pages edge — RIGHT side of front book */
.book-front .face-pages {
  outline: 1px solid rgba(0,0,0,0.18);
  position: absolute;
  width: var(--sp);
  height: var(--bh);
  right: calc(var(--sp) * -1);
  transform-origin: left center;
  transform: rotateY(90deg);
  background:
    repeating-linear-gradient(90deg,
      transparent 0px, transparent 2px,
      rgba(160,150,135,.30) 2px, rgba(160,150,135,.30) 3px),
    linear-gradient(90deg,
      #d8d2c6 0%, #f0ebe2 25%, #f8f4ed 50%, #f0ebe2 75%, #d8d2c6 100%);
  box-shadow: inset 2px 0 6px rgba(0,0,0,.10), inset -2px 0 4px rgba(0,0,0,.08);
}

.book-front .face-top {
  outline: 1px solid rgba(0,0,0,0.18);
  position: absolute;
  width: var(--bw);
  height: var(--sp);
  top: 0;
  transform-origin: top center;
  transform: rotateX(-90deg);
  background:
    repeating-linear-gradient(90deg,
      var(--page-warm) 0px, var(--page-warm) 3px,
      var(--page-line) 3px, var(--page-line) 4px),
    linear-gradient(180deg, rgba(255,255,255,.28) 0%, rgba(0,0,0,.08) 100%);
  box-shadow: 0 2px 6px rgba(0,0,0,.10);
}

.book-front .face-bottom {
  outline: 1px solid rgba(0,0,0,0.18);
  position: absolute;
  width: var(--bw);
  height: var(--sp);
  bottom: 0;
  transform-origin: bottom center;
  transform: rotateX(90deg);
  background: linear-gradient(90deg, #222222, #3a3a3a, #282828, #202020);
}

/* Front book individual shadow */
.book-front-shadow {
  position: absolute;
  bottom: 5px;
  left: 120px;
  width: 320px;
  height: 55px;
  background: radial-gradient(ellipse at 50% 40%,
    rgba(0,0,0,.25) 0%, rgba(0,0,0,.09) 55%, rgba(0,0,0,0) 80%);
  filter: blur(10px);
  pointer-events: none;
}

/* Combined ambient floor shadow */
.scene-shadow {
  position: absolute;
  bottom: 5px; left: 50%;
  transform: translateX(-50%);
  width: 250px; height: 40px;
  background: radial-gradient(ellipse at 50% 35%,
    rgba(0,0,0,.16) 0%, rgba(0,0,0,.06) 55%, rgba(0,0,0,0) 80%);
  filter: blur(12px);
  pointer-events: none;
}
</style>
</head>
<body>

<div class="scene">

  <!-- ══ REAR BOOK ══ -->
  <div class="book-rear">
    <div class="face-back">
      <div class="back-cover-title">BOOK_NAME · SUBTITLE_TEXT</div>
      <div class="back-cover-lorem">
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco.<br><br>
        Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident deserunt mollit anim id est laborum.<br><br>
        Sed perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium totam rem aperiam eaque ipsa quae ab illo inventore veritatis quasi architecto.
      </div>
      <div class="back-cover-barcode">
        <div class="barcode-lines">
          <span style="height:100%"></span><span style="height:65%"></span><span style="height:100%"></span>
          <span style="height:80%;width:3px"></span><span style="height:100%"></span><span style="height:55%"></span>
          <span style="height:100%"></span><span style="height:85%"></span><span style="height:70%"></span>
          <span style="height:100%"></span><span style="height:45%"></span><span style="height:100%"></span>
          <span style="height:75%"></span><span style="height:100%;width:3px"></span><span style="height:60%"></span>
          <span style="height:100%"></span><span style="height:70%"></span><span style="height:100%"></span>
        </div>
        <div class="back-cover-isbn">ISBN 978-93-XXXXX-XX-X</div>
      </div>
    </div>
    <div class="face-front"></div>
    <div class="face-spine">
      <div class="spine-title">BOOK_NAME · AUTHOR_NAME</div>
    </div>
    <div class="face-pages"></div>
    <div class="face-top"></div>
    <div class="face-bottom"></div>
  </div>

  <!-- ══ FRONT BOOK ══ -->
  <div class="book-front">
    <div class="face-front">
      <div class="cover-publisher">PUBLISHER_NAME</div>
      <div class="cover-foreword">
        "...an outstanding work..."<br>
        <strong>Foreword by FOREWORD_BY</strong>
      </div>
      <div class="cover-title-wrap">
        <div class="cover-title">BOOK_NAME</div>
        <div class="cover-subtitle">SUBTITLE_TEXT</div>
      </div>
      <div class="cover-icon">🔥</div>
      <div class="cover-author">AUTHOR_NAME</div>
      <div class="cover-bind"></div>
    </div>
    <div class="face-spine">
      <div class="spine-title">BOOK_NAME · SUBTITLE_TEXT · AUTHOR_NAME</div>
    </div>
    <div class="face-back"></div>
    <div class="face-pages"></div>
    <div class="face-top"></div>
    <div class="face-bottom"></div>
  </div>

  <div class="book-rear-shadow"></div>
  <div class="book-front-shadow"></div>
  <div class="scene-shadow"></div>
</div>

</body>
</html>
```
