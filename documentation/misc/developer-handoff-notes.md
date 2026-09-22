---
icon: code
---

# Developer Handoff Notes

## SECTION 32 — DEVELOPER HANDOFF NOTES & DESIGN ASSETS

### 32.1 Design Assets

| Screen/Flow             | Design Status          | Notes                                                                      |
| ----------------------- | ---------------------- | -------------------------------------------------------------------------- |
| Select priority screens | Figma designs provided | Developer implements pixel-perfect                                         |
| Remaining screens       | No Figma               | Developer has full creative freedom within Section 6 principles            |
| Directional reference   | On request             | Product owner provides reference app or competitor screenshot upon request |

> **Rule**: Where Figma is provided — implement pixel-perfect. Where it is not — ShadCN + minimalist principles + WCAG 2.1 AA. No other UI frameworks.

### 32.2 Mandatory Component Choices

| Component            | Library/Source                                                                                                                  | Applies To                                       |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| Rich text editor     | Lexical.dev                                                                                                                     | All document canvas screens                      |
| Mind map canvas      | `mind-elixir-core`                                                                                                              | Mindmap creation/review screens                  |
| Presentation mode    | Reveal.js                                                                                                                       | PPTX presentation mode                           |
| PDF rendering        | WeasyPrint (Python)                                                                                                             | All PDF generation (documents and invoices)      |
| PPTX generation      | **PptxGenJS** (client-side JS, v5) — see §17.1                                                                                  | PPTX export (client-side, no backend round trip) |
| AI Q\&A generation   | OpenRouter (Structured Outputs) — see §22B                                                                                      | Pre-submission and late-add Q\&A flow            |
| UI component library | ShadCN UI or compatible third-party ShadCN registries (e.g. `ui.tripled.work` or as discovered via shoogle.dev MCP exploration) | All general UI components                        |
| Styling framework    | Tailwind CSS                                                                                                                    | Throughout                                       |
| Email automation     | Resend                                                                                                                          | All transactional and notification emails        |

### 32.3 Key Implementation Dependencies (Developer Must Research)

Before implementation, the developer should read the documentation for the [tech-stack.md](../getting-started/tech-stack.md "mention")

### 32.4 JSON5 Comment Handling

All document JSON schemas in this PRD use JSON5-compatible `/* */` comments for annotation. Strip these before runtime JSON parsing using:

* NPM package: `strip-json-comments`
* Or a JSON5 parser library
* These are documentation annotations — never feed unstripped JSON5 to `JSON.parse()`

### 32.6 Build Order — Layered Roadmap (v5)

This roadmap is **layered by dependency**, not by feature wishlist. Each phase produces a working, demoable system; later phases compose on earlier substrate without rework. Skipping a phase or reordering across phase boundaries will force rebuilds. Within a phase, ordering of sub-tasks can be parallelised.

> **Reading guide**: every phase declares (a) what to build, (b) why it must come now (its dependency on prior phases or its role as a substrate for later ones), (c) the PRD sections it implements, and (d) **the tests written before any implementation in that phase** (TDD). The build is structured so the **Newsletter atomic document** path reaches production usability by end of Phase 5 — that is the smallest end-to-end vertical slice the product can ship and bill against. Everything after Phase 5 is incremental extension on a working core.

***

#### Testing Methodology — Test-Driven Development (TDD)

Every phase below follows a strict TDD loop. The agent or developer writes tests first, watches them fail, then writes the minimum implementation to turn them green. No production code is written without a failing test that justifies it.

**The TDD loop applied uniformly across phases**:

1. **Write tests first** based on expected input/output pairs derived from the PRD section being implemented. Do not create mock implementations of functionality that does not yet exist.
2. **Run the tests and confirm they fail.** Do not write implementation code at this stage.
3. **Commit the tests** once you are satisfied with them — they are the executable contract.
4. **Write implementation** that passes the tests without modifying them. Iterate until green.
5. **Commit the implementation** as a separate commit from the tests.

**Test pyramid (applies in every phase)**:

| Layer             | Tooling                                                                        | What it covers                                                                                                                                                       |
| ----------------- | ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Unit**          | Vitest (TS/JS), pytest (Python)                                                | Pure functions, validators, state-machine transitions, schema serialisers, math (SLA timers, billing, token estimation), parsers (PRD JSON5, Lexical tree walkers)   |
| **Integration**   | Vitest + Prisma test DB (Neon branch), pytest + httpx for FastAPI              | DB writes against a throwaway Neon branch, Cloudinary/Drive/OpenRouter mocked at SDK boundary, WeasyPrint fixture HTML → PDF, Yjs op apply → snapshot                |
| **Contract**      | Pact (consumer-driven) for OpenRouter, Razorpay, Drive, Translate, Cloudinary  | Pinned request/response shapes per external API. Failures here mean the external surface changed and we adapt before production breaks                               |
| **e2e**           | Playwright (browser)                                                           | Full role-scoped user flows: Teacher onboarding, Newsletter create → Editor approve → PDF delivered, revision wizard with screenshot, Q\&A wizard, archive + recover |
| **Property/fuzz** | fast-check (TS), hypothesis (Python)                                           | Schema invariants (any Lexical tree round-trips through ref-image extractor; any atomic\_uid passes the format regex; any crop transform is bounded)                 |
| **Snapshot**      | Vitest snapshots, pytest-snapshot                                              | JSON schemas produced from fixtures, WeasyPrint HTML before PDF, PptxGenJS XML, HTML export markup                                                                   |
| **Security**      | RBAC matrix tests per endpoint, HMAC verification tests, input validation fuzz | Every endpoint exercised against every role; archive recovery rejects tampered manifests; SQL injection fuzz on free-text fields                                     |
| **Performance**   | k6 / Locust, Lighthouse CI                                                     | Yjs op throughput, WeasyPrint duration, editor frame budget, page LCP                                                                                                |
| **Accessibility** | axe-core via Playwright                                                        | WCAG 2.1 AA per screen, fails CI on regressions                                                                                                                      |

**CI gates (active from Phase 0)**:

* Unit + integration + contract + snapshot + a11y suites run on every PR. Red = block merge.
* e2e suite runs on every PR against an ephemeral staging stack.
* Coverage threshold per package: 80% line, 70% branch (raised over time, never lowered).
* Mutation testing (Stryker for TS, mutmut for Python) runs nightly on critical packages — billing, SLA state machine, atomic UID, HMAC signing.

**TDD anti-patterns the agent must avoid**:

* Writing tests after implementation ("test-last") — defeats the purpose. The test must fail before any production code exists.
* Mocking the unit under test ("mocking yourself") — only mock at external system boundaries.
* Writing implementation that only satisfies the test fixture and not the PRD invariant — use property tests to catch this.
* Deleting or weakening a test to make CI green — flag the conflict, do not silence the alarm.

***

#### PHASE 0 — Project Foundations

**Why now**: nothing can be built before the repo, deploy target, and secrets boundary exist. Establishing these once prevents per-feature improvisation.

