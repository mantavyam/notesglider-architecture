# Ads Banner

## SECTION 20 — AD BANNER MANAGEMENT SYSTEM

### 20.1 Overview

Teachers manage their own ad banners (course promotions, announcements, etc.) which are dynamically injected into their PDF outputs. Ad management is in **Teacher Account Settings → Ad Banners**.

### 20.2 Ad Banner Configuration

Each ad banner has the following fields:

| Field        | Type    | Required | Description                                       |
| ------------ | ------- | -------- | ------------------------------------------------- |
| `image`      | URL     | Yes      | Cloudinary URL of the static ad image             |
| `alt-text`   | String  | No       | Accessibility alt text for the image              |
| `full-page`  | Boolean | No       | If true, ad takes up a full PDF page              |
| `target-url` | URL     | Yes      | Where clicking the ad leads (in interactive PDFs) |
| `caption`    | String  | No       | Text displayed below the ad image                 |

### 20.3 Document Type Assignment

* Each ad banner is assigned to one or more document types: Newsletter, Compilation, Magazine (any combination).
* Assignment is per-banner, not per-document-instance.

### 20.4 Ad Scheduling Controls

Each ad banner has scheduling settings that determine when it appears:

| Schedule Type                      | Description                                                                                   |
| ---------------------------------- | --------------------------------------------------------------------------------------------- |
| **Daily**                          | Ad appears in every daily Newsletter                                                          |
| **Weekly**                         | Ad appears once per weekly Compilation                                                        |
| **Monthly**                        | Ad appears once per monthly Magazine                                                          |
| **Specific festive/notable dates** | Teacher selects specific calendar dates within the current academic year using a date picker  |
| **Custom date range**              | Teacher defines a start date and end date — ad appears in documents created within this range |

> Multiple schedule types can be combined for a single ad banner (e.g. always daily AND also on specific festive dates).

### 20.5 Injection Logic

* During WeasyPrint PDF rendering, the backend:
  1. Checks which ad banners are active for the document's Teacher, document type, and document date.
  2. Injects qualifying ads into the PDF at designated positions in the WeasyPrint template.
  3. Scheduling rules are evaluated server-side — not client-side.
