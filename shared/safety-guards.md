# Safety guards (shared — the honesty fences; NEVER weaken for convenience)

These are correctness, not fat. Any change that loosens one of these is a regression wearing a
feature's clothing — call it out, do not pass it. A fast/convenient/simpler version that crosses
one of these lines is OUT OF BOUNDS.

## The fences

- **Name-in-key-field guard.** A human-readable name must never be passed where SAP requires a
  code. The guard declines name-shaped values before any SAP call. Never let a name through to a
  key field to "make it work."

- **Synthesis causal fences.** In the cross-domain synthesis engine: side-evidence (e.g. execution
  task data bundled into a batch pattern) is CORRELATIONAL CONTEXT ONLY. It must never form a
  relationship link, never feed `evidence_strength` or `allowed_claims`, and never license a causal
  claim ("expired BECAUSE of the backlog"). Execution/side-evidence is scoped out of the
  relationship set structurally — keep it that way.

- **Count ≠ cause; count ≠ culpability.** "Has the most X" / "leads on Y" is a fact about a count.
  It does not make that entity "the problem" or "at fault." State what the data shows and what it
  does NOT establish.

- **A per-entity rate is a metric, not a verdict — don't grade people.** A per-operator rate
  (exception rate, short-pick rate) is reported as a metric to investigate (stock availability, bin
  accuracy, training), NEVER as "this person is underperforming / the problem / the worst." This is
  especially live for picker/operator questions, which are answerable as metrics but must carry the
  investigate-not-blame framing.

- **No invented values.** Never fabricate, estimate, or back-fill a number, name, date, or quantity.
  A blank field is "no value recorded," full stop — never a value to guess. (The planner once
  invented execution averages on top of real counts — that is the exact failure to prevent.)

- **Truncation honesty — slice vs population.** A count from a truncated slice is NOT a population
  count. When a result is truncated, say so, and never present a slice count as the whole. Read
  `total_count` for row counts and `aggregate_record_count` for population counts — they are
  different fields and mean different things.

- **No causal "because" without proof.** For "why" questions, use "points to" / "is consistent
  with" / "co-occurs with" — never "because" / "caused by" — unless the evidence genuinely
  establishes the mechanism. The honest answer often includes what the data CANNOT prove.

## The rule

If implementing a change requires loosening any guard above, that is a signal the PLAN is wrong —
send it back, do not quietly do it. If a "speed up" or "simplify" proposal would weaken a fence,
it is a quality regression, not an optimisation. Escalate it; never absorb it.