* Monorepo scaffold (Next.js App Router + FastAPI service + shared types). Use the existing starter as base.
* Environment + secrets management (Google Secret Manager). Define `.env.example` covering every secret the PRD references (OpenRouter, Razorpay, Better-Auth, Drive, Cloudinary, Translate, archive HMAC seed).
* CI pipeline (lint, typecheck, unit tests, build). Block merge on red.
* Containerisation for Cloud Run (frontend and backend images).
* Staging and production project separation in GCP from day one.
* **Cross-cutting policy install**: codify §6 UI Design Principles as ESLint/Stylelint rules where possible (no inline styles, ShadCN-only component import paths, tailwind utility lint), and §31.3 a11y as a Playwright + axe-core baseline that every later phase inherits.

**Tests written first (red before any implementation in this phase)**:

* CI smoke test that fails if any required `.env` key is missing on container boot.
* Build pipeline test: `lint && typecheck && test && build` exits 0 on a hello-world commit.
* Container health endpoint contract test: `/healthz` returns 200 with `{ status: "ok", version: <git sha> }`.
* Secret access test: a deliberate attempt to read a Secret Manager key from a service account without binding must be denied (negative test).
* Staging↔production isolation test: a write to staging DB must not appear in production DB (executed as part of CI on every infra-touching PR).

PRD: §3, §6 (cross-cutting), §30, §31.3 (a11y baseline).

#### PHASE 1 — Data Layer & Identity Foundations

**Why now**: schema and identity are the substrate every later phase reads from. Building features before the schema settles forces destructive migrations.

* Full Prisma schema implementing every table in §29, including all **v5 additions** (`qa_items`, `revision_screenshots`, `openrouter_config`, `openrouter_call_log`, `ay_archives`) and all **v5 column additions** to `organisations`, `images`, `sub_members`, `documents`.
* Migration framework + seed scripts. Seed the Super Admin record + a default `openrouter_config` row.
* Tenant isolation pattern: every repository query must take `org_id` as the primary scoping key. Codify this as a Prisma middleware or repo-base-class so it cannot be forgotten.
* **Atomic UID service** (§11A): backend generator, collision check against `atomic_uid_log`, immutability enforcement. Build this now — every document, image, and Q\&A item created later depends on it.
* Better-Auth integration: Google OAuth, Email + Password, Magic Link. Provision flow for the three login role types (Super Admin / Editor / Teacher / Sub-member).
* RBAC middleware — server-side enforcement of the 4 roles + publication scoping. Client-side gating is decorative; the server is the gatekeeper.

**Tests written first**:

* Atomic UID format regex test per entity type (Cat / News / thumb / ref) — covers every example in §11A.2 plus property-based fuzz on `headline[:10]` sanitisation.
* Atomic UID collision test: forced collision against `atomic_uid_log` must trigger regeneration, never return a duplicate.
* Atomic UID immutability test: any attempt to update an existing UID row must be rejected at the repo layer.
* Tenant isolation test: every repository method call without an `org_id` argument must throw `MissingTenantContext` (negative test asserts this for every public repo method).
* RBAC matrix tests: a tabular test (driven by the §4.9 permission matrix) hits every endpoint with every role and asserts allow/deny matches the matrix exactly.
* Schema migration round-trip test: apply all migrations to an empty Neon branch, dump schema, diff against checked-in `schema.prisma` — zero drift.
* Better-Auth callback contract tests (Google OAuth flow, magic-link flow) using stubbed identity provider.

PRD: §4, §5, §11A, §29.

#### PHASE 2 — Tenant Lifecycle

**Why now**: every document, billing event, and notification is scoped to an authorised tenant. Without the lifecycle in place, there is nowhere for documents to live.

* Super Admin dashboard skeleton: Editor management panel, Organisation approval queue, system-wide audit log table, platform metrics card shells.
* Editor onboarding (§5.4).
* Teacher self-registration → Organisation creation → Super Admin approval → Editor mapping → Editor authorisation gate (§5.5).
* Sub-member invite + permission configuration (§21), including the new `permissions.qa` boolean.
* Publication streams (§9A): default `CURRENT-AFFAIRS` auto-created on Org registration, custom publication CRUD with the Teacher↔Super-Admin (and Editor-requested) approval chain.
* Duplicate document prevention constraint (§9A.7) on the `documents` table.
* Super Admin dashboard (§8.0): Editor management panel, Organisation management panel, system-wide activity & audit log, platform metrics. (Billing dashboard panels wait until Phase 13.)

**Tests written first**:

* State-machine test for Org approval: `pending_approval → active` requires both Super Admin approval AND Editor authorisation; any other path rejected.
* State-machine test for Teacher status: `pending → active` requires Editor authorisation after Org approval; orphan transitions blocked.
* Sub-member permission JSON shape test: defaults to `{ create: false, edit: false, submit: false, export: false, qa: false }`; only Teacher can mutate.
* Publication uniqueness test: insert two `doc_type_name` rows with same `(org_id, doc_type_name)` across different publications → constraint violation.
* Publication approval-chain test: Editor-requested publication requires Teacher approval THEN Super Admin approval; either skip rejected.
* Duplicate-document e2e: attempt to create two Newsletters for same `(org_id, publication_id, doc_type_name, document_date)` → second create blocked at API boundary AND DB constraint (defence in depth).
* Sub-member `permissions.qa` test: with `qa = false`, Q\&A wizard endpoint returns 403 even when route is hit directly.
* Playwright e2e: full Teacher signup → SA approves Org → SA maps Editor → Editor authorises → Teacher can land in dashboard.

PRD: §4, §5, §8.0, §9A, §21.

#### PHASE 3 — Storage Substrate

**Why now**: the editor in Phase 4 binds directly to Yjs and Cloudinary. Building these first means the editor work has nothing to mock.

* Cloudinary integration: image upload pipeline, **WebP at upload** (§26.6), `atomic_uid` injection into Cloudinary MIME metadata (§11A.5), `images` table persistence with `crop`, `origin`, `storage_format`.
* Yjs server (`y-websocket` sidecar, Node, on Cloud Run): authentication, document-room provisioning per `documents.id`, write-permission check that respects `is-locked` and reviewer overrides.
* Lexical ↔ Yjs binding library on the client.
* `y-indexeddb` provider on the client (replaces v4 keystroke browser cache; doubles as offline recovery — §24.2).
* **240s Neon snapshot worker** (§24.3): periodically reads canonical Yjs doc state and writes to `documents.content_json`. Also flushes immediately on lock / named-version / close events.
* Named version system (§24.4).
* Google Drive OAuth (Editor and Teacher accounts) + folder bootstrap on Org approval (§25.2).

**Tests written first**:

* Cloudinary upload contract test: uploaded asset URL contains `/f_webp/`; response includes `public_id`; MIME metadata read-back contains the supplied `atomic_uid`.
* Image record persistence test: every Cloudinary upload writes an `images` row with `storage_format = "webp"`, `origin`, and `crop = null` by default.
* Yjs server authorisation test: write op from a Teacher when `is-locked = true` is rejected; Editor write during `review` is accepted.
* Yjs op apply → snapshot test: deterministic doc state after N ops is byte-equal across two replicas.
* 240s snapshot worker test: with fake timers, advance 239s → no Neon write; advance to 240s → exactly one write occurs with the canonical Yjs state.
* IndexedDB offline test (Playwright with `context.setOffline(true)`): edits made offline survive page reload and merge correctly on `setOffline(false)`.
* Named version test: finalising bumps `1.0.0 → 1.1.0` (or `1.x.0 → 1.(x+1).0`); the prior version's `content_json` is immutable (UPDATE blocked).
* Drive folder bootstrap test: after Org approval, calling the Drive client mock returns the expected folder tree per §25.2.

