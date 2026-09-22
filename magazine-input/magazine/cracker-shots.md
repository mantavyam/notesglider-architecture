# Cracker Shots

## Structure

* Content Flow: Sequentially from top to bottom
* Content Space contains no columns, exclusively allowed to span full width of available space.
* INPUT - General Overview:
  * Contains Table Data Only:
    * HEADING LEVEL 2 for Description of Table = Append this description as table's header row cell by merging all cells.
    * Table Itself (May contain merged cells, handle them truly by keeping the merged cells intact as in the input, internal cell data may have some markdown formatting like bold, italic etc.)
    * Color = The Header Row and the column header shall have brand colour blue, text of the headers cells must be bold if not already, text color = white , alternate table row shall have slight variation in cell color, use light shades only, leading column shall have a different shade as well, all the borders of table shall have a brand color tint.

### ASCII Layout

```asciidoc
┌──────────────────────────────────────────────────────────────────┐
│  HEADER: 25mm (First Page Chapter Banner)                        │
│              C R A C K E R   S H O T S                           │
└──────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────┐
│  HEADER: 15mm (Subsequent Pages - Runtime Injected via Jinja)    │
│  Cracker Shots                                                   │
└──────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────┐
│  ┌──────────────────────────────────────────────────────────┐    │
│  │  [TABLE HEADER ROW - Merged across all columns]          │    │
│  │  [H2 Description text rendered as single spanning cell]  │    │
│  │  [Background: Brand Blue]  [Text: White, Bold]           │    │
│  ├──────────────┬──────────────┬──────────────┬─────────────┤    │
│  │  [Col 1]     │  [Col 2]     │  [Col 3]     │  [Col N]    │    │
│  │  [Header]    │  [Header]    │  [Header]    │  [Header]   │    │
│  │  [Blue BG]   │  [Blue BG]   │  [Blue BG]   │  [Blue BG]  │    │
│  │  [White]     │  [White]     │  [White]     │  [White]    │    │
│  │  [Bold]      │  [Bold]      │  [Bold]      │  [Bold]     │    │
│  ├──────────────┼──────────────┼──────────────┼─────────────┤    │
│  │  [Cell data] │  [Cell data] │  [Cell data] │  [Cell...]  │    │
│  │  [Leading    │  [Alt row    │  [Alt row    │  [Alt row   │    │
│  │   col shade] │   shade 1]   │   shade 1]   │   shade 1]  │    │
│  ├──────────────┼──────────────┼──────────────┼─────────────┤    │
│  │  [Cell data] │  [Cell data] │  [Cell data] │  [Cell...]  │    │
│  │  [Leading    │  [Alt row    │  [Alt row    │  [Alt row   │    │
│  │   col shade] │   shade 2]   │   shade 2]   │   shade 2]  │    │
│  ├──────────────┼──────────────┼──────────────┼─────────────┤    │
│  │  [Cell data] │  [Cell data] │  [Cell data] │  [Cell...]  │    │
│  │  [Leading    │  [Alt row    │  [Alt row    │  [Alt row   │    │
│  │   col shade] │   shade 1]   │   shade 1]   │   shade 1]  │    │
│  │              │              │              │             │    │
│  │  [... rows continue with alternating shades ...]         │    │
│  │              │              │              │             │    │
│  ├──────────────┴──────────────┴──────────────┴─────────────┤    │
│  │  [MERGED CELL EXAMPLE - colspan across all columns]      │    │
│  │  [When input has merged cells, preserve exact structure] │    │
│  ├──────────────┬─────────────────────────┬─────────────────┤    │
│  │  [Cell]      │  [Merged cell -         │  [Cell]         │    │
│  │              │   colspan=2]            │                 │    │
│  └──────────────┴─────────────────────────┴─────────────────┘    │
│                                                                  │
│  [All borders: Brand color tint]                                 │
│  [Internal cell text: Supports **bold**, *italic* markdown]      │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│  [Tables stack sequentially top-to-bottom, full width]           │
│  [content spans entire available width on the page]              │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  [weasyprint auto-pagination if content exceeds page height]     │
│  [Next page continues with same pattern: H2 → Table → H2 → Table]│
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────┐
│  FOOTER: 15mm (Runtime Injected via Jinja)                       │
└──────────────────────────────────────────────────────────────────┘
```
