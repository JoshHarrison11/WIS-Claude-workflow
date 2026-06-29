# Docs as source of truth (shared — read freely, write under the gate)

The project docs should reflect reality after each change. But auto-maintained docs have a
dangerous failure mode: a doc that drifts from reality while still being TRUSTED becomes a
confident liar, and the next cycle reads its error as fact. So:

**READ — generously.** Every role consults the docs for context (see engineering-principles #5).

**WRITE — under the gate.** A doc update is a CHANGE and goes through review like code.
- **Doc update is part of the change's scope.** When a change alters documented behaviour or
  settles a decision, updating the relevant doc is part of that change — planned, implemented,
  reviewed.
- **Settled facts only.** Docs record what is TRUE after the change lands — never speculation,
  plans, or in-progress state. "We intend to" does not go in the source of truth; "this now does X"
  does, once X is verified.
- **Local, not monolithic.** Update the relevant close-to-the-thing doc — the decision log for a
  decision, CLAUDE.md for a standing rule (rarely, deliberately), the contract `_note` for contract
  behaviour, a changelog for what changed. Do NOT maintain one giant auto-rewritten "project
  overview" file — that's the doc most likely to rot, because no one re-reads it and errors
  accumulate invisibly. The "overview" a cycle works from is ASSEMBLED by reading the local docs at
  the start, not stored as a drifting megafile.
- **Code is truth; docs match code; never the reverse.** (See engineering-principles #6.)
- **Human glances at doc updates.** A bad doc misleads every FUTURE cycle, not just this one — so
  doc updates carry extra caution: propose them and surface them to the human on the way to commit,
  same as a code diff. Do not auto-write the source of truth unsupervised until the loop has earned
  trust.