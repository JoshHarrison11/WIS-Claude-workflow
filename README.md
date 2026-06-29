# WIS change-review workflow (Claude Code marketplace repo)

A gated, multi-role change workflow for the WIS project, distributed via Claude Code's marketplace
(add this repo's URL as a marketplace source; the team gets the roles + workflow).

The point of the roles is NOT to agree pleasantly — it is for each to CHALLENGE and, where
warranted, say no. The gate converges on a good change or surfaces the disagreement to a human.

## What's here

**Roles (subagents)** — specialised evaluators, each with its own context and a narrow charter:
- `agents/ceo-product.md` — should this change happen at all? (necessary / right approach / timing)
- `agents/tech-lead.md` — plan it: scope, blast radius, gate, test, blockers.
- `agents/sap-functional-lead.md` — SAP/CDS specialist; flags SAP-shaped risks (cardinality, NUMC,
  associations, field-name pockets, filter gotchas) and names what must be verified against the DDL
  / Rafael. NEVER asserts SAP facts it can't see from the repo. Returns NO_SAP_SURFACE on non-SAP
  changes. Its checklist is reused by the reviewer.
- `agents/implementer.md` — build it to the plan, on verified evidence; flag deviations.
- `agents/reviewer.md` — adversarial final gate; APPROVED / CHANGES_REQUIRED / ESCALATE_HUMAN.

**Workflow (skill)** — orchestration:
- `skills/workflow/SKILL.md` — triage (light path vs full gate), the loop, the 3-iteration cap,
  human escalation. The orchestrator owns the loop; the roles don't drive workflow state.

**Shared context (single source of truth — referenced by all roles, never duplicated):**
- `skills/shared/engineering-principles.md` — evidence-or-silence, repo boundaries, surgical scope,
  verify-don't-assume, code-is-truth, anti-sycophancy.
- `skills/shared/safety-guards.md` — the honesty fences that must never be weakened.
- `skills/shared/platform-discipline.md` — how changes are made on WIS (DB-not-front-end, the
  wis-tool-change gate, known landmines).
- `skills/shared/docs-as-source-of-truth.md` — docs read freely, written under the gate.

**Examples:**
- `examples/safe-change.md` — a light-path change (the invoice-note consistency fix).
- `examples/risky-change.md` — a full-gate change (the router flip) showing the gate catching wrong
  premises.

## Design (why subagents + a workflow skill, not one big thing)
- **Roles are evaluators → subagents** (separate context per role; the reviewer shouldn't be primed
  by the implementer's deliberation, the CEO shouldn't wade through code edits).
- **Process is orchestration → a workflow skill** (owns the loop and the iteration cap;
  deterministic; doesn't rely on agents recursively invoking each other).
- **Rules are shared context → referenced files** (one source of truth; update once, all roles see
  it; no drift).

## ⚠️ VERIFY THESE AGAINST CURRENT CLAUDE CODE DOCS before relying on the repo
The CONTENT (role charters, shared rules, workflow logic) is stable. The FORMAT/MECHANICS below
evolve — confirm against docs.claude.com, don't trust this repo's assumptions:
1. **Marketplace manifest** — whether a top-level manifest/index file is required for Claude Code to
   recognise this repo as a marketplace source, and its exact format. This repo does NOT include one
   yet — add per the current marketplace docs.
2. **Subagent file format** — the exact frontmatter keys and directory convention for `agents/*.md`.
   These files use `name` + `description` frontmatter; confirm that matches the current spec.
3. **Skill file format** — the `SKILL.md` frontmatter/structure under `skills/*/`.
4. **Orchestration semantics** — whether a skill can drive subagents in sequence (the workflow skill
   assumes this). If not, a human or a top-level command drives the same sequence; the charters and
   loop rules are unchanged either way.

The architecture and the WIS-specific content are the durable part. The wiring is the part to
confirm.
