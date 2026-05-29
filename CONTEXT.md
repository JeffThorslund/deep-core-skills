# deep-core-skills

The ubiquitous language of this marketplace: the skills it distributes and the
**agent pipeline** those skills compose into. Terms below are the canonical
vocabulary — use them verbatim in skills, routines, and ADRs.

The pipeline confines autonomy to **proposing** work (scheduled reviews that file
findings). The judgment-heavy middle — triage and slicing — stays human: the
maintainer is the stage gate. See `docs/adr/0001` and `docs/adr/0002`.

## Language

### Building blocks

**Skill**:
A unit of capability with its own runtime instructions, living at
`plugins/deep-core-skills/skills/<name>/SKILL.md`. Carries the *meat* — how to do
the work.
_Avoid_: command, prompt, agent (a skill is invoked *by* an agent, it is not one).

**Routine**:
A thin scheduled-agent wrapper in `routines/` that invokes one or more skills in a
fixed order and supplies the instance-specific config a skill omits — scope,
cadence, dedup, escalation. Carries the *wrapper*, never the meat.
_Avoid_: job, cron, task, workflow.

**Interactive skill**:
A skill that converses with a maintainer — waits for direction, quizzes, iterates
to approval. The `improve-codebase-*`, `triage`, `to-issues`, `tdd` skills.
_Avoid_: manual, HITL-skill.

**Autonomous skill**:
A skill that takes one pass and stops with a result, never waiting for a human. The
`review-*` skills, and `health-check`. The repo convention pairs each review domain
as one interactive + one autonomous sibling sharing a lens.
_Avoid_: headless, unattended, AFK-skill.

### The pipeline

**Pipeline**:
The end-to-end flow: scheduled review skills file findings → every ticket lands as
`needs-triage` → the maintainer triages (and slices, if needed) by hand → approved
tickets sit in `ready-for-agent`. Autonomy proposes; the maintainer disposes.
_Avoid_: flow, system, automation.

**Stage gate**:
The maintainer, triaging manually. Deciding *what's worth doing* and *how to slice
it* is the judgment the pipeline deliberately keeps human (ADR-0001). Autonomy never
assigns a state on its own.
_Avoid_: gatekeeper, approver.

**State label**:
One of the mutually-exclusive Linear labels marking where a ticket sits in triage.
A triaged ticket carries exactly one. Canonical set in
[LABELS.md](plugins/deep-core-skills/skills/LABELS.md).
_Avoid_: status, stage, phase.

**Category label**:
`bug` or `enhancement` — *what kind* of ticket, orthogonal to state. Canonical in
[LABELS.md](plugins/deep-core-skills/skills/LABELS.md).
_Avoid_: type, kind.

**Finding**:
The single highest-impact issue a `review-*` skill emits in one pass (or nothing).
Analysis-only — the skill never files it. A routine wraps the skill and files the
finding as **one** `needs-triage` ticket; it does not split or self-triage.
_Avoid_: result, report, issue (until it's filed as a ticket).

**Slice**:
A vertical, tracer-bullet ticket cut end-to-end through every layer, produced by the
maintainer running `to-issues` by hand when a ticket is too big for one change.
Slicing is a manual judgment call, never automated.
_Avoid_: sub-task, chunk, horizontal slice.

**AFK / HITL**:
Whether a ticket can be implemented away-from-keyboard with no human context
(`ready-for-agent`) or requires a human (`ready-for-human`). The maintainer assigns
this at the stage gate.
_Avoid_: auto/manual, autonomous/interactive (those describe skills, not tickets).
