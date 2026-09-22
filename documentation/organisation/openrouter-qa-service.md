---
icon: robot
---

# OpenRouter Q\&A Service

## SECTION 22B — OPENROUTER Q\&A SERVICE

### 22B.1 Overview

The Q\&A generation feature (§10.10) is powered by AI models accessed through the **OpenRouter** aggregator API. OpenRouter exposes a unified OpenAI-compatible REST surface across many model providers, including OpenAI's **Structured Outputs** spec.

The Notesglider backend:

* Holds a single platform-wide OpenRouter API key.
* Routes every Q\&A request through this key.
* Resolves the model to call per a Super-Admin-configurable resolution chain (§22B.2).
* Enforces spend guardrails per Organisation (§22B.4).
* Returns Structured Outputs validated against the JSON Schema for the requested question type.

#### Why Structured Outputs

> _"Structured Outputs is a feature that ensures the model will always generate responses that adhere to your supplied JSON Schema, so you don't need to worry about the model omitting a required key, or hallucinating an invalid enum value."_

Benefits we rely on:

* **Reliable type-safety** — the response shape matches our schema; no defensive parsing of free-form prose.
* **Explicit refusals** — safety-based model refusals are programmatically detectable (handled via the §22B.3 refusal branch, not the validation chain).
* **Simpler prompting** — no need for strongly-worded "RESPOND IN JSON ONLY" instructions; the schema is the contract.

Schemas are defined **once** as **Pydantic** models on the FastAPI side (used both for outbound JSON Schema generation against the OpenRouter request and for inbound validation). The TypeScript frontend uses **Zod** mirrors of the same shapes for wizard-side edit validation.

> **Note on the inspirational reference prompt**: legacy markdown-driven prompt patterns exist in industry write-ups (which use `#`/`##`/`###` headings to encode document structure). **Our project does NOT use markdown input** for the prompt. Notesglider passes news items as a structured JSON payload extracted directly from the canonical Lexical tree (ephemeral `ref` handle, headline, category, Brief text-flattened). **atomic\_uid values are stripped server-side before payload assembly** and never reach the model — see §22B.3.A "Identity Boundary".

### 22B.2 Model Resolution Chain

Only the **Super Admin** can configure models. No other role has any visibility into model configuration.

| Setting                      | Scope            | Description                                                                                                                                                   |
| ---------------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `default_model`              | System-wide      | The default model used for all Orgs unless overridden.                                                                                                        |
| `fallback_models`            | System-wide      | Ordered array of exactly **4 models**. Used if the primary fails (deprecated, removed, validation failure after retries — see §22B.3).                        |
| `per_org_override_model`     | Per Organisation | Optional override for a single Org. When set, replaces `default_model` for that Org only. Falls through to the same system-wide `fallback_models` on failure. |
| `per_org_override_fallbacks` | Per Organisation | Optional. Replaces system-wide `fallback_models` for that Org only.                                                                                           |

Resolution order at request time:

1. `per_org_override_model` if set, else `default_model`.
2. On failure → iterate the resolved fallback list (per-org override if set, else system-wide).

### 22B.3 Request Lifecycle

