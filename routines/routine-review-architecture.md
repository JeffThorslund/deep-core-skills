# Routine: Review Architecture

Autonomous architecture review of a target codebase, filed as a Linear ticket. Wraps the
`review-architecture` skill (the analysis) and adds the operator config the skill leaves out.

## Invoke

Run **`/deep-core-skills:review-architecture`** against the scoped codebase. It reads the shared
architecture lens, surveys the code, and emits **one** structured finding — the single most
significant systemic architectural weakness — or exits cleanly if nothing meets the bar.

The skill is analysis-only: it does not touch Linear. This routine owns everything below.

## Scope

`<target repo / project / path>` — set before running. If the agent runs inside one repo, scope is
that repo; otherwise name the path or project explicitly.

## Cadence

On-demand, or scheduled (e.g. weekly). Architecture drift accumulates slowly — daily is overkill;
weekly or per-milestone is the sweet spot.

## On finding → file a ticket

If the skill returns a finding, hand it to **`/deep-core-skills:to-issues`** to create a Linear
issue (team `Engineering`):

- Title from the finding's one-line problem statement.
- Body = the finding's Evidence / Impact / Suggested direction / Confidence sections.
- Apply the category + state labels from the canonical vocabulary (`to-issues` reads `LABELS.md`):
  `enhancement` + `ready-for-human` — architectural reshaping is a judgment call, so it's HITL, not
  AFK.

If the skill exits with no finding, do nothing — no empty tickets.

## Dedup

Before filing, list open Linear issues that look like prior architecture findings (search the
finding's subject/module). **Skip filing** if an open issue already covers the same root weakness in
the same subsystem; instead add a short comment noting the finding recurred (with today's date and
any new `file:line` evidence). "Same root" = same lens concept in the same subsystem — a leaked seam
in persistence and one in rendering are different tickets.

## Escalation

Architecture findings are rarely "drop everything." File at normal priority. If the finding's
Severity is the skill's top band **and** it blocks active work, also surface it in the run summary
so a human sees it without opening Linear.

## Output

End with a one-paragraph summary: the finding, the Linear action taken (created `ENG-NNN` /
commented on `ENG-NNN` / no action because nothing met the bar), and the hypotheses considered and
discarded.
