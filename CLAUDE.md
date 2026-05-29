# CLAUDE.md

Guidance for working on **this repo** — Jeff's `deep-core-skills` Claude Code plugin marketplace. This file is about *authoring* the marketplace; the skills themselves carry their own runtime instructions in each `SKILL.md`.

## What this repo is

A plugin marketplace distributing one plugin (`deep-core-skills`). Skills live at `plugins/deep-core-skills/skills/<name>/SKILL.md`, one folder per skill (plus any supporting `.md` files the skill references via relative links). Shared reference docs (`LABELS.md`, the architecture lens, `SECURITY-LENS.md`) sit flat at the `skills/` root. A separate top-level `routines/` dir — **outside** the plugin — holds version-controlled routine wrappers that get copied verbatim into Jeff's scheduled-agent config. See `README.md` for install/usage and `routines/README.md` for the routine convention.

## Conventions for skills here

- **Issue tracker is Linear.** The tracker skills (`to-issues`, `to-prd`, `triage`) target Linear (team `Engineering`) via the Linear MCP server — not `gh`/`glab`. Any new tracker-touching skill should follow the same pattern.
- **Triage labels live in one file.** The canonical label vocabulary and meanings are defined once in `plugins/deep-core-skills/skills/LABELS.md`. The three tracker skills (`to-issues`, `to-prd`, `triage`) reference it with a relative markdown link — `[LABELS.md](../LABELS.md)` — and instruct the agent to read it and apply the strings verbatim. **Edit labels there, not in the skills.** Any new label-aware skill should link the same file instead of restating the list. The labels already exist in the `Engineering` team — never add label-creation logic.
- **Cross-references are relative markdown links.** Skills reference each other and their shared docs with plain relative links (`[LABELS.md](../LABELS.md)`, `[ARCHITECTURE-LANGUAGE.md](../ARCHITECTURE-LANGUAGE.md)`), not `` !`cat …` `` runtime inlines or absolute paths. Keep every link relative and resolving. (We previously inlined `LABELS.md` via `` !`cat` `` with `allowed-tools: Bash(cat *)`; that was replaced with links — don't reintroduce the `cat` form.)
- **Review lenses are single-sourced.** Each review domain has one shared lens at the skills root, linked by both its interactive and autonomous skill so the pair never drifts. Edit the lens there, not in either skill:
  - **Architecture** — `ARCHITECTURE-LANGUAGE.md` (vocabulary + principles) + `ARCHITECTURE-DEEPENING.md` (how to deepen across a seam), shared by `improve-codebase-architecture` (interactive) and `review-architecture` (autonomous).
  - **Security** — `SECURITY-LENS.md` (defensive stance + S&S/STRIDE/OWASP), shared by `improve-codebase-security` (interactive) and `review-security` (autonomous).
  Any new review domain should follow the same shape: one shared lens, an interactive `improve-codebase-*` skill, an autonomous `review-*` skill.
- **Report-craft is single-sourced too.** The interactive `improve-codebase-*` skills render an HTML report; the domain-neutral craft (scaffold, card structure, diagram patterns, tone) lives once in `HTML-REPORT.md` at the skills root. Each skill links it and supplies only its own legend, badge taxonomy, and controlled vocabulary inline. Keep the two interactive skills parallel: their report sections should differ **only** in those domain specifics. (Architecture additionally has `INTERFACE-DESIGN.md`, the "Design It Twice" interface pattern — deliberately architecture-only; security has no analog and should not invent one.)
- **Analysis skills don't file tickets.** The "review archetype" — `health-check`, `review-architecture`, `review-security` — produces structured findings (or a report) and stops. Filing, labelling, dedup, and escalation are *routine* concerns delegated to the tracker skills, keeping Linear logic single-sourced. Don't add Linear calls or label-creation to an analysis skill.
- **Routines wrap skills by invocation name.** Routine wrappers in `routines/` reference skills as `/deep-core-skills:<skill>` (never by file path — they're copied out of this repo) and carry the instance-specific config the skill omits (scope, cadence, dedup, escalation, the tracker-skill chain). Keep the meat in the skill; keep only orchestration in the routine.
- **Domain docs are single-context.** Skills assume a root `CONTEXT.md` + `docs/adr/`, created lazily by `grill-with-docs`.

## Provenance

Most skills are adapted from [mattpocock/skills](https://github.com/mattpocock/skills) (the `engineering` set). Matt's repo has a `setup-matt-pocock-skills` skill that scaffolds per-repo tracker/label/domain config; **we deliberately did not copy it** — its decisions are baked directly into our skills instead. Do not re-introduce references to `/setup-matt-pocock-skills`.

**Intentional divergences from upstream (don't "fix" these in a drift audit):**

- `improve-codebase-architecture` no longer matches Matt's verbatim: its `LANGUAGE.md`/`DEEPENING.md` were lifted to the skills root as the shared architecture lens (`ARCHITECTURE-LANGUAGE.md` / `ARCHITECTURE-DEEPENING.md`) and the skill's inline glossary was collapsed to a link. The lens content is unchanged — only its location and the references moved.
- `review-architecture` is **original** (no upstream counterpart) — the autonomous, analysis-only sibling of `improve-codebase-architecture`, sharing the same lens.
- `health-check` is original (no upstream counterpart).
- `review-security` was the original `security-review` skill, renamed for parallelism with `review-architecture`; its lenses were lifted to the shared `SECURITY-LENS.md`. `improve-codebase-security` is **original** — the interactive sibling, mirroring `improve-codebase-architecture`'s flow. Both are original (no upstream counterpart).
- `to-issues`, `to-prd`, `triage` diverge only by the Linear/LABELS adaptations above — no other behavioural drift.

## Editing

- No `version` in `plugin.json` is intentional — every commit is treated as a new version.
- Validate before pushing: `claude plugin validate .` (marketplace) and `claude plugin validate ./plugins/deep-core-skills` (plugin + skill frontmatter).
