# HCS-25 (Signal Family): SimpleMath / SimpleScience Evals (Informative)

This document complements [HCS-25](../../hcs-25.md) by describing a concrete, minimal evaluation methodology used by some ecosystems to populate “baseline correctness” trust signals.

The goal is not to exhaustively benchmark intelligence. The goal is to detect:

- subjects that cannot follow simple instructions reliably,
- endpoints that time out or fail to respond, and
- endpoints that respond but are consistently incorrect or unparseable.

## Identifier guidance

Simple eval signals SHOULD follow the HCS-25 signal identifier namespace pattern (see [Signal Identifier Namespacing](../../hcs-25.md#signal-identifier-namespacing)).

Implementations typically store per-run details (question id, status, timestamps) and also store an aggregated scalar score used by trust adapters.

## SimpleMath

### Prompt format (recommended)

Implementations SHOULD use a prompt that reduces ambiguity and makes automatic grading robust. For example:

- Instruction: “Answer with just the number.”
- A single arithmetic expression.

Example:

```
Answer with just the number.

What is 37 + 58?
```

### Question generation (recommended)

Implementations SHOULD generate a random arithmetic question per evaluation run. Common choices include:

- addition (two integers, e.g. `10–100`)
- subtraction (two integers, e.g. `10–100` minus `1–50`)
- multiplication (small integers, e.g. `2–12`)

Implementations SHOULD store a stable `questionId` describing the operation and operands (e.g., `math:add:37+58`).

### Grading

Implementations SHOULD grade by extracting the first numeric token from the response and comparing it to the expected answer.

Recommended outcomes:

- `correct` → `score = 100`
- `wrong` → `score = 0`
- `unparseable` (no numeric token) → `score = 0`
- `timeout` / `missing` / `error` → `score = 0`

### Stored signal fields

Implementations SHOULD store:

- `questionId`
- `expected` (optional; can be omitted if sensitive)
- `response` (optional; can be omitted/redacted)
- `status` and `score`
- `sessionId` / `conversationId` (optional; helps correlate debugging)

## SimpleScience

### Prompt format (recommended)

Implementations SHOULD use a small multiple-choice question set so grading is deterministic and language-agnostic.

Recommended prompt structure:

- A question stem
- Four choices labeled `A`, `B`, `C`, `D`
- Instruction: “Answer with just A, B, C, or D.”

Example:

```
Answer with just A, B, C, or D.

Which gas do plants primarily absorb during photosynthesis?
A) Oxygen
B) Carbon dioxide
C) Nitrogen
D) Helium
```

### Question bank

Implementations SHOULD use a curated, small question bank of basic science facts (chemistry, biology, physics, earth science). Each question MUST have:

- stable `questionId`
- prompt text
- exactly four options
- an unambiguous correct choice `A|B|C|D`

Implementations SHOULD randomly sample a question per evaluation run.

### Grading

Implementations SHOULD grade by extracting a single-letter answer from the response:

- If a standalone `A|B|C|D` token exists, use it.
- Optionally, fall back to matching an option string when the response contains the full option text.

Recommended outcomes:

- `correct` → `score = 100`
- `wrong` → `score = 0`
- `unparseable` → `score = 0`
- `timeout` / `missing` / `error` → `score = 0`

## Time budgets and retries

Implementations SHOULD:

- set a time budget per eval run (e.g., 20–30s for synchronous chat endpoints);
- treat timeouts as a scored failure (`0`) unless the ecosystem explicitly chooses not to penalize missingness (see HCS-25 contribution modes).

Implementations MAY retry transient failures, but SHOULD cap retries to avoid amplifying load.

## Anti-gaming guidance (informative)

Simple evals are easy to game. Implementations SHOULD:

- rotate operands/questions,
- avoid publishing the exact full question bank if gaming becomes an issue (publish hashes/ids instead),
- add a small randomized “instruction compliance” check (e.g., “answer with just the number”) to reduce prompt-injection style failures.
