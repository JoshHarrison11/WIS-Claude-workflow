# Example — FULL GATE (risky change): the router strict-flag flip

A real change from this project, used to illustrate the full gate AND the most important lesson:
the gate catching WRONG PREMISES before they became bugs.

## The request
"Flip DIRECT_EXEC `strict = True → False` so a lopsided single-domain route no longer locks to one
agent (keeps handoff agents reachable, winner still recommended)."

## Triage → FULL GATE
Touches the router AND test-pinned behaviour → full gate, looping.

## ceo-product
- Necessary? Yes — a real routing-fragility gap (a lopsided cross-domain question was locked to one
  agent with no escalation). Right approach? Yes — keeping agents reachable with the winner still
  preferred is sound. Risk? Routing correctness; needs tests. Timing? Fine. → APPROVE.

## tech-lead — PLAN
- Scope: one-line source flip + the tests that pin the behaviour.
- Blast radius (named up front): does it leave a dead `strict=True` path? Does it break a pinned
  test? Is `strict_handoff_filter` shared with another product (IBP)?
- Gate: update tests to assert the NEW behaviour, not silence the old. Net = run the routing suite.
- Test: a unit test pinning "DIRECT_EXEC keeps handoffs reachable, strict=False" so flipping back
  fails it.

## implementer — and the KEY MOMENT: two plan premises were WRONG, and were FLAGGED not absorbed
The earlier review (input to the plan) had asserted two things. The implementer VERIFIED against the
running tests and found:
1. The "failing test at line 416" does NOT fail — it belongs to a different (synthesis-reasoning)
   branch the flip never touches. Confirmed by running it.
2. There is NO dead `strict=True` path — `strict_handoff_filter=True` is still emitted by the
   synthesis-reasoning branch AND by IBP, and consumed generically.
→ The implementer reported these as DEVIATIONS rather than building to the wrong premises. The real
break was the §4.1 routing-corpus ratchet (a test DESIGNED to flip red when the fix lands). It
converted the gap-markers to regression guards, added the pinning unit test, and left the shared
IBP path untouched (flagging it rather than ripping it out).
- Also flagged: one pre-existing unrelated test failure (`test_synthesis_patterns_intersection`) —
  reported as out of scope, NOT absorbed to make the suite green.

## reviewer
- Matches plan (adjusted for the verified deviations). Tests assert the NEW behaviour (not silenced)
  — confirmed the unit test genuinely routes through DIRECT_EXEC, not a blind pass. No guard
  touched. The shared IBP path correctly left intact. The pre-existing failure correctly excluded.
  → APPROVED.

## The lesson
The single most valuable thing the gate did was let the implementer say "two of the plan's premises
are wrong — here's the evidence" instead of dutifully building to a wrong plan. Verify against
reality, flag deviations, don't absorb unrelated failures. That is the gate working.