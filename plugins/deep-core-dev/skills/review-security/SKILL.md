---
name: review-security
description: Autonomously scan a codebase for the single most significant systemic security weakness, framed through the shared security lens (Saltzer & Schroeder / STRIDE / OWASP). Produces one severity-ranked structured finding. Use for a deliberate, hands-off security pass that a routine can hand to a tracker skill. Analysis-only — does not file tickets itself.
allowed-tools: Read, Glob, Grep, Bash
---

# Review Security

You identify the **one** dominant systemic security weakness in a codebase — a design-level issue that spans the system, not an isolated bug. You run autonomously: no grilling loop, no human in the loop. You survey, map trust boundaries, hypothesise, confirm, rank, and emit a single structured finding.

This is the **autonomous** sibling of [`improve-codebase-security`](../improve-codebase-security/SKILL.md). That skill is interactive — it presents candidate hardenings and grills the chosen one. This one does a deliberate hands-off pass and stops at a finding. Both read the **same** security lens, so their vocabulary and stance never diverge.

## The security lens

Read it before concluding — it is the single source of truth for *what* you look for and *how* you reason about it:

- **[SECURITY-LENS.md](../SECURITY-LENS.md)** — the defensive stance, what counts as systemic, the trust-boundary checklist, and the diagnostic lenses (Saltzer & Schroeder design principles, STRIDE per boundary, OWASP Top 10 as a pattern library, modern LLM/agent surfaces), plus how to rank a finding.

Apply it; don't restate it here.

## Scope: analysis, not filing

This skill **produces a finding**; it does **not** create tickets. Filing, labelling, dedup, and escalation are routine concerns that delegate to the tracker skills ([`to-prd`](../to-prd/SKILL.md) / [`to-issues`](../to-issues/SKILL.md) / [`triage`](../triage/SKILL.md)), which own the Linear conventions and the canonical label vocabulary in [LABELS.md](../LABELS.md). Keeping filing out of here means the Linear logic stays single-sourced.

- Do **not** call Linear MCP tools, apply labels, or invent a `review-security` label.
- Do **not** decide a dedup heuristic or a "Critical escalation" policy here — those are instance-specific and live in the routine that invokes this skill.
- **Do** emit the finding in the structured shape below so a routine (or the user) can hand it to a tracker skill cleanly.

## Methodology

Investigate before concluding — don't pattern-match from a glance.

1. **Survey.** Map trust boundaries first: where untrusted input enters, where sensitive operations happen, the auth model, what tenants/users are separated and how. Read routes, middleware, RLS policies, service-to-service auth. Use `Glob` and `Read`.
2. **Identify trust boundaries.** List them explicitly using the checklist in [SECURITY-LENS.md](../SECURITY-LENS.md) (untrusted ↔ trusted, tenant ↔ tenant, user ↔ admin, client ↔ server, external ↔ internal, LLM output ↔ executable surface).
3. **Hypothesise.** Pick 2–3 candidates. Name the principle (S&S) or threat (STRIDE) and predict the symptom.
4. **Test.** `Grep` and `Read` to verify. Confirm systemic (multiple sites/boundaries) vs local. Check `git log` if tactical drift is suspected on a security-critical surface.
5. **Rank and pick one.** Use the ranking rule in the lens: **systemic reach × exploitability × sensitivity of what's at risk**; on a tie, prefer the chokepoint/policy fix.
6. **Evidence.** Collect 3–6 specific `file:line` references. Describe the weakness defensively — the design gap, not an exploit recipe.

A weak finding is worse than none. If nothing meets the bar, say so plainly and exit.

## Output

Emit your result as a structured finding so it can be handed to a tracker skill or reviewed directly. Use defensive framing throughout.

```
## Finding
<one-line weakness statement at the design level — specific and plain, e.g.
 "Authorization checks scattered across 14 API handlers with no central enforcement",
 NOT "Broken Access Control">

## Principle / threat
The S&S principle violated, the STRIDE category, and/or the OWASP class — briefly, in your own words.

## Trust boundary
Which boundary this affects (e.g. "untrusted user input → server-side query construction",
"tenant A → tenant B isolation in shared cache").

## Evidence
- `path/to/file.ts:42` — what design gap is here
- `path/to/other.ts:118` — what design gap is here
- (3–6 entries)

## Impact
What an attacker could plausibly affect — at the level of data classes / privilege levels / scope,
NOT exploit steps. E.g. "any authenticated user could read other tenants' project metadata".

## Severity
Critical / High / Medium / Low — one-line justification grounded in exploitability, sensitivity of
data at risk, blast radius, and prerequisites.

## Suggested direction
A direction, not a prescription. Prefer chokepoints, policy enforcement, and secure-by-default
refactors over "add a check here". 2–4 sentences.

## Confidence
Low / Medium / High, with a one-line reason. Lower confidence is fine and useful — flag what
would need to be verified.
```

If a finding is **genuinely Critical, exploitable, and currently-live** (active RCE, auth bypass, hardcoded production secrets in the repo, public PII exposure), **lead your chat summary with it** and make the severity unmistakable — don't let it read like a routine finding. Whether and how that becomes an escalated ticket is the routine's policy, not this skill's.

After the run, also summarise in chat:
- **Finding** — one paragraph.
- **Considered and discarded** — the other hypotheses you tested and why they didn't make the cut.

## Tone

Direct, defensive, non-alarmist. You are a senior engineer pointing out where the design will hurt later, not a pentester writing a report. Use the codebase's vocabulary. Cite principles by name when useful, then move on.
