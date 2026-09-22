---
icon: language
---

# Translation System

## SECTION 18 — TRANSLATION SYSTEM

### 18.1 Overview

The translation system uses Google Translate API to produce parallel multilingual versions of any document. The original document is always preserved — translation never overwrites the source.

### 18.2 Initiating Translation

* Teacher clicks "Translate" from the canvas action menu.
* A language selection modal opens showing available target languages.
* Teacher selects one or more target languages.
* Each selected language triggers a separate backend API call to Google Translate.

### 18.3 Translation Output

* For each target language: a new **parallel document version** is created in the database.
* The parallel document is linked to the original via `doc-id` reference.
* Each translated document carries its own `doc-status`, `locale`, and full version history.
* Translation metadata in the original document's `metadata.translations` array is updated: `{ language-code, language-name, doc-id (of the linked translated document) }`.

### 18.4 Language Switcher

* A language switcher component appears in the document canvas header once any translation exists.
* Teacher can toggle between the original and any translated versions in real-time.
* The switcher shows: language name, language code, a flag icon (optional).

### 18.5 Multilingual PPTX

* When exporting to PPTX, Teacher has the option to include translated version(s).
* The backend merges original and selected translated JSON into a single unified `.pptx` file where translated slides are appended after the original slides (or interleaved — to be determined by product owner upon implementation review).

### 18.6 Multilingual PDF (Editor Pipeline)

During the Editor's PDF pipeline, the following options are available per document:

| Option                    | Description                                                                               |
| ------------------------- | ----------------------------------------------------------------------------------------- |
| Single PDF per language   | Distinct separate PDFs: `NEWSLETTER-01-03-26-EN.pdf`, `NEWSLETTER-01-03-26-HI.pdf`        |
| Combined multilingual PDF | Single PDF with all languages in sequence: original language first, then each translation |

These options are configured by the Teacher or Editor on a per-document basis.

### 18.7 Per-Type Translation Toggle

* Translation can be disabled per document type in Teacher settings.
* Configurable toggles: Newsletter (default ON), Compilation (default ON), Magazine (default ON), Mindmap (default OFF — Mindmap translation is disabled by default as it may distort visual node layout).
* The `metadata.translation-disabled-for` array reflects the current toggles.
* Before calling Google Translate API for any document, the backend must check this array and skip if the document type is listed.
