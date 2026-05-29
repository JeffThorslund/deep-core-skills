# deep-core-skills

Jeff's personal [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces).
It distributes one plugin — `deep-core-skills` — so your skills are available **locally**, in
**remote/scheduled routines**, and across **any repository**, all from this one git repo.

## Structure

```
.
├── .claude-plugin/
│   └── marketplace.json          # the marketplace catalog (lists the plugin)
└── plugins/
    └── deep-core-skills/
        ├── .claude-plugin/
        │   └── plugin.json        # the plugin manifest
        └── skills/                # your skills go here (one folder each)
            └── <skill-name>/
                └── SKILL.md
```

## Use it

Add the marketplace once (per machine / per environment), then install the plugin:

```shell
/plugin marketplace add JeffThorslund/deep-core-skills
/plugin install deep-core-skills@deep-core-skills
```

Or from the CLI (useful for remote routines / CI):

```bash
claude plugin marketplace add JeffThorslund/deep-core-skills
claude plugin install deep-core-skills@deep-core-skills
```

Because it's installed at the user/plugin level (not committed into a project's `.claude/`),
the skills work in **every repo** you open and in any remote routine that runs the same install.

## Skills

Skills currently in the plugin. (Adapted from [mattpocock/skills](https://github.com/mattpocock/skills).)

- **[grill-with-docs](./plugins/deep-core-skills/skills/grill-with-docs/SKILL.md)** — Grilling session that challenges your plan against the existing domain model, sharpens terminology, and updates `CONTEXT.md` and ADRs inline.
- **[improve-codebase-architecture](./plugins/deep-core-skills/skills/improve-codebase-architecture/SKILL.md)** — Find deepening opportunities in a codebase, informed by the domain language in `CONTEXT.md` and the decisions in `docs/adr/`.
- **[tdd](./plugins/deep-core-skills/skills/tdd/SKILL.md)** — Test-driven development with a red-green-refactor loop. Builds features or fixes bugs one vertical slice at a time.
- **[to-issues](./plugins/deep-core-skills/skills/to-issues/SKILL.md)** — Break any plan, spec, or PRD into independently-grabbable issues using vertical slices.
- **[to-prd](./plugins/deep-core-skills/skills/to-prd/SKILL.md)** — Turn the current conversation context into a PRD and submit it to the issue tracker.
- **[triage](./plugins/deep-core-skills/skills/triage/SKILL.md)** — Triage issues through a state machine of triage roles.
- **[zoom-out](./plugins/deep-core-skills/skills/zoom-out/SKILL.md)** — Tell the agent to zoom out and give broader context or a higher-level perspective on an unfamiliar section of code.

### Conventions

The three issue-tracker skills — `to-issues`, `to-prd`, and `triage` — assume **[Linear](https://linear.app)** (team `Engineering`) as the tracker, driven through the Linear MCP server rather than a CLI. They share one canonical label vocabulary defined in [`plugins/deep-core-skills/skills/LABELS.md`](./plugins/deep-core-skills/skills/LABELS.md) — each skill inlines that file at runtime, so the labels and their meanings are written once. The doc-oriented skills (`grill-with-docs`, `improve-codebase-architecture`, `tdd`) read a root `CONTEXT.md` and `docs/adr/` when present and create them lazily otherwise.

## Add a new skill

1. Create `plugins/deep-core-skills/skills/<skill-name>/SKILL.md` with YAML frontmatter:

   ```markdown
   ---
   description: One line telling Claude when to use this skill.
   ---

   Instructions for the skill go here.
   ```

2. Commit and push. No version bump needed — `version` is intentionally omitted from
   `plugin.json`, so **every commit is treated as a new version** automatically.

3. In any session, refresh and update to pick up the change:

   ```shell
   /plugin marketplace update deep-core-skills   # refresh the catalog
   /plugin update deep-core-skills@deep-core-skills  # move installed plugin to latest commit
   ```

   (Background auto-update does both over time; the manual two-step is for getting a change immediately.)

Skills are namespaced by the plugin, so they're invoked as `/deep-core-skills:<skill-name>`.

## Validate before pushing

```bash
claude plugin validate .                           # checks marketplace.json
claude plugin validate ./plugins/deep-core-skills  # checks the plugin + skill frontmatter
```
