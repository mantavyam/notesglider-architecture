# Current Affairs

## Structure

* Follows a 2 Column Layout
* Has a Black Thin Center Division Line for visual bifurcation
* Content Flow: Left Column first , then in the Right Column, followed by pagination to next page using weasyprint's automatic handling.
* INPUT - General Overview of Markdown Content Type:
  * Questions (Heading 1)
    * Include Full markdown support with Formatted Text using Bold, Italic, Ordered / Unordered List, Table Data etc.
    * 3 Types of Question Data:
      * Straightforward Questions - Objective Type
      * Statement Analysis from ordered list of statement - Objective Type
      * Table Data Pair Verification - Objective Type
  * Answers (Heading 1)
    * Table Data (2 Column):
      * First Column - Question Number
      * Second Column - Correct Answer Alphabet
  * NOTE: there will not be any internal separation to identify the Question Data, hence no such validation exist during processing, only one classification would exist as described above for Question and Answers respectively by a use of Heading level 1 in markdown style. The Answer data would be presented on a separate dedicated page of it's own, ideally there would be a 100 Questions (could be more or less not a hard limit), hence the answer data table would contain so and so rows as per the questions, you shall ideally present 100 answers on a single page (paginate if more data present) by having 5 columns each of 20 rows (1-20, 21-40, 41-60, 61-80, 81-100) with spacing between each table equally for visual coherence.

### ASCII Layout

```asciidoc
PAGE DIMENSIONS (A4):
┌────────────────────────────────────────┐
│ ◄── 10mm ──►◄────── 190mm ────►◄──► │  ← Left/Right Margins
│                                        │
│  ┌────────────────────────────────┐    │
|  | [CHAPTER TITLE - CENTER]       |    │
|  |  C U R R E N T   A F F A I R S |    │
│  │      HEADER (15mm/25mm)        │    │
│  ├────────────────────────────────┤    │
│  │                                │    │
│  │  ┌──────────┐  │  ┌──────────┐ │    │
│  │  │          │  │  │          │ │    │
│  │  │  LEFT    │  │  │  RIGHT   │ │    │
│  │  │ COLUMN   │  │  │ COLUMN   │ │    │
│  │  │ (~91mm)  │  │  │ (~91mm)  │ │    │
│  │  │          │  │  │          │ │    │
│  │  │ Flows    │  │  │ Flows    │ │    │
│  │  │ First    │  │  │ Second   │ │    │
│  │  │          │  │  │          │ │    │
│  │  └──────────┘  │  └──────────┘ │    │
│  │       ▲        │        ▲      │    │
│  │       └────────┴────────┘      │    │
│  │         CENTER DIVISION        │    │
│  │         (Thin Black Line)      │    │
│  │                                │    │
│  ├────────────────────────────────┤    │
│  │      FOOTER (15mm)             │    │
│  └────────────────────────────────┘    │
│                                        │
└────────────────────────────────────────┘


---
              DEDICATED ANSWER KEY PAGE

┌──────────────────────────────────────────────────────────────────┐
│  HEADER: 25mm (First Page of Answer Section)                     │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                      A N S W E R   K E Y                         │
│  ┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐
│  │ Q.No   │    │ Q.No.  │    │ Q.No   │    │ Q.No.  │    │ Q.No.  │
│  │ Ans    │    │ Ans    │    │ Ans    │    │ Ans    │    │ Ans    │
│  ├────────┤    ├────────┤    ├────────┤    ├────────┤    ├────────┤
│  │ [1] -X │    │ [21]-X │    │ [41]-X │    │ [61]-X │    │ [81]-X │
│  │ [2] -X │    │ [22]-X │    │ [42]-X │    │ [62]-X │    │ [82]-X │
│  │ [3] -X │    │ [23]-X │    │ [43]-X │    │ [63]-X │    │ [83]-X │
│  │  ...   │    │  ...   │    │  ...   │    │  ...   │    │  ...   │
│  │ [20]-X │    │ [40]-X │    │ [60]-X │    │ [80]-X │    │[100]-X │
│  └────────┘    └────────┘    └────────┘    └────────┘    └────────┘
│                                                                  │
│   ↑ 20 rows      ↑ 20 rows      ↑ 20 rows      ↑ 20 rows         |
│                                                                  │
│  [Equal spacing between table columns for visual coherence]      │
│                                                                  │
│  Each table: ~18% width  │  Gap: ~2.5%  │  Total: 5 cols = 100%  │
│                                                                  │
│  [If >100 questions, auto-paginate to next Answer Key page]      │
│  [weasyprint handles overflow automatically]                     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│  FOOTER: 15mm (Runtime Injected via Jinja)                       │
└──────────────────────────────────────────────────────────────────┘
```