PRD: §24, §25, §26.

#### PHASE 4 — Newsletter Atomic Document Editor

**Why now**: Newsletter is the fundamental atomic unit that every aggregation in Phase 9 derives from. Build it first; build it complete.

* Lexical canvas (§10.3): category and news-item blocks, plus-button additions, clear-field + delete dialogs with "don't ask again", drag-and-drop reordering, sidebar minimap.
* Thumbnail upload slot with **non-destructive crop** UI (§10.8): crop dialog, reset icon, frozen-on-lock behaviour, persistence of the `crop` transform.
* **Brief = Lexical JSON tree** (§10.9): inline image paste flow, async background Cloudinary upload, pending/complete/failed `upload_state` rendering, broken-placeholder block on `Send to Editor` if any image is failed.
* Derived `reference-images` array rebuilt server-side on every save by walking each Brief tree.
* Newsletter document JSON v5 persistence end-to-end (matches §11).
* Metadata sheet (§10.2), YouTube link attachment via custom URL parse (§27).
* Teacher dashboard (§8.1): entry-point cards (grouped by publication when multiple exist), Kanban board, logs table.

**Tests written first**:

* Property test on Brief Lexical tree: any sequence of paste / delete / reorder operations yields a tree the derived-`reference-images` extractor can walk without error.
* Derived `reference-images` extractor: given a fixture Brief tree with 3 inline image nodes in known positions, returns exactly 3 entries in the correct order with matching `atomic_uid` values.
* Crop transform validator: `{x, y, w, h}` bounds — `x + w <= original_width`, `y + h <= original_height`; invalid crops rejected.
* Crop frozen-on-lock invariant: after `is-locked = true`, any attempt to write a new `crop` value on the same image row is rejected.
* Snapshot test: §11 JSON produced from a canonical fixture (3 categories, 5 news items, 2 inline images, 1 crop) matches the committed snapshot byte-for-byte (modulo timestamps).
* Inline paste broken-placeholder test: with Cloudinary upload mocked to fail, the "Send to Editor" action is blocked; error surfaced is the §10.9 wording.
* YouTube URL parse contract test: known URL shapes (watch, share, shortlink) all extract the correct channel/video ID without invoking YouTube API.
* Duplicate-prevention real-time UI test: typing a date in the date picker that has an existing doc surfaces the red indicator within 200ms.
* a11y test on the canvas: axe-core passes on the Lexical canvas, the metadata sheet, the crop dialog, and the duplicate-prevention warning.

PRD: §8.1, §10, §11, §11A, §27.

#### PHASE 5 — Editor Pipeline & First Billable Output

**Why now**: this phase closes the smallest end-to-end loop — Teacher creates a Newsletter, Editor approves, PDF is delivered and billed. After this phase, the product is demoable, sellable, and revenue-capable for the default `CURRENT-AFFAIRS` Newsletter.

* Editor review queue + Kanban (queued / generated / delivered / flagged) (§8.2).
* SLA state machine for Stages 0–3 (§13.3): timers, auto-continuation, `sla-paused`, pipeline event log.
* WeasyPrint PDF generation pipeline (§17.6): branded templates, header/footer injection, page count recording, billable-page-count.
* Stage 1 live co-visibility for the Teacher via Yjs read-only (§13.4).
* Stage 3 delivery: in-app + email notification (always); billing meter increment (always); **Reveal.js auto-persistence** of slide-deck PDF to backend (always, server-side headless print, compressed per §26.5); Drive sync to `YYYY/MMMYY/DAILY/FINAL/` **only if** `organisations.drive_sync_enabled = true` (§25.0).
* Non-payment pipeline block scaffolding (§13.5) — full Razorpay integration arrives in Phase 13; here only the block-and-tooltip logic plus a manual `billing_status` flag.
* Editor dashboard (§8.2): entry-point cards (aggregate across publications and assigned Orgs), Editor Kanban board, queue management, priority notification panel, document metrics dashboard.

**Tests written first**:

* SLA timer arithmetic test: `stage1-sla-deadline = submitted-to-editor-at + 2h`; with `sla-paused = true`, the deadline does not advance regardless of wall time.
* Full state-machine transition table test: every transition in §13.3 Stages 0–3 exercised with valid and invalid preconditions; invalid transitions rejected with structured errors.
* Auto-continuation test (fake timers): no Editor action for 2h on Stage 1 → system auto-triggers PDF generation; pipeline\_event row written with `triggered_by = "system"`.
* WeasyPrint contract test: given a fixture HTML for a known Newsletter JSON, output is a valid PDF whose page count matches the snapshot. Compare rendered PDF text content via `pdftotext` against the snapshot text.
* Billing meter test: PDF delivery increments `billing_cycles.total_pages` by exactly `pdf_outputs.page_count`.
* Drive sync toggle test (two states): with `organisations.drive_sync_enabled = true` and on Stage 3, FINAL PDF appears at the expected `YYYY/MMMYY/DAILY/FINAL/` path in both Teacher and (mapped) Editor Drives. With `drive_sync_enabled = false`, **zero** Drive SDK calls are made and the document remains fully retrievable via the in-app dashboard and signed backend URL.
* Reveal.js auto-persistence test: at `event_type = "delivered"`, the backend produces a slide-deck PDF in the backend store; size after §26.5 compression is smaller than the raw print output. Teacher's local print-to-PDF action (if invoked) does NOT overwrite the backend artefact and is NOT compressed.
* Yjs co-visibility test (Playwright, two browser contexts): Editor types in Stage 1; Teacher's read-only canvas reflects the change within 250ms; Teacher write attempt is rejected with the `is-locked = true` error.
* Non-payment block test: with `billing_status = "overdue"` past grace, `POST /documents/:id/submit` returns 402 with the §13.5 tooltip copy; document creation and PPTX export endpoints still return 200.
* e2e: Teacher submits → Editor approves → PDF delivered → Teacher sees Delivered card → billing cycle `total_pages` increment matches the WeasyPrint page count.

PRD: §8.2, §13, §17.6.

#### PHASE 6 — Revision Flow & Screenshot Wizard

**Why now**: revision is the natural follow-up to delivery in the SLA pipeline. Building it directly after Phase 5 keeps state-machine logic contiguous.

* Stage 4 manual revision queue (§13.3 Stage 4): no SLA, manual approval at every step, distinct visual treatment.
* **Mandatory PDF screenshot wizard** (§13.3.4A): PDF.js iframe viewer, drag-quadrilateral region tool, page-range whole-page capture, "Add more" loop, mandatory ≥1 attachment with reason note.
* `revision_screenshots` table with the 5-per-flag / 1MB-per-PNG hard caps; insert trigger to enforce.
* 30-day purge cron after `revision-history[-1].resolved-at`.
* Editor-side rendering: thumbnail strip, expand-on-click with quadrilateral overlay preserved.

**Tests written first**:

