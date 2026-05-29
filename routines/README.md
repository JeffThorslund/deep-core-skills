# Routines

Version-controlled source for the **routines** Jeff runs as scheduled Claude agents. A routine is a thin orchestrator: it invokes one or more skills (from the `deep-core-skills` plugin) in a set order and supplies the *instance-specific* config that deliberately does **not** live in a skill — scope, cadence, dedup policy, severity/escalation handling, and which tracker skill the findings chain into.

## Why these live here

The skills carry the *meat* (how to do the work); routines carry the *how-to-run-it-as-a-routine* wrapper. Keeping the routines here means they're version-controlled and reviewable even though they ultimately get **copied verbatim into the scheduled-agent config** (which is not itself in this repo).

## How they reference skills

Routines reference skills by their **installed invocation name** — e.g. `/deep-core-skills:review-architecture` — never by filesystem path. The plugin resolves that name wherever it's installed, so a routine keeps working after it's copied out of this repo. (A filesystem link like `../plugins/.../SKILL.md` would break on copy.)

**Prerequisite:** the `deep-core-skills` marketplace + plugin must be installed in the environment the routine runs in:

```bash
claude plugin marketplace add JeffThorslund/deep-core-skills
claude plugin install deep-core-skills@deep-core-skills
```

## The routines

- **[routine-review-architecture.md](./routine-review-architecture.md)** — autonomous architecture pass → files a Linear ticket.
- **[routine-review-security.md](./routine-review-security.md)** — autonomous security pass → files a Linear ticket.
- **[routine-health-check.md](./routine-health-check.md)** — monitoring/observability sweep → report only (no ticket).

## Editing a routine

Fill the `Scope` line for the target repo/project before pasting into a scheduled agent. Everything else is ready to run. The analysis skills are deliberately read-only and never file tickets themselves — the routine owns the filing decision (see each routine's `On finding` step).
