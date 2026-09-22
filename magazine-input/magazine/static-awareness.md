# Static Awareness

## Structure

* Follows a 2 Column Layout
* Has a Black Thin Center Division Line for visual bifurcation
* Content Flow: Left Column first , then in the Right Column, followed by pagination to next page using weasyprint's automatic handling.
* INPUT - General Overview:
  * Include Full markdown support with Formatted Text using Bold, Italic, Ordered / Unordered List, Table Data etc.
  * Data would contain static facts about entities classified under respective groups.
  * Groups would contain a Heading Level 1 of Markdown style.
    * For a basic idea understand that, entities shall be classified into groups as follows:
      * INTERNATIONAL ORGANISATION
      * GOVERNMENT ORGANISATION
      * PERSONALITIES
    * Wherein each group would contain several static information like facts and figures:
      * Example: Suppose 'World Bank' classified under the 'INTERNATIONAL ORGANISATION' would be having data like:

```
**World Bank**
- Established - [data]
- Director General - [data]
- HQ - [data]
```

* IMPORTANT NOTE: The Static facts could also be given via a table format if not via unordered list style, so add that compatibility as well. And, for the GROUP TITLE, you shall use big font size in bold. Allow data to flow naturally , do not break content at group level, it is possible for multiple groups to be present on one page, not required for them to always start from new page.

### ASCII Layout

```asciidoc
┌──────────────────────────────────────────────────────────────────┐
│  HEADER: 25mm (First Page Chapter Banner)                        │
│           S T A T I C   A W A R E N E S S                        │
└──────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────┐
│  HEADER: 15mm (Subsequent Pages - Runtime Injected via Jinja)    │
│  Static Awareness                                                │
└──────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────┐
│  ┌──────────────────────────┐  │  ┌──────────────────────────┐   │
│  │  LEFT COLUMN (48%)       │  │  │  RIGHT COLUMN (48%)      │   │
│  │  (Flow First)            │  │  │  (Flow After Left)       │   │
│  │                          │  │  │                          │   │
│  │  # GROUP TITLE           │  │  │  # GROUP TITLE           │   │
│  │  (Large Bold)            │  │  │  (Large Bold)            │   │
│  │                          │  │  │                          │   │
│  │  ## Entity Name          │  │  │  ## Entity Name          │   │
│  │  (Bold)                  │  │  │  (Bold)                  │   │
│  │                          │  │  │                          │   │
│  │  - Fact: [data]          │  │  │  - Fact: [data]          │   │
│  │  - Fact: [data]          │  │  │  - Fact: [data]          │   │
│  │  - Fact: [data]          │  │  │  - Fact: [data]          │   │
│  │                          │  │  │                          │   │
│  │  ## Next Entity          │  │  │  ## Next Entity          │   │
│  │  (Bold)                  │  │  │  (Bold)                  │   │
│  │                          │  │  │                          │   │
│  │  | Header | Header |     │  │  │  | Header | Header |     │   │
│  │  |:---|:---|             │  │  │  |:---|:---|             │   │
│  │  | Cell   | Cell   |     │  │  │  | Cell   | Cell   |     │   │
│  │  | Cell   | Cell   |     │  │  │  | Cell   | Cell   |     │   │
│  │                          │  │  │                          │   │
│  │  ## Next Entity          │  │  │  [Continues...]          │   │
│  │  (Bold)                  │  │  │                          │   │
│  │                          │  │  │                          │   │
│  │  - Fact: [data]          │  │  │                          │   │
│  │  - Fact: [data]          │  │  │                          │   │
│  │                          │  │  │                          │   │
│  │  [Next Group may start   │  │  │                          │   │
│  │   here if space permits] │  │  │                          │   │
│  └──────────────────────────┘  │  └──────────────────────────┘   │
│       ▲                        │                        ▲        │
│       └────────────────────────┴────────────────────────┘        │
│                    CENTER DIVISION (Thin Black Line)             │
└──────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────┐
│  FOOTER: 15mm (Runtime Injected via Jinja)                       │
└──────────────────────────────────────────────────────────────────┘
```