* Screenshot size cap test: a 1.01 MB PNG is rejected client-side (returned error string from the wizard validator) AND a forged direct API call with a 1.01 MB blob is rejected server-side (defence in depth).
* Max-5 enforcement test: 6th insert into `revision_screenshots` for the same `(document_id, revision_id)` violates the insert trigger.
* Min-1 enforcement test: revision submission with zero screenshots returns 422; wizard "Submit" button remains disabled in UI.
* Quadrilateral point validator: arrays with !=4 points, or any point outside the PDF page bounds, are rejected.
* Page-range parser: known good inputs (`"3"`, `"5-7"`, `"3,region"`) parsed correctly; malformed inputs rejected.
* Purge cron test (fake timers): with `purge_after` set to T-1d, the cron deletes the row; with `purge_after` set to T+1d, the row survives.
* Playwright e2e: Teacher opens delivered PDF, drags a quadrilateral on page 3, captures, adds a second whole-page capture for page 5, types reason, submits → revision card with both thumbnails visible in Editor's queue.

PRD: §13.3 Stage 4, §13.3.4A, §29.2 (`revision_screenshots`).

#### PHASE 7 — Client-Side Exports

**Why now**: PptxGenJS, Reveal.js, TXT, and ZIP all consume the Brief Lexical tree finalised in Phase 4. They are zero-dependency on Phase 5/6 backend work and can ship as soon as the editor stabilises.

* **PptxGenJS** client-side PPTX generation (§17.1), including the **shared text-height-estimator utility** (build once, reuse for all text blocks).
* Reveal.js presentation mode (live iframe + new tab + fullscreen) and the print-to-PDF path (§17.2, §17.5).
* TXT export (§17.3).
* ZIP export (images + TXT) (§17.4).
* All exporters walk the Brief Lexical tree and resolve image nodes via the `atomic_uid → cloudinary URL + crop transform` map.

**Tests written first**:

* PptxGenJS snapshot test: a fixture Newsletter JSON produces a `.pptx` whose OOXML structure (slide count, image refs, table dimensions) matches the committed snapshot.
* Text height estimator unit test: given fixture text strings at the configured slide width + font, the estimator returns heights within ±5% of a measured ground-truth set.
* Auto-paginate-table contract test: a table with rows that overflow one slide produces N+1 slides; row count is preserved end-to-end.
* Image resolution test: an inline image node with `atomic_uid` and a non-null `crop` produces a Cloudinary URL containing `c_crop,x_,y_,w_,h_` params.
* Reveal.js HTML snapshot: a fixture JSON produces the expected slide-deck HTML structure.
* TXT export round-trip: text content from a fixture matches `pdftotext`-style flattening of the same JSON.
* ZIP export structure test: archive contains exactly `/images/*` (atomic-UID-named) + a single `.txt` at root.
* Visual regression on Reveal.js HTML render (Playwright + pixelmatch).
* a11y: axe-core sweep on the Reveal.js presentation iframe + the export menu.

PRD: §17.1–§17.5.

#### PHASE 8 — AI Q\&A Service

**Why now**: Q\&A depends on the atomic\_uid system (Phase 1), the canvas + Brief tree (Phase 4), and the post-delivery regen mechanic (Phase 5). All prerequisites are now in place.

* OpenRouter service backend (§22B): platform API key, model resolution chain (per-Org override → default → 4 fallbacks), 15-attempt retry envelope, Structured Outputs schemas as Pydantic models, validation chain.
* Per-Org monthly token cap + per-request cap enforcement; `openrouter_call_log` telemetry; Super Admin telemetry surface.
* Pre-submission Q\&A wizard UI (§10.10): count input, news-item selection grid, per-question type customisation, edit-and-accept workflow.
* `qa` node persistence in `content_json`; `qa_items` table writes; `qa-late-added` and `has-qa` metadata flags.
* PDF and PPTX rendering of Q\&A items (tail-section of slides for PPTX; templated section in WeasyPrint).
* **Q\&A late-add regen** (§13.6): post-Phase-5 delivered docs accept Q\&A, regen PDF as new minor version, replace Drive FINAL, archive old version, deliver to Teacher.
* Sub-member `permissions.qa` gating in §21 and §22B.5.

**Tests written first**:

* JSON Schema validator test per Q\&A type (shapes per §11.1 `qa.items[]`): `subjective.straightforward` `{statement, answer}`; `objective.direct` `{statement, options:{A,B,C,D}, correct_option∈A|B|C|D}`; `objective.statement_analysis` `{topic, statements:[s1,s2,s3], options:{A,B,C,D}, correct_option}`. Valid payloads pass; invalid (missing option key, `correct_option` not present in `options`, `statements.length != 3`, statement\_analysis option value outside the allowed set `{ "1","2","3","1+2","2+3","1+3","All of the above","None" }`, duplicate option values within an item) fail.
* 15-attempt retry chain test: mock OpenRouter returns malformed JSON 3 times → same model retried 3x → first fallback model used → continues through chain. With all 5 models failing 3x, total attempts = 15 and final response is the error envelope.
* Model resolution test: per-Org override `qa_model_override` wins over system `default_model`; null override falls back; per-Org fallback override replaces system list.
* Token cap test: a request whose estimated tokens push `qa_tokens_used_current_cycle` over `qa_monthly_token_cap` is rejected up front with the §22B.4 tooltip copy. Within-cap requests succeed.
* Cap reset test: at billing-cycle start, `qa_tokens_used_current_cycle` resets to 0.
* Statement-analysis business rule: response with !=1 correct option is rejected by validator (retried, not accepted).
* `source_atomic_uids` linkage test: every generated `qa_items` row stores the input news items' UIDs; aggregation join in Phase 10 will use this.
* **Provenance split tests (§22B.3.E step 7)**:
  * `content_json.qa.items[]` lean shape: per item, the persisted JSON contains exactly `{qa_id, source_atomic_uids, type, …type-specific render fields…}` and **nothing else**. A fixture assertion fails if `model_used`, `generated_by_role`, `accepted_at`, `slot_index`, or any other provenance field leaks into items\[].
  * `qa_items` row completeness: for each accepted item, the row carries `qa_id`, `qa_id_seq`, `slot_index`, `source_atomic_uids`, `type`, `payload`, `generated_by_role`, `generated_by_id`, `model_used`, `is_late_added`, `accepted_at`.
  * §11.1 byte-equality: the assembled `content_json.qa` for a canonical 5-slot fixture is byte-equal (modulo timestamps) to the §11.1 reference snapshot.
  * First-acceptance metadata stability: top-level `generated_by_role`, `generated_at`, `model_used` are written on first acceptance and NOT overwritten by a subsequent late-add wizard run by a different actor on a different model. Verified by inspecting `content_json.qa` before and after the late-add.
  * Late-add items\[] replacement: a late-add run by a sub-member appends new items\[] entries (qa\_id seq continues from existing max), top-level fields unchanged. `qa_items` rows for the late-added items carry `is_late_added = true` and the sub-member's `generated_by_id`.
