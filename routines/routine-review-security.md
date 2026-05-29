# Routine: Review Security

Autonomous security review of a target codebase, filed as a Linear ticket. Wraps the
`review-security` skill (the analysis) and adds the operator config the skill leaves out.

## Invoke

Run **`/deep-core-skills:review-security`** against the scoped codebase. It reads the shared
security lens, maps trust boundaries, and emits **one** structured finding — the single most
significant systemic security weakness — or exits cleanly if nothing meets the bar.

The skill is analysis-only and strictly defensive: it never produces exploit code, and it does not
touch Linear. This routine owns everything below.

## Scope

`<target repo / project / path>` — set before running. If the agent runs inside one repo, scope is
that repo; otherwise name the path or project explicitly.

## Cadence

On-demand, or scheduled (e.g. weekly, or on every significant change to an auth/trust-boundary
surface). More frequent than architecture is reasonable — security regressions bite harder.

## On finding → file a ticket

If the skill returns a finding, hand it to **`/deep-core-skills:to-issues`** to create a Linear
issue (team `Engineering`):

- Title from the finding's one-line weakness statement (defensive framing — the design gap, not an
  exploit).
- Body = the finding's Trust boundary / Evidence / Impact / Suggested direction / Severity /
  Confidence sections.
- Apply the category + state labels from the canonical vocabulary (`to-issues` reads `LABELS.md`):
  `bug` + `ready-for-human` — security hardening needs human judgment, so it's HITL.

If the skill exits with no finding, do nothing — no empty tickets.

## Dedup

Before filing, search open Linear issues for the same weakness (by trust boundary + affected
surface). **Skip filing** if an open issue already covers the same root weakness; instead comment
that it recurred, with today's date and any new `file:line` evidence. "Same root" = same boundary +
same class of gap, not just the same OWASP category anywhere.

## Escalation

If the finding is **genuinely Critical, exploitable, and currently-live** (active RCE, auth bypass,
hardcoded production secret, public PII exposure), the skill leads its summary with it — this
routine then escalates: file at urgent priority **and** raise it out-of-band (e.g. notify the
on-call channel) rather than letting it sit in the triage queue. Non-critical findings file at
normal priority.

## Output

End with a one-paragraph summary: the finding, the Linear action taken (created `ENG-NNN` /
commented / none), and the hypotheses considered and discarded. Keep it defensive throughout — no
weaponizable detail.