1. **Receive request** from Q\&A wizard with: number of questions `N`, per-question type spec, selected news-item atomic\_uids (resolved server-side from the wizard's grid selection).
2. **Prompt assembly**:
   * Backend builds the request-scoped `ref → atomic_uid` map (`{ "n1": <uid_1>, "n2": <uid_2>, … }`) held in memory for this single OpenRouter call.
   * Backend constructs the payload with **only `ref` values**, headlines, categories, and flattened Brief text (Lexical text-only walk; images and formatting stripped). atomic\_uid values are stripped out before the payload leaves the backend boundary.
   * Backend appends the per-type prompt template (§22B.3.A) appropriate to the question type. Mixed-type requests dispatch one call per type group.
3. **JSON Schema attachment**: per the OpenAI Structured Outputs spec, the request includes the JSON Schema for the exact response shape. Schemas (one per question type variant) are defined in code as Pydantic models (server side) and JSON Schema (sent to OpenRouter).
4. **Model dispatch**: call the resolved primary model.
5. **Validate response**:
   * Parse JSON (Structured Outputs guarantees JSON, but parse for safety).
   * Run JSON Schema validation against the canonical shape for the requested type (per §11.1 `qa.items[]`):
     * `subjective.straightforward`: `{ statement: string, answer: string }`.
     * `objective.direct`: `{ statement: string, options: { A, B, C, D: string }, correct_option: "A"|"B"|"C"|"D" }`. All four option keys required.
     * `objective.statement_analysis`: `{ topic: string, statements: [string, string, string] (exactly 3), options: { A, B, C, D: string }, correct_option: "A"|"B"|"C"|"D" }`.
   * Run business-rule validation:
     * `correct_option` must reference a key actually present in `options`.
     * For `objective.statement_analysis`: each of the 4 `options` values must be drawn (without repetition) from the **allowed set** `{ "1", "2", "3", "1+2", "2+3", "1+3", "All of the above", "None" }`.
     * `statements` array length must be exactly 3.
     * Exactly one `correct_option` per item — multi-correct or zero-correct payloads are rejected.
6. **Retry chain**:
   * Same model → up to **3 retries** on validation failure or transport error.
   * If still failing → advance to the next model in the fallback list, restart at retry 0.
   * 4 fallback models × up to 3 retries each + 1 primary × 3 retries = **15 attempts maximum** per request.
7. **Return to client**: validated Q\&A items.
8. **On total failure** (all 15 attempts exhausted): return error response. Wizard surfaces _"Q\&A generation failed after retries. Please try again later, or skip Q\&A for this document."_

#### 22B.3.A Per-Type Prompt Templates

Each question type uses its **own dedicated prompt**. The selected news items are serialised as a structured JSON payload (not markdown) and appended after the prompt. Inspirational interrogative styles are listed inline so the model adopts the right voice; strict rules eliminate prose padding.

#### Identity Boundary — atomic\_uid Never Reaches the Model

This is the **non-negotiable rule** for identity handling in Q\&A generation:

| Identifier                      | Who generates it                                                    | Does the model ever see it? | Does the model ever return it? |
| ------------------------------- | ------------------------------------------------------------------- | --------------------------- | ------------------------------ |
| `atomic_uid` (news items, etc.) | Backend only, collision-checked against `atomic_uid_log` (§11A.3)   | **No**                      | **No**                         |
| `qa_id` (QA item)               | Backend only, sequential per document (`QA-{doc-id}-{3-digit seq}`) | **No**                      | **No**                         |
| `ref` (positional handle)       | Backend, request-scoped, ephemeral                                  | **Yes** (in input)          | **Yes** (in `source_refs`)     |

**Why**: atomic\_uids are immutable identity primitives that must never be fabricated. Allowing the model to echo an atomic\_uid back exposes us to hallucinated or malformed UIDs that would either pollute the audit chain (§11A) or fail the `atomic_uid_log` lookup. The model is scoped strictly to **content generation** for the three question types. Identity is **always** backend-assigned and backend-resolved.

The model speaks only in `ref` handles — short opaque tokens (`n1`, `n2`, …) scoped to a single request. After validation, the backend swaps `source_refs[]` → `source_atomic_uids[]` via the request-scoped lookup map (§22B.3.D), then assigns the QA item's own `qa_id`.

**Common payload structure** sent to the model alongside every prompt:

```json
{
  "news_items": [
    {
      "ref": "n1",
      "category": "SCIENCE-&-TECHNOLOGY",
      "headline": "India Launches Its Most Powerful AI Supercomputer",
      "brief_text": "Flattened plain-text walk of the Brief Lexical tree. Image nodes omitted. Inline links preserved as their visible text only."
    },
    {
      "ref": "n2",
      "category": "GEOPOLITICS",
      "headline": "...",
      "brief_text": "..."
    }
  ],
  "document_date": "2026-03-01",
  "audience_hint": "competitive-exam aspirants reading current-affairs newsletters"
}
```

> **Note**: `ref` values are simple positional handles (`n1`, `n2`, `n3`, …) assigned by the backend at request assembly time. They have **no meaning outside this single OpenRouter call** and are discarded after post-processing. The backend keeps a private in-memory map `{ "n1": <atomic_uid>, "n2": <atomic_uid>, … }` for the duration of the request.

**Template T1 — `subjective.straightforward`**

```
ROLE: You generate single-sentence factual Q&A items for current-affairs assessment.

TASK: For each item in the `news_items` payload selected for question generation, produce ONE question and ONE answer.

QUESTIONNAIRE LEVEL: Challenging enough to confirm careful reading. Students should not be able to answer correctly from the headline alone — the answer must require attention to the Brief text.

INTERROGATIVE STYLE (pick the one that fits best per item):
  - Who…? / When…? / What…? / Which…? / Where…?
  - According to {source named in the Brief}, {who/what/when/which/where}…?
  - Under which {Section/Regulation/Act/Initiative} … , {who/what/when/which/where}…?
  - Which {initiative/program/policy} was launched to address …?

STRICT RULES:
  1. Each question is a single sentence. No multi-sentence stems.
  2. Each answer is a direct factual phrase: a name, a date, a place, a number, or a specific term.
  3. No preambles in the answer ("According to the newsletter…", "As mentioned…").
  4. If the natural answer exceeds 12 words, return only the most essential portion.
  5. Integrity Always Verified - Each generated item must reference back to the source news_item via `source_refs` (an array of `ref` values copied **verbatim** from the input payload's `news_items[].ref`). NEVER fabricate, modify, or invent a `ref` value.
  6. No two items may share the same `source_refs` set unless explicitly requested.

OUTPUT SHAPE: Conform exactly to the attached JSON Schema for `subjective.straightforward`. See §11.1.
```

**Template T2 — `objective.direct`**

```
ROLE: You generate four-option multiple-choice questions for current-affairs assessment.

TASK: For each selected news_item, produce ONE MCQ with exactly four options keyed A–D and exactly one correct option.

QUESTIONNAIRE LEVEL: Distractors (the three wrong options) must be plausible domain-adjacent values, not absurd. The correct option must be directly verifiable from the Brief text.

INTERROGATIVE STYLE: Use the same style guide as Template T1 for the question stem.

STRICT RULES:
  1. The stem is a single sentence ending in a question mark.
  2. Exactly four options. Keys MUST be the literal strings "A", "B", "C", "D".
  3. Option values are short phrases — never full sentences, never explanations.
  4. Exactly one option is correct. `correct_option` is one of "A" | "B" | "C" | "D".
  5. Do not include "All of the above" or "None of the above" as values in this template (those belong to Template T3).
  6. Do not reveal the correct answer in the stem.
  7. Reference the source news_item via `source_refs` (an array of `ref` values copied verbatim from the input payload). NEVER fabricate a `ref`.

OUTPUT SHAPE: Conform exactly to the attached JSON Schema for `objective.direct`. See §11.1.
```

**Template T3 — `objective.statement_analysis`**

```
ROLE: You generate statement-analysis questions for current-affairs assessment.

TASK: For each selected news_item, produce ONE item containing:
  - `topic`: a short noun phrase identifying what the statements are about.
  - `statements`: exactly three short, declarative statements derived from the Brief. A mix of true and false (at least one of each) is required.
  - `options`: exactly four entries keyed A–D. Each VALUE must be drawn (without repetition) from the allowed set:
       { "1", "2", "3", "1+2", "2+3", "1+3", "All of the above", "None" }
  - `correct_option`: one of "A" | "B" | "C" | "D".

DISPLAY FORM (assembled by backend, not by the model):
  "Consider the following statements about {topic}:
    1. {statements[0]}
    2. {statements[1]}
    3. {statements[2]}
   Which of the above statements is/are true?"

STRICT RULES:
  1. Exactly three statements. Each a single declarative sentence, ≤ 20 words.
  2. Exactly four options. No duplicate values.
  3. The truth set encoded by `correct_option`'s value must match the actual truth of the statements you wrote (the validator will not check semantic truth, but a clearly inconsistent pairing reduces quality; produce internally consistent items).
  4. Reference the source news_item via `source_refs` (an array of `ref` values copied verbatim from the input payload). NEVER fabricate a `ref`.

OUTPUT SHAPE: Conform exactly to the attached JSON Schema for `objective.statement_analysis`. See §11.1.
```

When the wizard request mixes types across the N requested questions, the backend dispatches a **separate** generation call per type group (T1, T2, T3) in a sequence one by one — never a single call mixing schemas — and merges the validated items in wizard order before returning, until then shows a UI level feedback of 'loading state' for enhanced UX.

#### 22B.3.B Sample Structured Outputs

These are the exact response shapes the model must return for a single-item generation. Multi-item requests return an array of these.

**Sample for `subjective.straightforward`** (Template T1):

```json
{
  "items": [
    {
      "source_refs": ["n1"],
      "statement": "What is the peak performance of India's new AI supercomputer?",
      "answer": "210 petaflops"
    }
  ]
}
```

**Sample for `objective.direct`** (Template T2):

```json
{
  "items": [
    {
      "source_refs": ["n2"],
      "statement": "Which planet's moon is NASA's Europa Clipper mission designed to study?",
      "options": { "A": "Mars", "B": "Jupiter", "C": "Saturn", "D": "Neptune" },
      "correct_option": "B"
    }
  ]
}
```

**Sample for `objective.statement_analysis`** (Template T3):

```json
{
  "items": [
    {
      "source_refs": ["n3"],
      "topic": "the IEEE Global Quantum Internet Standards",
      "statements": [
        "The IEEE standards define a uniform protocol stack for quantum key distribution.",
        "The standards are binding on all member states of the United Nations.",
        "The standards include interoperability requirements for entanglement-based networks."
      ],
      "options": { "A": "1", "B": "1+3", "C": "2+3", "D": "All of the above" },
      "correct_option": "B"
    }
  ]
}
```

These samples are also stored in `tests/fixtures/qa/` and consumed by the §32.6 Phase 8 snapshot tests.

#### 22B.3.C Validation, Rejection & Regeneration Logic (Model Live)

The model is treated as a live, occasionally-drifting collaborator. Even with Structured Outputs the response can deviate in subtle ways (semantic-rule violations, allowed-set drift on Template T3). The backend handles every deviation deterministically without manual intervention.

**Validation pipeline** (runs for every response, every attempt):

```
1. Refusal check       → model returned a structured refusal (Structured Outputs surfaces this).
                          If yes  → terminate retry chain; return refusal error to wizard.
                          If no   → continue.

2. Parse JSON          → safe parse. Failure (extremely rare with Structured Outputs)
                          counts as a validation failure, triggers retry (see below).

3. Schema validate     → JSON Schema validation against the requested type
                          (Pydantic model on backend, mirrored Zod on frontend).
                          Hard reject on any schema failure.

4. Business validate   → per-type rules:
   T1: nothing extra beyond schema.
   T2: correct_option ∈ keys(options); options has exactly four keys A/B/C/D;
       no option value repeats; no "All of the above" / "None" in any value.
   T3: statements.length === 3; options has exactly four keys A/B/C/D;
       every options value ∈ allowed_set; no duplicate values; correct_option
       references an existing key.

5. Source linkage     → source_refs (model response field) must be a non-empty
                          subset of the `ref` values supplied in the request
                          payload's news_items[]. Any unknown ref = hard reject
                          (model fabrication); retry. The model NEVER returns
                          atomic_uid values — those are backend-only (see
                          §22B.3.A Identity Boundary).

6. Per-request shape   → number of items returned === number of items requested.
                          Over/under-count is a validation failure.
```

**Rejection (terminal, this attempt)** — any of the above checks failing marks the attempt failed. The deviating response is **discarded entirely** — no partial-accept, no field-level patching. The next attempt regenerates from scratch.

**Regeneration logic** when the model is **live and accepting requests** but its output deviates:

```
attempt 0 (primary model)
  ├─ fail → attempt 1 (same model, same prompt unchanged)
  │   ├─ fail → attempt 2 (same model, prompt augmented with
  │   │                   a SYSTEM-level "Your prior response failed
  │   │                   validation: <terse error summary>. Regenerate
  │   │                   strictly per the schema." note)
  │   │   ├─ fail → advance to fallback model 1 (attempt 3)
  │   │   │   └─ … same 3-attempt cycle …
  │   │   │       └─ … advance through fallback 2, 3, 4 …
  │   │   │           └─ all 15 attempts exhausted → terminal error
```

**Augmentation rule for attempt 2** (last attempt on a given model): the failure reason is summarised to ≤ 200 chars and prepended to the prompt as a system message. This is the only attempt where prompt is mutated; all other retries replay the exact original prompt.

**Per-attempt logging** (writes a row to `openrouter_call_log`, §22B.4 telemetry):

```json
{
  "attempt_index": 2,
  "model_used": "openai/gpt-4o-mini",
  "success": false,
  "validation_errors": [
    "T3 business rule: options.B value '1+4' not in allowed_set",
    "T3 business rule: statements.length === 4 (expected 3)"
  ],
  "prompt_tokens": 812,
  "completion_tokens": 240,
  "total_tokens": 1052
}
```

**Live-model edge cases handled**:

| Edge case                                                       | Handling                                                                                                                                        |
| --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| Model returns extra fields outside the schema                   | Schema validator (Pydantic `extra="forbid"`) rejects; counted as validation failure.                                                            |
| Model drifts an allowed-set value in T3 (e.g. emits "1 and 2")  | Business rule rejects; the attempt-2 augmentation explicitly re-lists the allowed set.                                                          |
| Model returns fewer items than requested                        | Per-request shape check fails; retry.                                                                                                           |
| Model returns more items than requested                         | Per-request shape check fails; retry. Never silently truncate.                                                                                  |
| Model produces a refusal under safety filter                    | §22B.3 step 1 detects; chain terminates; wizard shows refusal copy: _"The model declined to generate Q\&A for the selected content."_           |
| Model unreachable / 5xx / timeout                               | Counts as a transport failure; identical retry/fallback policy.                                                                                 |
| Model deprecated mid-cycle (OpenRouter returns "model removed") | Treated as terminal for that model; chain immediately advances to the next fallback model without consuming further attempts on the dead model. |
| Partial drift: 4 of 5 items valid, 1 invalid                    | All-or-nothing — the full response is rejected and retried. No partial accept (avoids subtle quality inconsistencies in the saved set).         |

**Why all-or-nothing**: partial accept means the saved Q\&A set mixes high-quality and degraded items; users cannot tell which is which. Cost of regeneration is bounded by the 15-attempt cap.

#### 22B.3.D Backend Post-Processing & Identity Assignment

After the validation pipeline (§22B.3.C) returns a green response, the backend performs **mandatory identity translation** before the items are presentable to the wizard or persisted to the database. The model's output never touches the database unmodified.

**Step-by-step**:

```
1. swap_refs_to_atomic_uids(response.items, ref_to_uid_map):
     For each item:
       item.source_atomic_uids = [ ref_to_uid_map[r] for r in item.source_refs ]
       del item.source_refs
     (The ref map is discarded after this step — refs do not persist.)

2. assign_qa_ids(items, document_id):
     Read the current max `qa_items.qa_id` sequence for this document_id.
     For each item in wizard order:
       seq += 1
       item.qa_id = f"QA-{document_id}-{seq:03d}"
     qa_id is assigned ONLY here, ONLY by the backend, ONLY post-validation.

3. attach_provenance(items, model_used_per_group, accepted_by, role):
     For each item, attach a private `_provenance` attribute (NOT part of
     content_json — destined for the qa_items row only):
       item._provenance = {
         model_used: <OpenRouter slug of the model that succeeded for this item's group>,
         generated_by_role: <"teacher" | "editor" | "sub-member">,
         generated_by_id:   <user-id of the actor>,
         accepted_at:       <ISO8601 now>   (set on Accept, not on Generate),
         is_late_added:     <bool>,
         slot_index:        <wizard slot index>,
       }
     Per-item provenance is persisted on qa_items rows for relational audit.
     It is NEVER written into content_json.qa.items[] — see §22B.3.E step 7
     for the split between content_json fields and qa_items row fields.

4. return items to the wizard for Teacher review/edit.

5. on "Accept Q&A": run the §22B.3.E step 7 transactional write
   (per-item rows into qa_items + top-level qa node into content_json).
```

**Invariants enforced at this layer** (no exceptions):

| Invariant                                                                                              | Mechanism                                                                                                                                                                                                                       |
| ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The model never produces an `atomic_uid` value.                                                        | Strict schema (Pydantic `extra="forbid"`) on the response shape: only `source_refs` (array of strings matching `^n\d+$`) accepted. Any field named `atomic_uid` or `source_atomic_uids` in the model response → reject + retry. |
| Every `source_refs` entry resolves to a known atomic\_uid.                                             | Map lookup fails → §22B.3.C step 5 rejection.                                                                                                                                                                                   |
| `qa_id` is never present in the model response.                                                        | Schema rejects it. Assignment happens server-side only.                                                                                                                                                                         |
| `qa_id` is unique within a document.                                                                   | Database unique constraint `(document_id, qa_id)` + backend sequence read inside a transaction.                                                                                                                                 |
| Saved `qa_items.source_atomic_uids` is a non-empty subset of the actual atomic\_uids in this document. | Backend re-checks each resolved uid against `documents.content_json.categories[*].News-Items[*].atomic_uid` before INSERT. Belt-and-braces.                                                                                     |
| The model cannot influence which document the QA item attaches to.                                     | `document_id` is derived from the request session (Teacher's open document), never from the model response.                                                                                                                     |

**What this gives us**:

* Zero risk of model-fabricated atomic\_uids ever entering the audit chain.
* `qa_id` ordering is deterministic and survives multiple wizard runs (new acceptances append after the existing max sequence).
* Aggregation linkage (`source_atomic_uids` used by Compilation/Magazine Q\&A merging in §10/Phase 10) is **provably correct** — every uid was sourced from the backend's own resolved map, not from the model's text.
* Late-add regen (§13.6) reuses the same pipeline — the only difference is the actor role on step 3 and the resulting `qa-late-added = true` flag.

> **Implementation note for the developer**: place this post-processing in a single `qa/post_process.py` module. Any code path that writes to `qa_items` must go through it. Direct INSERTs from model output are an architectural defect.

#### 22B.3.E Assembly, Merge & Atomicity (Multi-Call → Single `qa` Node)

A single wizard run can request a **mix of types** (e.g. 3 × T1 + 2 × T2 + 5 × T3 = 10 items). Per §22B.3.A, the backend dispatches **one OpenRouter call per type group** — never a mixed-schema call. That produces multiple parallel responses that must be merged into a single, slot-ordered, persistable `qa` node.

This subsection is the full backend walkthrough from "Generate clicked" to "row in `qa_items` + node in `content_json.qa`."

**1. Wizard slot model**

The wizard collects N questions as an ordered list of **slots** (1-indexed in wizard order):

```python
slots = [
  Slot(slot_index=1, type="subjective.straightforward", news_item_uids=[uid_A]),
  Slot(slot_index=2, type="objective.direct",           news_item_uids=[uid_B]),
  Slot(slot_index=3, type="subjective.straightforward", news_item_uids=[uid_C]),
  Slot(slot_index=4, type="objective.statement_analysis", news_item_uids=[uid_D]),
  Slot(slot_index=5, type="objective.direct",           news_item_uids=[uid_B, uid_E]),
]
```

`slot_index` is the **single ordering authority** for the entire pipeline. The model never sees it; it lives only on the backend.

**2. Group → Dispatch → Per-Group Ref Maps**

Group slots by type while preserving each slot's original `slot_index`:

```python
groups = {
  "subjective.straightforward":   [slots[0], slots[2]],
  "objective.direct":             [slots[1], slots[4]],
  "objective.statement_analysis": [slots[3]],
}
```

For each group, build a **request-scoped `ref → atomic_uid` map** and a **parallel `ref → slot_index` map**:

```python
# group "objective.direct" — 2 slots, slots[1] and slots[4]
# slot[1] has 1 news_item (uid_B); slot[4] has 2 news_items (uid_B, uid_E)
ref_to_uid_T2  = { "n1": uid_B, "n2": uid_B, "n3": uid_E }
ref_to_slot_T2 = { "n1": 2,     "n2": 5,     "n3": 5     }  # slot_index values
```

Refs are **per-group** — `n1` in the T2 call has no relation to `n1` in the T1 call. Maps live only for the duration of their group's call.

**3. Dispatch (sequential, one per group)**

Per §22B.3.A: a sequence of three calls in fixed order T1 → T2 → T3 (whichever groups are non-empty). The wizard shows a single progress indicator that advances per group. Each call follows the §22B.3 lifecycle independently (own 15-attempt retry chain, own validation pipeline).

```
dispatch_order = [T1_group, T2_group, T3_group]  # only non-empty groups
for group in dispatch_order:
    response = openrouter_call(group, retry_chain=15)
    if response.terminal_error:
        ABORT_ENTIRE_WIZARD_RUN(reason=response.error)   # see §22B.3.E step 6
    group.response = response
```

**4. Re-association (per group)**

Each group's response is `{ "items": [...] }` in payload-positional order — the model produces one item per news\_item entry it received. Walk the response and re-attach `slot_index` via the per-group map:

```python
for group in dispatch_order:
    for response_item in group.response.items:
        # The model copied source_refs verbatim from the input payload.
        # Find the slot_index by looking up any one of the source_refs in the
        # per-group ref→slot map. (All refs in a single item's source_refs
        # belong to the same slot, by construction at step 2.)
        slot_index = ref_to_slot_for(group)[response_item.source_refs[0]]
        response_item._slot_index = slot_index   # private attribute, not persisted
```

If a returned item's `source_refs` resolves to more than one distinct slot\_index (model fabrication across slots), the entire response is rejected per §22B.3.C step 5.

**5. Merge into slot order + Identity Assignment**

Collect all re-associated items from all groups, sort by `slot_index`, then run §22B.3.D post-processing in that order. Because `slot_index` ascends 1..N, the resulting `qa_id` sequence numbers (assigned by §22B.3.D step 2) honour wizard order.

```python
merged = [item for group in dispatch_order for item in group.response.items]
merged.sort(key=lambda it: it._slot_index)

# Step 1 of §22B.3.D (per-item ref swap, using each item's own group map)
for item in merged:
    item.source_atomic_uids = [ ref_to_uid_for(item._group)[r] for r in item.source_refs ]
    del item.source_refs
    del item._slot_index    # discard ephemeral wizard ordering
    del item._group

# Step 2 of §22B.3.D (qa_id assignment in wizard order)
seq = read_max_qa_seq(document_id)   # 0 on first wizard run; N on late-add
for item in merged:
    seq += 1
    item.qa_id = f"QA-{document_id}-{seq:03d}"
```

Provenance (model\_used, generated\_by\_role, generated\_by\_id) is set per item from the group that produced it (different type groups may have succeeded on different fallback models — that detail is preserved per item).

**6. Atomicity rule — all-or-nothing across groups**

If **any** group's OpenRouter call terminally fails (15-attempt cap exhausted, model refusal, transport collapse), the **entire wizard run is aborted**. No partial qa\_items are saved, no qa node is written. The wizard returns to the type-selection step with an error banner: _"Q\&A generation failed during the {type} group. Please retry, or skip Q\&A for this document."_

Rationale: a mixed wizard run is a single user intent ("give me these 10 items"). Persisting 6 successful items and silently dropping 4 produces a misleading saved set. The user always knows their full request either succeeded entirely or failed cleanly.

**7. Wizard review → Accept → DB transaction (provenance split)**

The merged, identity-assigned items are returned to the wizard for Teacher review/edit (§10.10). Teacher can edit any field (statement, answer, option text, correct\_option, statements text, topic). Teacher **cannot** change: `qa_id`, `source_atomic_uids`, `type`. Those are immutable post-assignment.

**Provenance split** — two destinations for each item, never mixed:

| Destination                         | What goes there                                                                                                                                                                                                                                                                                          | Why                                                                                                                                                 |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `content_json.qa.items[]`           | **Rendering fields only**: `qa_id`, `source_atomic_uids`, `type`, plus type-specific (`statement`/`answer` OR `statement`/`options`/`correct_option` OR `topic`/`statements`/`options`/`correct_option`).                                                                                                | This is what WeasyPrint, PptxGenJS, HTML export, translation actually render. Must match §11.1 byte-for-byte. Stays lean.                           |
| `qa_items` table row (one per item) | **Full audit + content**: `qa_id`, `qa_id_seq`, `slot_index`, `document_id`, `org_id`, `source_atomic_uids`, `type`, `payload` (the JSON blob of type-specific fields), **`generated_by_role`**, **`generated_by_id`**, **`model_used`**, **`is_late_added`**, **`accepted_at`**, `edited_after_accept`. | Relational source of truth for joins, telemetry, aggregation (§10/Phase 10), and late-add granularity. Provenance lives here, not in content\_json. |

**Top-level `qa` node fields** (set ONCE, on the **first** acceptance for a document; later acceptances do NOT overwrite):

| Field               | Source                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `generated_by_role` | Role of the actor who first accepted Q\&A on this document. Stable thereafter.                                                                                                                                                                                                                                                                                                                  |
| `generated_at`      | UTC timestamp of the first acceptance. Stable thereafter.                                                                                                                                                                                                                                                                                                                                       |
| `model_used`        | OpenRouter slug of the **primary** model used by the first wizard run (i.e. the model that succeeded for the first group dispatched). If the first run was mixed-type and different groups succeeded on different models, this carries the FIRST group's successful model. Per-item granularity (which exact model produced which item) lives on `qa_items.model_used` rows. Stable thereafter. |

Rationale: top-level fields are **render-time metadata** (e.g. PDF footer might show "AI-generated by gpt-4o-mini on 2026-03-01"). They must be stable; otherwise late-adds flip the displayed model name and acceptance time, which confuses users. Per-item provenance for full audit (every actor, every model, every acceptance time) lives on the `qa_items` rows.

**The transaction**:

```sql
BEGIN;

-- 7.1 Concurrency lock (see step 10 for race semantics).
SELECT MAX(qa_id_seq) AS current_max FROM qa_items
  WHERE document_id = :doc_id
  FOR UPDATE;

-- 7.2 If current_max differs from the seq window assigned at step 5,
--     re-assign qa_ids starting from current_max + 1, in slot_index order.

-- 7.3 Insert one row per item into qa_items WITH per-item provenance.
INSERT INTO qa_items
  (qa_id, qa_id_seq, slot_index, document_id, org_id,
   source_atomic_uids, type, payload,
   generated_by_role, generated_by_id, model_used,
   is_late_added, accepted_at, edited_after_accept)
VALUES
  (...), (...), (...);   -- N rows, one per merged item

-- 7.4 Build the content_json.qa node as a LEAN render-only structure.
--     Pseudocode for the JSONB value `merged_qa_node`:
--       merged_qa_node = {
--         "generated_by_role": <preserved if qa already exists; else from this run's actor>,
--         "generated_at":      <preserved if qa already exists; else now()>,
--         "model_used":        <preserved if qa already exists; else first group's successful model>,
--         "items": [
--           {  // for each row inserted at 7.3, in qa_id_seq ascending order:
--              "qa_id": <row.qa_id>,
--              "source_atomic_uids": <row.source_atomic_uids>,
--              "type": <row.type>,
--              ...<expand row.payload — type-specific render fields ONLY>
--           },
--           ...
--         ]
--       }
--     Note: provenance fields (model_used, generated_by_role, generated_by_id,
--           accepted_at, is_late_added, slot_index) are NEVER copied into the
--           items[] entries. They live exclusively on qa_items rows.

-- 7.5 If first acceptance: write the whole qa node.
--     If late-add (qa node already exists): preserve top-level fields, replace items[].
UPDATE documents
SET content_json = CASE
      WHEN content_json ? 'qa' THEN
        -- Late-add: preserve top-level metadata, replace items[] only.
        jsonb_set(
          content_json,
          '{qa,items}',
          :merged_qa_items_jsonb,
          true
        )
      ELSE
        -- First acceptance: write the whole qa node.
        jsonb_set(
          content_json,
          '{qa}',
          :merged_qa_node_jsonb,
          true
        )
    END,
    has_qa = true,
    qa_item_count = (SELECT COUNT(*) FROM qa_items WHERE document_id = :doc_id),
    qa_late_added = (content_json ? 'qa')   -- true iff qa node existed before this txn
WHERE id = :doc_id;

-- 7.6 Pipeline event for late-add (§13.6) to trigger PDF regen.
INSERT INTO pipeline_events (document_id, event_type, triggered_by, triggered_at, ...)
VALUES (:doc_id, 'qa_late_added', :actor, now(), ...)
  WHERE :is_late_add = true;

COMMIT;
```

If late-add (§13.6), the `qa_late_added` event fires the PDF regen pipeline outside this transaction.

**8. Final shape of the assembled `qa` node — matches §11.1 byte-for-byte**

After commit, `documents.content_json.qa` matches the canonical §11.1 schema exactly. One `items[]` entry per accepted question, in `slot_index` order, each with a unique sequential `qa_id`. Top-level fields (`generated_by_role`, `generated_at`, `model_used`) are render-only metadata set ONCE on first acceptance. Per-item provenance does NOT appear here — it lives on `qa_items` rows.

Concrete output for the 5-slot wizard run above (mixed T1+T2+T3), produced by the §22B.3.E step 7 transaction:

```json
{
  "qa": {
    "generated_by_role": "teacher",
    "generated_at": "2026-03-01T15:45:00.000Z",
    "model_used": "openai/gpt-4o-mini",
    "items": [
      {
        "qa_id": "QA-NL-20260301-0001-001",
        "source_atomic_uids": ["010326-News-IndiaLaunc-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-e5f6g7h8"],
        "type": "subjective.straightforward",
        "statement": "What is the peak performance of India's new AI supercomputer?",
        "answer": "210 petaflops"
      },
      {
        "qa_id": "QA-NL-20260301-0001-002",
        "source_atomic_uids": ["010326-News-NASAEuropa-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-u1v2w3x4"],
        "type": "objective.direct",
        "statement": "Which planet's moon is NASA's Europa Clipper mission designed to study?",
        "options": { "A": "Mars", "B": "Jupiter", "C": "Saturn", "D": "Neptune" },
        "correct_option": "B"
      },
      {
        "qa_id": "QA-NL-20260301-0001-003",
        "source_atomic_uids": ["010326-News-Reserve3rd-Cat-ECONOMY-NL-20260301-0001-p1q2r3s4"],
        "type": "subjective.straightforward",
        "statement": "By how many basis points did the RBI hike the repo rate?",
        "answer": "25 basis points"
      },
      {
        "qa_id": "QA-NL-20260301-0001-004",
        "source_atomic_uids": ["010326-News-GlobalQuan-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-c9d0e1f2"],
        "type": "objective.statement_analysis",
        "topic": "the IEEE Global Quantum Internet Standards",
        "statements": [
          "The IEEE standards define a uniform protocol stack for quantum key distribution.",
          "The standards are binding on all member states of the United Nations.",
          "The standards include interoperability requirements for entanglement-based networks."
        ],
        "options": { "A": "1", "B": "1+3", "C": "2+3", "D": "All of the above" },
        "correct_option": "B"
      },
      {
        "qa_id": "QA-NL-20260301-0001-005",
        "source_atomic_uids": [
          "010326-News-NASAEuropa-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-u1v2w3x4",
          "010326-News-ESAJupiter-Cat-SCIENCE_AND_TECHNOLOGY-NL-20260301-0001-t8u9v0w1"
        ],
        "type": "objective.direct",
        "statement": "Which two missions are jointly studying Jupiter's icy moons in 2026?",
        "options": { "A": "Europa Clipper & JUICE", "B": "Cassini & Galileo", "C": "Juno & Voyager-2", "D": "MESSENGER & New Horizons" },
        "correct_option": "A"
      }
    ]
  }
}
```

**Diff vs your §11.1 reference**: identical structure. Same top-level fields, same per-item field set per type, same key ordering convention. The only addition is multi-uid `source_atomic_uids` on item 5 (slot 5 had 2 news\_items selected — entirely valid per the wizard model).

**Single sources of truth — non-overlapping**:

* `documents.content_json.qa` → **rendering** (WeasyPrint PDF, PptxGenJS PPTX, HTML export, translation). Lean, matches §11.1.
* `qa_items` rows → **relational joins** (per-item provenance, aggregation linkage in §10/Phase 10, telemetry, late-add audit trail). Carries every field both content\_json has AND the provenance fields content\_json deliberately omits.

**9. Late-add wizard run (§13.6)**

The pipeline above runs identically. Two differences:

* `seq` starts from the existing `MAX(qa_id_seq)` for the document (not from 0). New items append: `QA-{doc-id}-006`, `QA-{doc-id}-007`, ….
* `is_late_add = true` → sets `documents.qa_late_added = true` and fires the `qa_late_added` pipeline event that triggers Stage-3 PDF regen (§13.6).

**10. Concurrent wizard acceptances**

If two actors (e.g. Editor and sub-member both with `permissions.qa = true`) hit "Accept Q\&A" on the same document simultaneously, the `SELECT FOR UPDATE` in step 7 serialises them. The second-committing transaction observes the first's writes and re-assigns its qa\_ids starting from the updated max sequence. Both sets persist in slot order, just with a sequence-window offset for the second. Neither loses items; neither overwrites the other.

### 22B.4 Spend Guardrails

**Per-organisation monthly token cap** (configurable by Super Admin per Org; default applies system-wide):

| Setting                        | Default               | Notes                                                                                                                          |
| ------------------------------ | --------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `qa_monthly_token_cap`         | 100,000 tokens        | Sum of prompt + completion tokens reported by OpenRouter per call. Reset at the Org's billing-cycle start.                     |
| `qa_per_request_question_cap`  | 10 questions          | Hard ceiling regardless of wizard input (matches §10.10 max).                                                                  |
| `qa_per_request_news_item_cap` | = number of questions | Selected news-item count cannot exceed `N`. Enforced client-side and re-checked server-side.                                   |
| `qa_disabled_on_cap_breach`    | true                  | When `qa_monthly_token_cap` is reached, Q\&A generation is disabled for the Org until the cap resets or Super Admin raises it. |

**Cap-breach behaviour**:

* Wizard "Generate" button is disabled; tooltip surfaces _"Your organisation has reached its monthly Q\&A generation limit. Contact your administrator to increase the limit or wait for the next billing cycle."_
* No partial-completion: a request that would push the running token total over the cap is **rejected up front** (estimated tokens computed from prompt size + max completion tokens for the requested type).

**Telemetry**:

* Every OpenRouter call is logged in a `openrouter_call_log` table with: timestamp, org\_id, document\_id, model used, attempt index, prompt\_tokens, completion\_tokens, total\_tokens, success boolean, validation\_errors (if any).
* Super Admin dashboard surfaces aggregate spend per Org per month and a system-wide total.

### 22B.5 RBAC for Q\&A Actions

| Role        | Trigger Q\&A generation                                                                                       | Edit Q\&A items  | Accept Q\&A items | Configure models | View OpenRouter telemetry |
| ----------- | ------------------------------------------------------------------------------------------------------------- | ---------------- | ----------------- | ---------------- | ------------------------- |
| Super Admin | No (never operates inside an Org's document)                                                                  | No               | No                | **Yes (sole)**   | **Yes (sole)**            |
| Editor      | Yes (during Stage 1 review + post-Stage-3 late-add)                                                           | Yes              | Yes               | No               | No                        |
| Teacher     | Yes (pre-submission wizard + post-Stage-3 late-add)                                                           | Yes              | Yes               | No               | No                        |
| Sub-member  | Only if `permissions.qa = true` (granted by Teacher in §21.3) — pre-submission wizard + post-Stage-3 late-add | Yes (if granted) | Yes (if granted)  | No               | No                        |

A new boolean `permissions.qa` is added to the `sub_members.permissions` JSON (defaults to `false`).

### 22B.6 Translation Interaction

When the source document has translations (`is-translated = true`), the `qa` node is included in the translation request to Google Translate (§18). The translated `qa` node is written into each parallel translated document. Q\&A items in a translated document use the same `qa_id` value as in the source — this enables back-mapping during aggregation.