* **Multi-call assembly tests (§22B.3.E)**:
  * Slot-order preservation: a 5-slot wizard run with types `[T1, T2, T1, T3, T2]` dispatches 3 OpenRouter calls (one per type group), and the assembled `qa.items[]` array reflects wizard `slot_index` order (1→5), NOT dispatch order or per-group payload order.
  * `qa_id` sequencing across types: the same 5-slot run produces `qa_id` ending in `001..005` in slot order, regardless of which group's call returned first.
  * Late-add seq continuation: a second wizard run on a document with existing `qa_id_seq = 5` produces new items starting at `006`; no reuse, no gaps.
  * Cross-slot ref-fabrication rejection: a mocked T2 response where one item's `source_refs` mixes refs from two different slots (e.g. `["n1", "n3"]` when slots 2 and 5 supplied disjoint refs) is rejected per §22B.3.C step 5.
  * All-or-nothing across groups: if the T3 call exhausts its 15-attempt retry chain, the entire wizard run aborts — zero rows in `qa_items`, no `qa` node mutation, even though T1 and T2 calls succeeded.
  * Concurrent-acceptance race: two parallel "Accept Q\&A" requests on the same document (Teacher + sub-member) both succeed; the `SELECT FOR UPDATE` on `qa_id_seq` serialises them; second-committer's items append after first-committer's max; no qa\_id collisions; total `qa_item_count` equals the sum of both wizard runs.
* **Identity boundary tests (§22B.3.A / §22B.3.D)** — non-negotiable:
  * Outbound payload assertion: the JSON sent to OpenRouter contains zero occurrences of any `atomic_uid` string. Only `ref` handles (`^n\d+$`) appear. Verified by intercepting the HTTP request body and grep-asserting.
  * Inbound schema strictness: a mocked model response containing an `atomic_uid` or `source_atomic_uids` field is rejected by Pydantic `extra="forbid"`.
  * Ref fabrication rejection: a mocked response with `source_refs: ["n99"]` when the request only sent `n1, n2, n3` is rejected by §22B.3.C step 5 and counts as a validation failure (retry triggered).
  * `qa_id` model-fabrication rejection: a mocked response containing `qa_id` is rejected by schema (`extra="forbid"`); even if it slipped through, the post-processing step would overwrite with the backend-assigned value.
  * `qa_id` sequencing: two acceptances on the same document produce `QA-{doc-id}-001` and `QA-{doc-id}-002` deterministically; unique constraint `(document_id, qa_id)` enforced at the DB layer.
  * Ref→UID swap correctness: post-processing transforms `source_refs: ["n2"]` to `source_atomic_uids: [<uid of news item 2>]` per the request-scoped map.
  * Belt-and-braces check: backend re-validates each resolved `source_atomic_uids` entry against the document's current `News-Items[*].atomic_uid` set before INSERT.
* Late-add regen test (Phase-5 doc fixture): adding Q\&A on a `published` doc bumps `doc_version` minor; a new `pdf_outputs` row appears; Teacher receives notification; Drive FINAL is replaced and prior PDF moves to `_archive/`.
* Late-add billing test: if regen PDF page count is greater than prior `billable-page-count`, the delta is added to current cycle; if equal, no billing change.
* Late-add blocked-after-invoice test: when `documents.is_billed = true`, the "Add Q\&A" route returns 409 with the §13.6 tooltip copy.
* Sub-member without `permissions.qa = true` calling the Q\&A wizard endpoint receives 403.
* PDF + PPTX + HTML render snapshot tests: a fixture doc with a `qa` node produces outputs with a Q\&A tail-section in each format.
* Playwright e2e: Teacher runs wizard pre-submit → selects 3 news items → generates 3 Q\&A → edits one → accepts → submits document. End-to-end Q\&A appears in delivered PDF.

PRD: §10.10, §11 (`qa` node), §13.6, §22B.

#### PHASE 9 — Templates, Ads, HTML Export

**Why now**: templates and ad banners are injected by the WeasyPrint pipeline (Phase 5). Until Phase 5 ships, templates have nothing to inject into. HTML export needs the Lexical tree (Phase 4) and the CDN-URL-to-local-path rewriter (small extension on Phase 3 Cloudinary code).

* Template system (§19): drag-and-drop micro app, scope controls, multiple-templates-and-assignment, header/footer/full-page types, dynamic field placeholders, injection during WeasyPrint generation.
* Ad banner management (§20): library, scheduling, per-doc-type assignment, injection logic.
* HTML export — Editor-only (§17.7): CDN URL replacement with local relative paths, image bundling.

**Tests written first**:

* Template scope resolver test: given a doc with N pages and a template configured `scope_page_application = "range"` with `start = 2, end = 4`, the template is injected on pages 2–4 only.
* Multiple-template precedence test: when two templates target overlapping scopes, ordering rules from §19.5 are applied deterministically.
* Dynamic field substitution test: `{{date}}`, `{{yt_url}}`, `{{page_number}}`, `{{ad_banner}}`, `{{title}}`, `{{subtitle}}` all replaced from the document's known values.
* Ad banner schedule resolver test: with a banner scheduled for "weekdays 09:00–18:00 IST", the resolver returns it for a Tuesday 14:00 IST request and skips it for Sunday.
* Ad banner doc-type targeting test: a banner with `document_types = ["Newsletter"]` is injected into a Newsletter PDF but not a Magazine PDF.
* HTML export contract test: every `https://res.cloudinary.com/...` in the source HTML is replaced with a relative `./images/<filename>` path in the export; the corresponding image file exists in the bundle.
* HTML export atomic UID preservation test: every entity in the source has a `data-uid` attribute in the exported HTML (§11A.6).
* a11y: axe-core sweep on the template builder micro-app, the ad banner manager, and the HTML export viewer.

PRD: §17.7, §19, §20.

#### PHASE 10 — Aggregations & Category Extraction

**Why now**: every aggregation in §14 / §15 / §15A / §15B / §15C reads from atomic Newsletters created via Phase 4. With the atomic source stable and Q\&A persistent (Phase 8), aggregations can correctly merge content + Q\&A.

* Cron triggers via Google Cloud Scheduler for: Compilation, Magazine, Quarterly Collection, Bi-Annual Compendium, Annual Yearbook (§9.2, §9.3, §9.3A–C).
* Manual Editor on-demand override for each.
* Aggregation pipelines (§14, §15, §15A, §15B, §15C): pull from atomic docs within the **same publication**, never cross-publication.
* Category Extraction (§15D) Editor-only flow.
* Q\&A merging across aggregations using `qa_items.source_atomic_uids` as the join key.
* Eligibility rules (§14.3) and publication-scoped naming (§9A.4).

**Tests written first**:

* Aggregation eligibility test (§14.3): only `published` Newsletters within the configured window are included; `draft` / `review` / `archived` are excluded.
* Cross-publication isolation test: a Compilation in publication A must never include atomic docs from publication B (fuzz with multi-publication fixtures).
* Cron schedule test (fake timers): with the trigger configured for "every Friday 18:00 IST", advancing the fake clock past the boundary fires the job exactly once per period.
* Manual override test: Editor-triggered aggregation succeeds outside the cron window; cron is suppressed for the same window after a manual run (no double-aggregation).
* Q\&A merge test: a Compilation that pulls 3 Newsletters each with 2 Q\&A items produces a merged Q\&A section with 6 items in source order; each item still carries its original `source_atomic_uids`.
* Category Extraction test (§15D): selected categories from an aggregation produce an Extraction document whose `extraction_categories` matches the selection and whose content includes only those categories.
* Naming-convention test: aggregation output file names match §14.5, §15.4, §15A.4, §15B.4, §15C.4, §15D.3 exactly.
* Aggregation billing test: `count_in_billing = false` Extractions are excluded from billing cycle totals.
* Playwright e2e: Editor manually triggers a Compilation; resulting PDF appears in Drive at the right path and in Teacher's Kanban.

