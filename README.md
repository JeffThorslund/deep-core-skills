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
- **[health-check](./plugins/deep-core-skills/skills/health-check/SKILL.md)** — Read-only sweep of connected monitoring/observability services: active incidents, new errors, performance anomalies, resolved items, ranked by severity. No Linear, no codebase.
- **[improve-codebase-architecture](./plugins/deep-core-skills/skills/improve-codebase-architecture/SKILL.md)** — Interactive: find deepening opportunities in a codebase, present them as an HTML report, then grill the chosen one. Informed by the domain language in `CONTEXT.md` and the decisions in `docs/adr/`.
- **[review-architecture](./plugins/deep-core-skills/skills/review-architecture/SKILL.md)** — Autonomous, analysis-only sibling of `improve-codebase-architecture`: scans for the single most significant systemic architectural weakness and emits one structured finding. Shares the same architecture lens; filing is delegated to the tracker skills.
- **[security-review](./plugins/deep-core-skills/skills/security-review/SKILL.md)** — Scan for the single most significant systemic security weakness via Saltzer & Schroeder / STRIDE / OWASP. Produces severity-ranked findings; filing is delegated to the tracker skills.
- **[tdd](./plugins/deep-core-skills/skills/tdd/SKILL.md)** — Test-driven development with a red-green-refactor loop. Builds features or fixes bugs one vertical slice at a time.
- **[to-issues](./plugins/deep-core-skills/skills/to-issues/SKILL.md)** — Break any plan, spec, or PRD into independently-grabbable issues using vertical slices.
- **[to-prd](./plugins/deep-core-skills/skills/to-prd/SKILL.md)** — Turn the current conversation context into a PRD and submit it to the issue tracker.
- **[triage](./plugins/deep-core-skills/skills/triage/SKILL.md)** — Triage issues through a state machine of triage roles.
- **[zoom-out](./plugins/deep-core-skills/skills/zoom-out/SKILL.md)** — Tell the agent to zoom out and give broader context or a higher-level perspective on an unfamiliar section of code.

### Conventions

The three issue-tracker skills — `to-issues`, `to-prd`, and `triage` — assume **[Linear](https://linear.app)** (team `Engineering`) as the tracker, driven through the Linear MCP server rather than a CLI. They share one canonical label vocabulary defined in [`plugins/deep-core-skills/skills/LABELS.md`](./plugins/deep-core-skills/skills/LABELS.md) — each skill links that file and applies its labels verbatim, so the labels and their meanings are written once. The two architecture skills (`improve-codebase-architecture`, `review-architecture`) likewise share one **architecture lens** — [`ARCHITECTURE-LANGUAGE.md`](./plugins/deep-core-skills/skills/ARCHITECTURE-LANGUAGE.md) (vocabulary + principles) and [`ARCHITECTURE-DEEPENING.md`](./plugins/deep-core-skills/skills/ARCHITECTURE-DEEPENING.md) (how to deepen across a seam) — so the interactive and autonomous passes never drift. All cross-references between skills are plain relative markdown links. The doc-oriented skills (`grill-with-docs`, `improve-codebase-architecture`, `tdd`) read a root `CONTEXT.md` and `docs/adr/` when present and create them lazily otherwise.

## Skills vs. routines

A **skill** here is a reusable building block (version-controlled, in this repo). A **routine** is a thin
orchestrator that calls skills in a certain order on a schedule — it lives in your **non-version-controlled**
scheduled-agent config, not in this plugin. The goal: routines stay light, and any reusable substance lives in
a skill so it's written once.

Two consequences of that split, both load-bearing:

- **Analysis skills don't file tickets.** The *review archetype* — `health-check`, `security-review`, and
  `review-architecture` — *produces findings* and stops there. Turning findings into Linear issues is delegated to
  `to-issues` / `to-prd` / `triage`, which own the tracker conventions and the `LABELS.md` vocabulary. This keeps
  Linear logic single-sourced — an analysis skill never re-implements `create_issue` or label rules. A routine
  wires them together (e.g. `review-architecture` → `to-issues`, or `security-review` → `to-prd`).
- **Instance-specific config lives in the routine, not the skill.** Which monitoring project to poll, which
  severity escalates, dedup heuristics, the hourly cadence — none of that is hardcoded in a skill.

The review archetype splits *what* we look for from *how* it's delivered. `review-architecture` (autonomous,
analysis-only) and `improve-codebase-architecture` (interactive, grilling loop) are two deliveries over the **one**
shared architecture lens (`ARCHITECTURE-LANGUAGE.md` + `ARCHITECTURE-DEEPENING.md`) — edit the lens once, both
stay in sync.

When routine-wrapper skills *are* added to this plugin (they're still skills — Claude Code does not support a
separate "routine" type, and two-level `skills/` nesting is **not** discovered at runtime), prefix them
`routine-` (e.g. `routine-architecture-review`) so they group visually and read as orchestrators. Several
proposed routines deliberately have **no** new skill: an *autonomous architecture review* routine just chains
`review-architecture` into `to-issues`; an *hourly ticket implementer* chains `tdd`/`verify-and-ship`/`code-review`;
*needs-info tagging* is already owned by `triage`.

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
