# Example — LIGHT PATH (safe change): fixing a self-contradictory contract note

A real change from this project, used to illustrate the light path.

## The request
"The invoice_detail group is wired into SET_RETRIEVAL (a decided, intended state). But the
contract's `_invoice_detail_note` still says 'override-only / NOT in include_groups_by_intent',
which contradicts the wiring. Make the note consistent."

## Triage → LIGHT PATH
- Touches ONE file (inbound.json), one note string.
- Does NOT touch a view, a synthesis pattern, a guard, a default-query mapping, or a test.
- Reversible, self-contained (a note edit).
- No monetary output, no per-person metric.
→ Light path: tech-lead quick-check → implementer → reviewer-lite.

## Tech-lead quick-check
- Scope: one note string in inbound.json. Blast radius: none (it's a comment; behaviour unchanged).
- Gate: none beyond "change only the note, confirm JSON valid".
- Test: JSON validity + visual confirm the note now matches the wiring. No pinned test touched.
- Blockers: none.

## Implementer
- Verified the contradiction first (quoted the old note + the SET_RETRIEVAL entry).
- Changed ONLY the note string; left wiring, field list, all other groups untouched.
- Evidence: before/after of the note; confirmed no other part of the contract changed; JSON valid.

## Reviewer
- Matches plan: yes, one note string. Consistency: note now matches the wiring — contradiction gone.
- No guard touched, no silent-failure path, no regression (behaviour identical — it's a comment).
- Doc accuracy: the note now records the settled state (wired, 1:1, blank-pending-feed, camelCase
  pending alias). APPROVED.

## Why this is the light path
No challenge was *needed* because nothing risky was touched — but note the implementer still
*verified the contradiction against the actual file* rather than trusting the request's framing.
Even on the light path, evidence-before-assertion holds.