PRD: §9.2–§9.3D, §14, §15, §15A, §15B, §15C, §15D.

#### PHASE 11 — Translation & Mindmap

**Why now**: translation depends on a stable content schema (Phase 4) + Q\&A node (Phase 8) + aggregated docs (Phase 10) to be useful at every output level. Mindmaps need both the Newsletter source (Phase 4) and the SVG → PDF pipeline (extension of Phase 5).

* Google Translate API integration (§18): parallel document creation, language switcher, per-type translation toggle.
* Translation coverage: Brief Lexical tree (walking text nodes), `qa` node, metadata `language-code`, multilingual PPTX, multilingual PDF.
* Mind Elixir canvas integration (§16): standalone flow + post-Newsletter flow.
* Mindmap SVG → WeasyPrint PDF.
* Mindmap restrictions per publication (§9A.5): atomic-only source.
* Mindmap translation behaviour (§16.5 / §18.7).

**Tests written first**:

* Brief tree translation test: a fixture Brief Lexical tree is translated; text nodes carry translated strings; image nodes pass through unchanged (same `src`, same `atomic_uid`).
* Q\&A node translation test: each `qa.items[].statement` (and `answer` / `options` / `statements` per type) is translated; `qa_id` is preserved (so aggregation back-mapping still works).
* Parallel-doc invariant: translated docs share the source `doc-id` linkage, are marked `is-translated = true` on source, and the source content is never mutated.
* Per-type toggle test: with `translation-disabled-for = ["mindmap"]`, the translation pipeline skips that doc type entirely.
* Multilingual PPTX test: translated source produces a unified `.pptx` with alternating source/translated slide pairs.
* Multilingual PDF test: translated source produces a WeasyPrint PDF that embeds both languages per §18.6.
* Mindmap atomic-only constraint test: attempting to create a Mindmap from a Compilation / Magazine / Yearbook returns 422 with the §9A.5 error.
* Mind Elixir SVG → PDF snapshot test: a fixture mind map renders to a deterministic SVG, which renders to a deterministic PDF page.
* Playwright e2e: Teacher enables translation → switches language → translated content appears in canvas, in delivered PDF, and in PPTX export.

PRD: §16, §18.

#### PHASE 12 — Notifications, Comments, Support

**Why now**: every prior phase has produced events worth notifying about (document delivered, revision flagged, Q\&A late-added, publication request, ticket raised). Building notifications after the event-producing surfaces are stable avoids stub-and-rewire churn.

* Two-tier notification architecture (§22): in-app + email via Resend.
* Complete event-to-notification mapping (§22.2) — instrument every event from Phases 5–11.
* User notification controls (§22.3).
* Document comments thread (§4.11.3) — dispute-resolution channel for Teacher ↔ Editor.
* Support tickets (§22A): full lifecycle, Super Admin ticket dashboard, attachments, auto-close.

**Tests written first**:

* Event-to-notification table-driven test: for every event in §22.2, assert (in-app channel fired, email channel fired or suppressed per priority rule).
* Resend contract test: rendered email subjects + bodies for each template match snapshot copy from the PRD.
* Notification preference test: with a per-event preference set to off, the corresponding event does not produce a notification.
* Ticket lifecycle state-machine test (§22A.4): valid transitions allowed, invalid blocked; auto-close cron closes tickets after the configured idle window.
* Ticket attachment cap test: per §22A constraints, exceeding the attachment count or size cap is rejected.
* Comment immutability test: `PATCH /comments/:id` returns 405; only Super Admin soft-delete is allowed.
* Comment chronological ordering test: `GET /documents/:id/comments` returns comments ordered by `created_at` ASC.
* Playwright e2e: Teacher raises a ticket, Super Admin replies, status transitions through `open → in-progress → resolved`, auto-close fires after retention window.

PRD: §4.11.3, §22, §22A.

#### PHASE 13 — Billing, Compliance, Super Admin Dashboards

**Why now**: billing must observe real document volume, real page counts, real Q\&A late-add billing deltas, and real translation multipliers — all of which exist only after Phases 5, 8, 10, 11. Building billing before these phases means rewriting calculations.

* Razorpay integration (§23.2): platform-managed master account, webhook handler, postpaid and prepaid flows.
* Postpaid billing cycle lifecycle (§23.3) + running totals + retrospective document rules (§23.3A) + billability rules (§23.3B).
* Prepaid + discount commitment (§23.1A): semi-annual, annual, auto-renewal, refund policy.
* Manual invoice generation (§23.4) + invoice archive (§23.5).
* Non-payment access policy (§23.6) — wire the actual enforcement into the placeholder built in Phase 5.
* Super Admin billing dashboard (§23.7).
* Evader detection + compliance enforcement (§23.8): thresholds, grace period for new Orgs, the 4 enforcement actions, compliance dashboard.

**Tests written first**:

* Billing math test: `total_amount = total_pages × (rate_per_page - discount_per_page)` for a fixture cycle; rounding rules per §23 covered.
* Retrospective detection test (§23.3A): a document created with `document_date` outside the current cycle is flagged `is_retrospective = true` and routed per the §23.3A.3 invoice layout rules.
* Q\&A late-add billing test (re-run from Phase 8, now under full Razorpay): page count delta is added to the current cycle, not back-applied to a closed cycle.
* Translation billing test: per-language outputs are billed per §23 rules; ensure no double-billing of the source.
* Razorpay webhook contract test (Pact): valid `payment.captured` payload moves the invoice to `paid`; an unsigned or tampered payload is rejected.
* Discount commitment activation test: a paid prepaid invoice for the annual plan activates `discount_per_page = 0.10` for the commitment window; expiry reverts to default.
* Auto-renewal test (§23.1A): with `auto_renewal = true`, a tokenised charge fires on commitment-end; failure surfaces per §23 error path.
* Non-payment enforcement test: with `billing_status = "overdue"` past grace, `POST /documents/:id/submit` returns 402 — same behaviour as Phase 5 scaffold but now driven by real Razorpay state.
* Evader detection threshold test (§23.8): compliance % below threshold for N cycles triggers the configured enforcement action; grace period for new Orgs is respected.
* Enforcement action state-machine test: `allow / warning / temporarily-suspend / terminate` transitions are valid only along the §23.8 path.
* Playwright e2e: Super Admin generates an invoice, Teacher pays via Razorpay test mode, invoice moves to `paid`, billing block lifts.

PRD: §23.

#### PHASE 14 — Archival, Retention, Recovery

**Why now**: archival depends on every artefact type existing in the system (documents, images, PDFs, PPTX exports, Q\&A, revision screenshots). Recovery depends on archival having shipped at least one cycle.

