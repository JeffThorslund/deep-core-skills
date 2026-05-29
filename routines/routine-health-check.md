# Routine: Health Check

Periodic sweep of the connected monitoring/observability services, producing a severity-ranked
situational report. Wraps the `health-check` skill and adds the operator config the skill leaves
out.

## Invoke

Run **`/deep-core-skills:health-check`**. It discovers the connected monitoring MCP servers (error
tracking, hosting/deploys, database, etc.), polls them, and produces a four-section report: active
incidents, errors & warnings, performance anomalies, resolved items — ranked by severity.

The skill is strictly **read-only**: it reports, it never acknowledges alerts, resolves incidents,
files tickets, or touches the codebase. This routine supplies the scope and baseline.

## Scope

`<which projects / orgs / services to poll, and any thresholds>` — set before running. The skill
sweeps whatever is connected if scope is omitted, but naming the projects keeps the report focused.

## Cadence

Hourly is the typical cadence for a health sweep. Pass the cadence as the baseline window (below) so
"new since last check" lines up with how often the routine actually runs.

## Baseline window

The skill needs a "last check" baseline for *new* and *resolved* items. **Pass the window
explicitly** — set it to match this routine's cadence (e.g. last 1 hour for an hourly run). The
skill defaults to the last 1 hour and states the window it used if none is passed.

## On report → report only (no ticket)

This routine does **not** file tickets. Health Check is situational awareness; turning a finding
into tracked work is a separate, deliberate decision. If a sweep surfaces something worth tracking,
that's a follow-up — chain into `/deep-core-skills:to-issues` or `/deep-core-skills:triage`
**only** when a human (or a higher-level routine) decides to, not automatically.

## Escalation

If the report contains an **active incident** or a Critical-severity item, surface it at the top of
the run output unmistakably so it isn't buried under routine green status. Otherwise the four-section
report stands on its own.

## Output

The skill's report is the output. If nothing meets the bar across all services, it confirms **"All
systems nominal"** and names which services it checked and the window — so green is trustworthy, not
an empty result from a service that was actually unreachable.
