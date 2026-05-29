# Triage labels (canonical)

Single source of truth for the issue-tracker labels the `to-issues`, `to-prd`, and
`triage` skills use. These live in **Linear**, team `Engineering`, and the strings
below are the literal label names — apply them verbatim, never create new ones.

## Category labels (what kind of issue)

- `bug` — something is broken; unintended behaviour.
- `enhancement` — a new feature or improvement.

## State labels (where the issue is in triage)

- `needs-triage` — maintainer needs to evaluate this issue.
- `needs-info` — waiting on the reporter for more information.
- `ready-for-agent` — fully specified, ready for an AFK agent to pick up with no human context.
- `ready-for-human` — requires human implementation (judgment calls, external access, design decisions, manual testing).
- `wontfix` — will not be actioned.

A triaged issue carries exactly one category label and one state label.

## Notes

- "Closing" in Linear = post the explanatory comment, then move the issue to the
  `Canceled` workflow state. `wontfix` is a label, not a state.
- There is an older `Agent Ready` label in the workspace that overlaps `ready-for-agent`;
  ignore it — `ready-for-agent` is the one to use.
