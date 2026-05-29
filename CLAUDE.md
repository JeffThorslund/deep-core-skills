# CLAUDE.md

Guidance for working on **this repo** — Jeff's `deep-core-skills` Claude Code plugin marketplace. This file is about *authoring* the marketplace; the skills themselves carry their own runtime instructions in each `SKILL.md`.

## What this repo is

A plugin marketplace distributing one plugin (`deep-core-skills`). Skills live at `plugins/deep-core-skills/skills/<name>/SKILL.md`, one folder per skill (plus any supporting `.md` files the skill references via relative links). See `README.md` for install/usage.

## Conventions for skills here

- **Issue tracker is Linear.** The tracker skills (`to-issues`, `to-prd`, `triage`) target Linear (team `Engineering`) via the Linear MCP server — not `gh`/`glab`. Any new tracker-touching skill should follow the same pattern.
- **Triage labels live in one file.** The canonical label vocabulary and meanings are defined once in `plugins/deep-core-skills/skills/LABELS.md`. The three tracker skills inline it at runtime via `` !`cat ${CLAUDE_SKILL_DIR}/../LABELS.md` `` (with `allowed-tools: Bash(cat *)` in frontmatter). **Edit labels there, not in the skills.** Any new label-aware skill should inline the same file instead of restating the list. The labels already exist in the `Engineering` team — never add label-creation logic.
- **Domain docs are single-context.** Skills assume a root `CONTEXT.md` + `docs/adr/`, created lazily by `grill-with-docs`.

## Provenance

Most skills are adapted from [mattpocock/skills](https://github.com/mattpocock/skills) (the `engineering` set). Matt's repo has a `setup-matt-pocock-skills` skill that scaffolds per-repo tracker/label/domain config; **we deliberately did not copy it** — its decisions are baked directly into our skills instead. Do not re-introduce references to `/setup-matt-pocock-skills`.

## Editing

- No `version` in `plugin.json` is intentional — every commit is treated as a new version.
- Validate before pushing: `claude plugin validate .` (marketplace) and `claude plugin validate ./plugins/deep-core-skills` (plugin + skill frontmatter).
