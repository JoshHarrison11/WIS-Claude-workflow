# Platform discipline (shared — how changes are made on WIS)

## Where changes live
- **Agent prompts/configs are platform-owned** — changed via the DB / prompt-management surface,
  NOT the front-end. (A front-end edit path has a known bug that detaches tools.) Prompts are not
  in the repo.
- **Code changes** (contracts, views, tools, engine, router) are in the repo and go through Git.
- **Committed-code-is-running-code** after container restart. A change isn't live until committed
  and restarted.

## The wis-tool-change gate (for code that affects tools/engine/contracts)
1. Capture the net/golden FIRST (before any change) — that's the before-baseline.
2. Apply the change.
3. Confirm goldens are byte-identical EXCEPT the one intended fixture. If any unexpected golden
   moves, STOP and report — do not regenerate it to make the suite green.
4. Run the relevant test suite; a change that alters test-pinned behaviour MUST update the test to
   assert the NEW behaviour (not silence the old). A test whose expectation was merely flipped to
   stop failing is not a test.

## Diagnosis
- **Diagnose from traces, not rendered output.** The rendered answer can look right while the
  underlying data/route is wrong. Pull the trace/spans.
- **Trace-reads need their own verification.** A tool's analysis of a trace is itself a claim —
  demand the raw spans, don't trust the summary. (A trace-read once confidently alleged fabrication
  that wasn't there.)

## Known landmines in this codebase (check these specifically)
- **camelCase pockets vs snake_case convention:** `invoice_detail` (inbound) and all of masterdata's
  field_groups are raw camelCase, matched character-for-character to the view's emitted names. A
  casing drift or a view-side alias silently empties these columns (no error). Any change near them
  re-checks the exact emitted names.
- **Silent-degradation modes in the analytics primitives:** a wrong/empty CDS field → all-None rows
  → "insufficient_evidence" with no warning; ratio_pct metrics "surface null" silently. Do not add
  new silent-failure paths; prefer a logged warning or a typed error.
- **`reasoning` is model-family-gated** in agent_builder — silently dropped for non-o1/o3 models
  (incl. GPT-5) even though `supports_reasoning()` returns True. Don't assume a `reasoning` setting
  is applied; verify for the model family.
- **`strict_handoff_filter` is shared with IBP** — don't rip out the strict path assuming it's dead;
  the synthesis-reasoning and IBP routes still use it.

## Standing process
- One agent / one change at a time, tested by trace.
- Surgical fixes over bundled changes.
- Stage files explicitly by name for commits — never `git add .` (the tree often has unrelated
  uncommitted items).
- A commit message records the SETTLED state honestly, including what's deferred or unverified
  (e.g. "blank pending live feed", "uncollectable on host — unverified").