* Image-only year-end archival (§26.4) for Orgs with `full_ay_archive_enabled = false`.
* **Full-AY archival cron** (§26A.2): single ZIP per AY containing JSON, versions, PDFs, PPTX (when explicitly saved to Drive), images, Q\&A snapshot, revision screenshots.
* HMAC-signed `manifest.json` + `manifest.sig` using Org-scoped HMAC secret (§26A.3).
* **Recovery flow** (§26A.4): structural validate, Org-match authorisation, HMAC verify, hash verify, **refuse on conflict** (§26A.5), image re-upload to Cloudinary preserving atomic\_uid, URL rewire across all content\_json + versions (§26A.6), transactional rollback on any failure.
* Soft delete + tombstone + per-user retention policy (§24.6).
* Audit log + Super Admin deletion flow with mandatory pre-archive to Drive + CSV download.

**Tests written first**:

* HMAC round-trip test: `sign(manifest, secret) → verify(manifest, sig, secret) = true`; flipping a single byte of `manifest.json` makes `verify` return false.
* Org-mismatch rejection test: a ZIP whose `manifest.org_id` differs from the recovering Teacher's `org_id` is rejected with the §26A.4 step-4 error.
* Hash verification test: tampering with any file in the ZIP causes `manifest.files[]` hash mismatch and rejects recovery; an `audit_logs` row is written with HMAC-failure event.
* Conflict refusal test: with even 1 existing document for the target AY, recovery is refused with the §26A.5 message.
* Image re-upload + URL rewire test: a manifest with 100 images is recovered into a fresh Org → 100 new Cloudinary uploads happen → every `content_json` image URL (thumbnail + Brief tree image nodes) is rewritten to the new URL via the `atomic_uid` index.
* Transactional rollback test: a synthetic failure injected at step 9 leaves zero DB rows committed and the temp sandbox purged.
* Archive cron determinism test (fake timers): the cron fires exactly at end-of-grace-month for the Org's configured AY; not before, not after.
* Image-only fallback test: with `full_ay_archive_enabled = false`, the §26.4 image-only flow runs and the full-AY flow is suppressed.
* Retention policy test (§24.6): minimum 2-month floor enforced server-side; deletion configs below it are rejected. Soft-deleted rows hard-delete after the 7-day tombstone.
* Audit log deletion test: the SA deletion endpoint requires the pre-archive Drive upload to succeed before any DB delete is executed.
* Playwright e2e: SA toggles full-AY archive ON → cron runs → ZIP appears in Drive → Teacher uploads it back via recovery → all documents and images return; URLs in the canvas resolve to fresh Cloudinary assets.

PRD: §24.6, §26.4, §26A.

#### PHASE 15 — Cross-Cutting Hardening

**Why now**: hardening must observe the system at full feature-completeness. Performance tuning, error ladders, and accessibility on a half-built surface produces work that gets thrown away.

* Error handling & recovery ladders (§28): image upload failure, Drive sync failure, WeasyPrint failure, storage limit, real-time connection loss, generic API errors.
* Compression pipeline end-to-end (§26.5 / §26.6 / §26.7): PDF compression via `ghostscript`/`pikepdf`, WebP storage, JPEG transcode for render, ZIP DEFLATE level 9, gzip on transport.
* Monitoring (§30, §31.5): Google Cloud Monitoring + Logging + Error Reporting + Trace. Alert policies on SLA breaches, WeasyPrint failure escalation, OpenRouter token-cap hits, Drive quota warnings.
* Performance pass (§31.1): editor frame budget, Yjs operation latency, snapshot worker throughput, WeasyPrint generation duration.
* Security pass (§31.2): RBAC penetration of every endpoint, HMAC verification on archive recovery, secret rotation policy, input validation at every external boundary.
* Accessibility pass (§31.3): WCAG 2.1 AA across all screens.
* Browser support matrix (§31.7) verified.

**Tests written first**:

* Error-ladder unit test per recovery in §28: each failure mode triggers the specified ladder steps; final step (user notification / pause SLA / etc.) is observed.
* PDF compression test: a fixture PDF is compressed; output is smaller; pdftotext content matches the original byte-for-byte.
* WebP transcode-on-render test: WeasyPrint fetches the Cloudinary `f_jpg` variant for embedding; PptxGenJS fetches `f_jpg` (or `f_png` when alpha is present); HTML export fetches the same.
* ZIP DEFLATE level test: ZIP exports use level-9 compression (verify with the central directory inspection).
* Load test (k6): Yjs server handles N concurrent co-edit sessions with op latency under target ms; report regressions in CI.
* Mutation test (Stryker, mutmut) on billing, SLA, atomic UID, HMAC modules: surviving mutants below threshold = pass.
* OWASP ZAP baseline scan against staging: no high-severity findings; medium findings triaged.
* axe-core full-app sweep: zero WCAG 2.1 AA violations across every screen built since Phase 0.
* Browser support matrix test (§31.7): Playwright cross-browser run on Chromium, WebKit, Firefox at the supported versions.

PRD: §28, §30, §31.

#### PHASE 16 — Deployment & Launch

* Staging deploy and full end-to-end UAT covering every flow from §10 through §26A.
* Load test the Yjs server under realistic concurrent-co-edit scenarios.
* Production cutover.
* Day-1 on-call runbook covering: WeasyPrint failures, OpenRouter outages, Razorpay webhook lag, Drive quota exhaustion, Yjs server crash recovery.

**Tests written first**:

* Synthetic monitor scripts (Playwright in scheduled mode): hourly synthetic Teacher login + create + submit + delivered loop; pages on failure.
* Smoke test suite on staging: hits one endpoint per service; passes before promotion to prod.
* Disaster recovery drill: restore the Neon DB from a point-in-time backup into staging and verify the recovery time matches the RTO target.
* Rollback test: deploy v-current → deploy v-next → roll back → app and DB are functional at v-current.
* Runbook validation test: each on-call runbook step is exercised in a chaos-engineering drill (kill the Yjs pod, kill the Razorpay webhook listener, etc.) and the runbook resolves the incident.

PRD: §30, §31.5.

***

#### PRD Coverage Audit (every section mapped to a phase)

This audit confirms the build order touches every section of the v5 PRD. If a future PRD edit adds a section, append it here.

