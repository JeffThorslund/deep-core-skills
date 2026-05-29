---
name: review-architecture
description: Autonomously scan a codebase for the single most significant systemic architectural weakness, framed through the shared architecture lens (module depth, seams, locality, leverage). Produces one severity-ranked structured finding. Use for a deliberate, hands-off architectural pass that a routine can hand to a tracker skill. Analysis-only — does not file tickets itself.
---

# Review Architecture

You identify the **one** dominant systemic architectural weakness in a codebase — a shallow-module / leaked-seam / poor-locality problem that spans modules, not an isolated style nit. You run autonomously: no grilling loop, no human in the loop. You survey, hypothesise, confirm, rank, and emit a single structured finding.

This is the **autonomous** sibling of [`improve-codebase-architecture`](../improve-codebase-architecture/SKILL.md). That skill is interactive — it presents candidates as an HTML report and drops into a grilling conversation. This one does a deliberate hands-off pass and stops at a finding. Both read the **same** architecture lens, so their vocabulary and principles never diverge.

## The architecture lens

Read both before concluding — they are the single source of truth for *what* you hunt for and *how* you reason about it:

- **[ARCHITECTURE-LANGUAGE.md](../ARCHITECTURE-LANGUAGE.md)** — the terms (module, interface, depth, seam, adapter, leverage, locality) and the principles (the deletion test; the interface is the test surface; one adapter = hypothetical seam, two = real). Use these terms exactly — don't drift into "component," "service," "API," or "boundary."
- **[ARCHITECTURE-DEEPENING.md](../ARCHITECTURE-DEEPENING.md)** — dependency categories and how a deepened module is tested across its seam. Use it to judge whether a weakness is *fixable* and to shape the suggested direction.

If the repo has a root `CONTEXT.md` or `docs/adr/`, read them too: the domain language names good seams, and ADRs record decisions you should not re-litigate. Don't surface a finding that an ADR already settled unless the friction is real enough to warrant reopening it — and say so explicitly if you do.

## Scope: analysis, not filing

This skill **produces a finding**; it does **not** create tickets. Filing, labelling, dedup, and escalation are routine concerns that delegate to the tracker skills ([`to-prd`](../to-prd/SKILL.md) / [`to-issues`](../to-issues/SKILL.md) / [`triage`](../triage/SKILL.md)), which own the Linear conventions and the canonical label vocabulary in [LABELS.md](../LABELS.md). Keeping filing out of here means the Linear logic stays single-sourced.

- Do **not** call Linear MCP tools, apply labels, or invent an `architecture-review` label.
- Do **not** decide a dedup heuristic or an escalation policy here — those are instance-specific and live in the routine that invokes this skill.
- **Do** emit the finding in the structured shape below so a routine (or the user) can hand it to a tracker skill cleanly.

## What you are looking for

Hunt for **one** dominant systemic concern per run — the kind of issue that, left alone, compounds into change amplification, poor locality, or an untestable seam across the system. You are looking for patterns that span modules, layers, or subsystems.

A useful test: would the fix require reshaping a module's interface, relocating a seam, or applying a deepening across many call sites — such that a senior engineer doing it would touch 3+ modules and explain a design rationale in the PR? If yes, it's in scope.

Explicitly do **not** surface:

- A single badly named function, or one local duplication (not systemic).
- A shallow module with exactly one caller and no second adapter in sight (a hypothetical seam, not a real one — see the lens).
- Cosmetic or style issues a linter would catch.
- Anything an existing ADR already settled, unless the friction genuinely warrants reopening it.

A weak finding is worse than none. If nothing meets the bar, say so plainly and exit.

## Methodology

Investigate before concluding — don't pattern-match from a glance.

1. **Survey.** Map the top-level structure: directory layout, primary entry points, module boundaries, build/config, major dependencies. Prefer the `Explore` subagent or `Glob`/`Read`; hold off on `Grep` until you have a hypothesis.
2. **Hypothesise.** From the survey, pick 2–3 candidate weaknesses. For each, name the lens concept it violates (shallow module, leaked seam, poor locality, change amplification) and predict what the symptom looks like in code.
3. **Test.** Use `Grep` and targeted `Read` to confirm or kill each candidate. Apply the **deletion test** to anything you suspect is shallow. Confirm the weakness is *systemic* (multiple sites / callers) rather than local. Use `git log` to detect tactical drift on a hot surface if relevant.
4. **Rank and pick one.** Choose the finding with the highest blast radius: most change amplification, most modules affected, most likely to bite during planned work. On a tie, prefer the one whose deepening is more concrete (a clear dependency category and seam from [ARCHITECTURE-DEEPENING.md](../ARCHITECTURE-DEEPENING.md)).
5. **Evidence.** Collect 3–6 specific `file:line` references that demonstrate the pattern. Vague claims are useless to whoever triages.

## Output

Emit your result as a structured finding so it can be handed to a tracker skill or reviewed directly. Use the shared lens vocabulary throughout.

```
## Finding
<one-line weakness statement at the design level — specific and plain, e.g.
 "Order pricing logic spread thin across 6 handlers with no deep module to own it",
 NOT "Shallow modules">

## Lens concept
Which architecture-lens idea this violates (shallow module / leaked seam / poor
locality / change amplification), briefly, in your own words.

## Evidence
- `path/to/file.ts:42` — what's happening here
- `path/to/other.ts:118` — what's happening here
- (3–6 entries)

## Impact
What this costs the team, concretely: change amplification across N modules, blocks
workflow Y, untestable through the current interface. In terms of locality and leverage.

## Suggested direction
A direction, not a prescription. Name the deepening: what module would own the logic,
where the seam sits, the dependency category and adapter strategy (see the lens). 2–4
sentences. The right fix may differ once a human investigates.

## Severity
Critical / High / Medium / Low — one-line justification grounded in blast radius and
how soon it will bite.

## Confidence
Low / Medium / High, with a one-line reason. Lower confidence is fine and useful — flag
what would need verifying.
```

After the run, also summarise in chat:

- **Finding** — one paragraph.
- **Considered and discarded** — the other hypotheses you tested and why they didn't make the cut.

If nothing met the bar, say so plainly.

## Tone

Direct. You're advising a technical lead, not writing a textbook. Use the codebase's own domain vocabulary for the subject and the architecture lens for the diagnosis. Name a principle when it sharpens the point, then move on — don't pad.
