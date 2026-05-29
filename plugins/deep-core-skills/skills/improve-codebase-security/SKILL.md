---
name: improve-codebase-security
description: Find hardening opportunities in a codebase, informed by the domain language in CONTEXT.md and the decisions in docs/adr/. Use when the user wants to improve security posture, close systemic weaknesses, consolidate scattered authorization, or make trust boundaries explicit and enforceable. Interactive — presents candidates, then grills the chosen one.
---

# Improve Codebase Security

Surface systemic security friction and propose **hardening opportunities** — refactors that move the system toward secure-by-default: a chokepoint where authorization was scattered, a fail-safe default where a fall-through let requests through, a single enforced trust boundary where one had eroded. The aim is a design where the secure path is the easy path.

This is the **interactive** sibling of [`review-security`](../review-security/SKILL.md). That skill runs autonomously and emits one structured finding. This one explores with you, presents candidates as a report, and drops into a grilling conversation on the one you pick — recording decisions in the project's docs as they crystallise.

## The security lens

This skill and `review-security` share one stance and one set of diagnostic lenses — the **security lens**, single-sourced at the skills root so the two skills never drift:

- **[SECURITY-LENS.md](../SECURITY-LENS.md)** — the defensive stance (analyse, never weaponize), what counts as *systemic*, the trust-boundary checklist, and the lenses (Saltzer & Schroeder, STRIDE, OWASP Top 10, modern LLM/agent surfaces), plus how to rank candidates.

Read it before proposing anything. This skill is _informed_ by the project's domain model on top of that lens: the domain language names the assets worth protecting and the actors; ADRs record security decisions the skill should not re-litigate.

You analyse and recommend defensive direction. You never produce working exploit code or step-by-step attack instructions — describe the class of weakness and the hardening, not how to weaponize the gap.

## Process

### 1. Explore

Read the project's domain glossary and any ADRs in the area you're touching first — especially ADRs that record a deliberate security trade-off.

Then use the Agent tool with `subagent_type=Explore` to walk the codebase. Map **trust boundaries first** (per the lens), then note where the design fights secure-by-default:

- Where is authorization decentralised across many handlers instead of enforced at a chokepoint? (complete mediation)
- Where does the system **allow** by default on error or when unconfigured? (fail-safe defaults — middleware fall-throughs, RLS-off-by-default, default route handlers)
- Where do unrelated tenants/users share state, caches, or singletons that could leak across them? (least common mechanism)
- Where is the secure API harder to use than the unsafe one, so engineers route around it? (psychological acceptability)
- Where does untrusted input reach an executable surface (query construction, template, shell, LLM/tool prompt) without a mediating boundary?

Confirm each candidate is **systemic** (spans multiple sites/boundaries), not a single local mistake — the lens defines the bar.

### 2. Present candidates as an HTML report

Write a self-contained HTML file to the OS temp directory so nothing lands in the repo. Resolve the temp dir from `$TMPDIR`, falling back to `/tmp` (or `%TEMP%` on Windows), and write to `<tmpdir>/security-hardening-<timestamp>.html` so each run gets a fresh file. Open it for the user — `xdg-open <path>` on Linux, `open <path>` on macOS, `start <path>` on Windows — and tell them the absolute path.

The report uses **Tailwind via CDN** for layout and **Mermaid via CDN** for diagrams where a graph reliably communicates structure (trust-boundary diagrams, data-flow across a boundary, an authorization call graph). Each candidate gets a **before/after visualisation** showing the boundary as it is vs. as it would be with the chokepoint/default in place. Be visual, but stay defensive — diagram the design gap, never an attack runbook.

For each candidate, render a card with:

- **Trust boundary** — which boundary this concerns (from the lens checklist)
- **Weakness** — the systemic design gap, framed defensively, with the principle/threat it violates (S&S / STRIDE / OWASP)
- **Hardening** — plain-English description of the secure-by-default change (the chokepoint, the fail-safe default, the enforced boundary)
- **Evidence** — 3–6 `file:line` references demonstrating the pattern
- **Before / After diagram** — side-by-side, illustrating the boundary before and after the hardening
- **Severity** — Critical / High / Medium / Low, rendered as a badge, justified per the lens
- **Recommendation strength** — `Strong` / `Worth exploring` / `Speculative`, as a badge

End the report with a **Top recommendation** section: which hardening you'd tackle first and why. If anything is **genuinely Critical, exploitable, and currently-live**, lead with it unmistakably (per the lens) — don't let it read like a routine card.

**Use CONTEXT.md vocabulary for the domain, and [SECURITY-LENS.md](../SECURITY-LENS.md) vocabulary for the security reasoning.** Talk about "the Order intake boundary," not "the FooBarHandler."

**ADR conflicts**: if a candidate contradicts an existing ADR (e.g. a documented decision to trust a given input), only surface it when the risk genuinely warrants revisiting the ADR. Mark it clearly in the card (a warning callout: _"contradicts ADR-0007 — but worth reopening because…"_).

Do NOT propose the detailed remediation yet. After the file is written, ask the user: "Which of these would you like to explore?"

### 3. Grilling loop

Once the user picks a candidate, drop into a grilling conversation. Walk the design tree with them — the trust boundary, what crosses it, where the chokepoint should sit, what the fail-safe default is, what breaks if the boundary is enforced, how the secure path becomes the easy path.

Side effects happen inline as decisions crystallise (same discipline as [`grill-with-docs`](../grill-with-docs/SKILL.md)):

- **Naming a new boundary, chokepoint, or trust zone not in `CONTEXT.md`?** Add the term to `CONTEXT.md` (see [CONTEXT-FORMAT.md](../grill-with-docs/CONTEXT-FORMAT.md)). Create the file lazily if it doesn't exist.
- **Sharpening a fuzzy security term during the conversation?** Update `CONTEXT.md` right there.
- **User accepts a residual risk with a load-bearing reason, or rejects a hardening?** Offer an ADR: _"Want me to record this as an ADR so future security reviews don't re-suggest it?"_ Only offer when the reason would actually be needed by a future reviewer to avoid re-flagging the same gap. See [ADR-FORMAT.md](../grill-with-docs/ADR-FORMAT.md).