| PRD Section                                                                                         | Phase                                                                                     |
| --------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| §1 Executive Summary                                                                                | n/a (context only)                                                                        |
| §2 Vision & Problem Statement                                                                       | n/a (context only)                                                                        |
| §3 Tech Stack                                                                                       | Phase 0                                                                                   |
| §4 RBAC (incl. §4.11 visibility, comments, audit)                                                   | Phase 1; comments thread → Phase 12                                                       |
| §5 Auth & Onboarding                                                                                | Phases 1, 2                                                                               |
| §6 UI Design Principles                                                                             | Phase 0 (codified as lint rules; enforced across every phase)                             |
| §7 Application Architecture                                                                         | Phases 0, 3                                                                               |
| §8.0 Super Admin Dashboard                                                                          | Phase 2 (skeleton) + Phase 13 (billing panels)                                            |
| §8.1 Teacher Dashboard                                                                              | Phase 4                                                                                   |
| §8.2 Editor Dashboard                                                                               | Phase 5                                                                                   |
| §9 Document Types & Trigger Schedules                                                               | Phase 4 (atomic types); Phase 10 (aggregations)                                           |
| §9A Publication Streams                                                                             | Phase 2                                                                                   |
| §10 Newsletter Creation Flow (incl. §10.8 crop, §10.9 inline ref, §10.10 Q\&A wizard)               | Phase 4 (canvas, crop, inline paste); Phase 8 (Q\&A wizard)                               |
| §11 Newsletter JSON Schema                                                                          | Phase 4                                                                                   |
| §11A Atomic UID System                                                                              | Phase 1                                                                                   |
| §13 Document Pipeline (incl. §13.3.4A revision wizard, §13.4 Yjs co-vis, §13.6 Q\&A late-add regen) | Phase 5 (stages 0–3, §13.4); Phase 6 (Stage 4, §13.3.4A); Phase 8 (§13.6)                 |
| §14 Compilation                                                                                     | Phase 10                                                                                  |
| §15 Magazine                                                                                        | Phase 10                                                                                  |
| §15A Quarterly Collection                                                                           | Phase 10                                                                                  |
| §15B Bi-Annual Compendium                                                                           | Phase 10                                                                                  |
| §15C Annual Yearbook                                                                                | Phase 10                                                                                  |
| §15D Category Extraction                                                                            | Phase 10                                                                                  |
| §16 Mindmap                                                                                         | Phase 11                                                                                  |
| §17.1 PPTX (PptxGenJS)                                                                              | Phase 7                                                                                   |
| §17.2 Reveal.js PDF                                                                                 | Phase 7                                                                                   |
| §17.3 TXT                                                                                           | Phase 7                                                                                   |
| §17.4 ZIP                                                                                           | Phase 7                                                                                   |
| §17.5 Reveal.js Presentation Mode                                                                   | Phase 7                                                                                   |
| §17.6 WeasyPrint PDF Pipeline                                                                       | Phase 5                                                                                   |
| §17.7 HTML Webpage Export                                                                           | Phase 9                                                                                   |
| §18 Translation System                                                                              | Phase 11                                                                                  |
| §19 Template System                                                                                 | Phase 9                                                                                   |
| §20 Ad Banner Management                                                                            | Phase 9                                                                                   |
| §21 Team & Sub-member Management                                                                    | Phase 2                                                                                   |
| §22 Notification System                                                                             | Phase 12                                                                                  |
| §22A Customer Support & Tickets                                                                     | Phase 12                                                                                  |
| §22B OpenRouter Q\&A Service                                                                        | Phase 8                                                                                   |
| §23 Billing & Monetization (incl. §23.8 evader)                                                     | Phase 13                                                                                  |
| §24 Document Storage Architecture (incl. Yjs + 240s)                                                | Phase 3                                                                                   |
| §25 Google Drive Integration                                                                        | Phase 3                                                                                   |
| §26 Cloudinary & Image Lifecycle (incl. WebP)                                                       | Phase 3 (upload + WebP); Phase 14 (image-only AY archival)                                |
| §26A Full-AY Archival & Recovery                                                                    | Phase 14                                                                                  |
| §27 YouTube Video Integration                                                                       | Phase 4                                                                                   |
| §28 Error Handling & Recovery Ladders                                                               | Phase 15                                                                                  |
| §29 Database Schema (incl. §29.2 v5 tables, §29.3 column additions)                                 | Phase 1                                                                                   |
| §30 Deployment Architecture                                                                         | Phases 0, 16                                                                              |
| §31 Non-Functional Requirements                                                                     | Phase 15 (perf, security, a11y, browser); Phase 0 (foundational lint + axe baseline)      |
| §32 Developer Handoff Notes                                                                         | n/a (this section)                                                                        |
| Appendices A–E                                                                                      | Referenced from the phases that consume them (e.g. naming conventions in Phases 3, 5, 10) |

#### Dependency Quick-Reference

| Layer                   | Hard prerequisites                                                    |
| ----------------------- | --------------------------------------------------------------------- |
| Atomic UID service      | Schema (Phase 1)                                                      |
| Cloudinary upload       | Atomic UID (Phase 1), schema (Phase 1)                                |
| Yjs editor binding      | Yjs server, Lexical, IndexedDB (all Phase 3)                          |
| Newsletter Brief tree   | Lexical canvas, Cloudinary upload (Phases 3–4)                        |
| WeasyPrint PDF pipeline | Brief tree finalised (Phase 4)                                        |
| Q\&A wizard             | Atomic UID, Brief tree, post-delivery regen (1, 4, 5)                 |
| Aggregations            | Stable atomic Newsletter + Q\&A (Phases 4, 8)                         |
| Translation             | Stable Brief tree, Q\&A node, aggregations (4, 8, 10)                 |
| Billing                 | Page-count, retrospective, late-add regen, translation (5, 8, 10, 11) |
| Full-AY archive         | Every artefact type exists (everything through Phase 13)              |
| Recovery                | Archive cron has produced at least one ZIP (Phase 14)                 |
| Hardening               | Feature surface complete (Phase 15)                                   |

#### Non-Negotiable Sequencing Rules

1. **Schema before features.** Every v5 column and table from §29.2 / §29.3 must exist before any feature that writes to it.
2. **RBAC server-side, day one.** Never ship a UI feature whose access control lives only in the client.
3. **Atomic UID before any document write.** Documents created without a UID generator in place will need backfill.
4. **Yjs + IndexedDB before the editor goes live.** Building the editor on a temporary autosave shim and swapping in Yjs later is a guaranteed rewrite.
5. **PptxGenJS before any other PPTX work.** No `python-pptx` regression is acceptable in v5.
6. **Q\&A node round-trips through PDF, PPTX, HTML, translation before the wizard ships to production.** A skipped renderer means broken docs.
7. **Archive cron must produce one full cycle in staging before recovery is enabled in production.** Recovery without a verified archive on hand is operationally dangerous.
8. **Tests red before implementation, always.** No production code without a failing test that justifies it. The "Tests written first" list in each phase above is the entry condition for that phase — those tests are committed (red) and CI verifies they fail; only then does implementation begin. PR diffs that introduce production code without a matching test commit are blocked at review.
9. **PRD section coverage is enforced.** Every PRD section listed in a phase's `PRD:` footer is mapped to at least one test in the phase's "Tests written first" list. The mapping is checked by a `pre-merge-coverage` script that reads the PRD section headers and the test file `describe()` blocks; an uncovered section blocks merge.

### 32.7 Definition of Done (Per Feature)

A feature is considered "done" only when:

* **Tests were written first, committed red, then turned green** — the commit history shows the test commit landing before the implementation commit (enforced by review).
* Implementation matches this PRD exactly (or any approved deviation is documented).
* All edge cases specified in this PRD are handled.
* Error states are implemented (not just happy path).
* RBAC enforcement is active server-side (not just client-side hiding).
* Notifications fire correctly for the relevant events.
* Data is correctly persisted to the database.
* Google Drive sync triggers correctly where applicable.
* WCAG 2.1 AA is met for all new UI elements (axe-core green).
* The feature has been tested across the three supported browser environments (§31.7) via Playwright cross-browser.
* Unit + integration + contract + snapshot + a11y suites green; coverage threshold not regressed; mutation tests above the configured floor on critical packages.
* PRD section coverage check (sequencing rule 9) passes — every PRD section the feature implements has at least one named test.
* Feature flag (if applicable) is wired and tested in both states.
* Observability: at least one structured log per significant event, one metric per SLO-relevant operation, one trace span per user-visible action.
