---
name: health-check
description: Perform a health check across connected monitoring and observability services, reporting active incidents, new errors/warnings, performance anomalies, and recently-resolved items, ranked by severity. Use for a "what's the state of production" sweep. Read-only — never files tickets or touches the codebase.
---

# Health Check

Sweep the connected monitoring and observability services and produce a single, severity-ranked situational report. This skill is **read-only**: it reports, it does not act.

## Hard boundaries

These keep the skill harmonious with the rest of the plugin — do not cross them:

- **No Linear.** Do not create issues, apply labels, post comments, or read `LABELS.md`. Turning findings into tickets is a *routine* concern — chain this skill's report into `to-issues` or `triage` afterwards. Issue creation and the label vocabulary stay single-sourced in those skills.
- **No codebase work.** Do not explore the repo, read source, or reason about code design. That is what `security-review` / `improve-codebase-architecture` are for.
- **No mutations anywhere.** Acknowledging an alert, resolving an incident, or restarting a service is out of scope. Surface it; let a human decide.
- **No invented data.** Report only what the monitoring services actually return. If a service is unreachable or not connected, say so plainly rather than guessing.

## Which services to poll

Use whatever monitoring/observability MCP servers are connected this session. Discover them rather than assuming — search for tools by capability (errors, incidents, logs, performance, uptime). Common ones:

- **Error tracking** (e.g. Sentry-style): search recent issues/events, regressions, new error groups.
- **Hosting/deploys** (e.g. Vercel-style): runtime logs, deployment status, recent failed deploys.
- **Database** (e.g. Supabase-style): logs and advisors (security/performance warnings).
- **Anything else** exposing incident, alert, latency, or resource-usage data.

**Which specific projects/orgs/thresholds to check is instance-specific.** It belongs in the routine that invokes this skill (or in invocation arguments), not hardcoded here. If the user passed scope as an argument, honor it; otherwise sweep what's connected.

## The "last check" baseline

"New since the last check" and "resolved since the last check" need a baseline. Determine it in this order:

1. A timestamp/window passed as an argument by the caller (a routine usually supplies this).
2. Otherwise, default to the **last 1 hour** and say so explicitly in the report.

Never silently pick a window — state the window you used.

## Report format

Produce the four sections below, **prioritized by severity** (most severe first within each). For every issue include the **affected service** and a **direct link** when the source provides one.

### 1. Active incidents
Ongoing outages or incidents — severity and current status each.

### 2. Errors & warnings
New error or warning patterns since the baseline. Group by error signature; give counts and first/last-seen where available. Don't dump raw stack traces — summarize the pattern.

### 3. Performance anomalies
Unusual latency, error rates, throughput, or resource usage versus normal. Note the metric, the deviation, and the affected service.

### 4. Resolved items
Incidents or alerts that closed since the baseline — brief, so the reader sees what got better.

## If all is well

If nothing meets the bar across all services, confirm with a brief **"All systems nominal"** and note which services you checked and the time window — so the green status is trustworthy, not just an empty result from a service that was actually unreachable.
