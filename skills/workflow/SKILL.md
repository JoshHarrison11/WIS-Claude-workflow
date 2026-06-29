---
name: wis-change-workflow
description: >
  Orchestrates a gated change to the WIS project through the review roles. Use when making any
  non-trivial change to the WIS codebase. Triages the change to a light path (small/safe) or the
  full gate (risky), then drives the loop: ceo-product -> tech-lead -> sap-functional-lead ->
  implementer -> reviewer, looping on CHANGES_REQUIRED until APPROVED or a 3-iteration cap, then
  escalating to the human. The orchestrator owns the loop and the iteration count; the role
  subagents do the work but do not drive workflow state.
---

# WIS change workflow

This skill runs a change through the review roles. It owns the loop; the role subagents
(ceo-product, tech-lead, sap-functional-lead, implementer, reviewer) do the evaluation/work but do
not drive the sequence themselves.

Read first: `shared/engineering-principles.md` (applies to every step).

## Step 0 — TRIAGE (run first; decides the path)

**LIGHT PATH** (tech-lead quick-check -> implementer -> reviewer) when ALL of:
- touches ONE file or one contract group, AND
- does NOT touch a CDS view, a synthesis pattern, an honesty guard, a default-query intent mapping,
  or any test-pinned behaviour, AND
- is reversible and self-contained (a note edit, a doc, a single override-only field add, a
  typo/comment fix), AND
- has no monetary/financial-discrepancy output and no per-person metric.

**FULL GATE** (ceo-product -> tech-lead -> sap-functional-lead -> implementer -> reviewer, looping) when ANY of:
- touches a CDS view, a contract's include_groups_by_intent, a synthesis pattern/engine, an agent
  prompt, or the router, OR
- touches or could touch a guard in `shared/safety-guards.md`, OR
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
3. **sap-functional-lead** -> NO_SAP_SURFACE (pass through) / SAP_OK / SAP_RISKS. If SAP_RISKS at
   plan level, back to tech-lead to fold the verification into the plan. On a change with no SAP
   surface (pure Python/routing), this returns NO_SAP_SURFACE and the loop continues — it does NOT
   manufacture a concern. Its checklist is also applied later by the reviewer.
4. **implementer** -> CHANGE + EVIDENCE, or a DEVIATION report back to tech-lead.
5. **reviewer** -> APPROVED (done) / CHANGES_REQUIRED (back to the owning role) / ESCALATE_HUMAN.
   The reviewer applies the sap-functional-lead checklist to any SAP-touching change as part of its
   review.
6. Repeat 4-5 until APPROVED or the cap (the implementer-reviewer loop; re-run sap-functional-lead
   only if a fix re-touches SAP surface).

## Termination (hard rules — the loop must not run forever or converge on a bad answer)

- **CAP: max 3 review iterations.** If the reviewer still has findings after 3 rounds, STOP and
  escalate to the HUMAN with: the change, the open findings, and where the roles disagree. Do not
  keep looping; do not approve to break the loop.
- **HUMAN IS FINAL GATE.** The roles converge or they surface the disagreement. They NEVER ship
  something the reviewer flagged as a fence-weakening or silent-failure regression — that always
  escalates.
- **NON-CONVERGENCE IS A VALID OUTCOME.** "We could not agree this is the right change" sent to the
  human is the gate working, not failing.

## Anti-sycophancy
A cycle where ceo says "good idea", lead says "great plan", implementer says "done", reviewer says
"looks good" — with no challenge anywhere — is a FAILED cycle. If nothing was questioned, the gate
added latency and caught nothing. Each role must be willing to say no.

---

> **NOTE ON MECHANICS (verify against current Claude Code docs before relying on this):**
> This skill assumes the orchestrator can invoke the role subagents in sequence and read their
> outputs to drive the loop. Whether/how a skill drives subagents, and the exact subagent-invocation
> semantics, are Claude Code features that evolve — confirm the current behaviour in the official
> docs. If skill-driven subagent orchestration isn't supported the way this assumes, the fallback is
> a human (or a top-level command) driving the same sequence manually; the role charters and the loop
> rules above are unchanged either way.
