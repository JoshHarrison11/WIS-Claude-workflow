---
name: wis-change-workflow
description: >
  Orchestrates a gated change to the WIS project through the review roles. Use when making any
  non-trivial change to the WIS codebase. Triages the change to a light path (small/safe) or the
  full four-role gate (risky), then drives the loop: ceo-product -> tech-lead -> implementer ->
  reviewer, looping on CHANGES_REQUIRED until APPROVED or a 3-iteration cap, then escalating to the
  human. The orchestrator owns the loop and the iteration count; the role subagents do the work but
  do not drive workflow state.
---

# WIS change workflow

This skill runs a change through the review roles. It owns the loop; the role subagents
(ceo-product, tech-lead, implementer, reviewer) do the evaluation/work but do not drive the
sequence themselves.

Read first (bundled at the plugin root, located at runtime — a concept pointer, not a cwd-relative path): engineering-principles (applies to every step).

## Step 0 — TRIAGE (run first; decides the path)

**LIGHT PATH** (tech-lead quick-check -> implementer -> reviewer) when ALL of:
- touches ONE file or one contract group, AND
- does NOT touch a CDS view, a synthesis pattern, an honesty guard, a default-query intent mapping,
  or any test-pinned behaviour, AND
- is reversible and self-contained (a note edit, a doc, a single override-only field add, a
  typo/comment fix), AND
- has no monetary/financial-discrepancy output and no per-person metric.

**FULL GATE** (ceo-product -> tech-lead -> implementer -> reviewer, looping) when ANY of:
- touches a CDS view, a contract's include_groups_by_intent, a synthesis pattern/engine, an agent
  prompt, or the router, OR
- touches or could touch a guard in `safety-guards` (bundled at the plugin root), OR
- changes test-pinned behaviour, OR
- produces financial/discrepancy output or per-operator/per-person metrics, OR
- is a new capability, a multi-file change, or the requester is unsure, OR
- the light-path tech-lead quick-check surfaces hidden scope/risk -> ESCALATE to full.

Default to FULL when uncertain. A light-path change that turns out to touch a guard or a view STOPS
and escalates.

## The loop (full gate)

1. **ceo-product** -> APPROVE / RESHAPE / REJECT. If REJECT, stop. If RESHAPE, restart from the
   reshaped request.
2. **tech-lead** -> PLAN (scope, blast radius, gate, test, blockers). If unplannable small/testable,
   back to ceo-product.
3. **implementer** -> CHANGE + EVIDENCE, or a DEVIATION report back to tech-lead.
4. **reviewer** -> APPROVED (done) / CHANGES_REQUIRED (back to the owning role) / ESCALATE_HUMAN.
5. Repeat 3-4 until APPROVED or the cap.

## Termination (hard rules — the loop must not run forever or converge on a bad answer)

- **CAP: max 3 review iterations.** If the reviewer still has findings after 3 rounds, STOP and
  escalate to the HUMAN with: the change, the open findings, and where the roles disagree. Do not
  keep looping; do not approve to break the loop.
- **HUMAN IS FINAL GATE.** The roles converge or they surface the disagreement. They NEVER ship
  something the reviewer flagged as a fence-weakening or silent-failure regression — that always
  escalates.
- **NON-CONVERGENCE IS A VALID OUTCOME.** "We could not agree this is the right change" sent to the
  human is the gate working, not failing.
- **THE CAP IS PROMPT-ENFORCED, NOT RUNTIME-ENFORCED.** The 3-iteration cap and the "never approve
  to break the loop / never ship a flagged regression" rules are honoured by the orchestrating model
  itself — there is NO harness mechanism that hard-stops the loop. This is acceptable for a gated
  review workflow, but is a known limitation.
- **UPGRADE PATH** (only if a guaranteed cap ever becomes a hard requirement, e.g. cost/safety):
  port the orchestration to a Workflow script that calls `agent(..., {agentType: "ai-workflow:<role>"})`
  and enforces the `iteration < 3` loop in JS — same charters, same verdicts, deterministic
  termination. Treat as an upgrade, not a current necessity.

## Anti-sycophancy
A cycle where ceo says "good idea", lead says "great plan", implementer says "done", reviewer says
"looks good" — with no challenge anywhere — is a FAILED cycle. If nothing was questioned, the gate
added latency and caught nothing. Each role must be willing to say no.

---

> **NOTE ON MECHANICS (CONFIRMED — skill-driven subagent orchestration is supported in this
> Claude Code version):**
> - This skill runs INLINE in the main conversation loop (no `context: fork`), so the main agent's
>   Agent tool stays available to invoke the role subagents.
> - The orchestrator invokes each role by its PLUGIN-SCOPED name via the Agent tool —
>   `ai-workflow:ceo-product`, `ai-workflow:tech-lead`, `ai-workflow:implementer`,
>   `ai-workflow:reviewer` (adjust the `ai-workflow:` prefix to match this plugin's actual name).
> - REQUIREMENT: the plugin must be ENABLED for those scoped agent names to resolve.
> - The Agent tool returns ONLY each role's final message (not its transcript) — so the orchestrator
>   routes by matching the verdict token in that final message.
> - The orchestrator holds the iteration counter in its own context (optionally surfaced via
>   TodoWrite). There is no built-in state store.
> - The human-driven sequence remains a valid degradation path but is NO LONGER the expected path.