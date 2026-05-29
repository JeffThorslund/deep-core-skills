# The maintainer is the stage gate; triage and slicing stay human

Autonomy in the pipeline is confined to **proposing** work: scheduled `review-*`
skills survey the codebase and a routine files each finding as a single
`needs-triage` ticket. Everything downstream of that — deciding whether a finding is
worth doing, what state it belongs in, and how to slice a large ticket — is done by
the maintainer running the interactive `triage` and `to-issues` skills by hand. No
autonomous skill assigns a state label or splits a ticket.

## Considered Options

- **Autonomous triage + auto-split** (an earlier design) — scheduled agents read
  tickets, assigned states, and decomposed big findings into `ready-for-agent`
  children with no human in the loop until a PR. Rejected: triage and slicing carry
  the most judgment in the whole system, yet this put the *least* human oversight
  exactly there. The agent would rubber-stamp work no one decided was worth doing, and
  authored slice boundaries no one reviewed.
- **Maintainer as stage gate (chosen)** — autonomy proposes (reviews file findings);
  the maintainer disposes (triages, slices). The human checkpoint sits at the
  judgment-heavy step, not after it.

## Consequences

- The `review-*` routines file exactly **one** `needs-triage` ticket per finding and
  never split or self-triage. This is the load-bearing constraint — a drift audit
  should not "restore" autonomous triage to them.
- `to-issues` is invoked **only** interactively by the maintainer; no routine calls
  it to split.
- This supersedes the original autonomous route/split design (the prior ADR-0001 and
  ADR-0002, and the `route`/`split`/`park`/`bounce` vocabulary), which is abandoned.
