# deepcore-skills

Jeff's personal [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces).
It distributes one plugin — `deepcore-skills` — so your skills are available **locally**, in
**remote/scheduled routines**, and across **any repository**, all from this one git repo.

## Structure

```
.
├── .claude-plugin/
│   └── marketplace.json          # the marketplace catalog (lists the plugin)
└── plugins/
    └── deepcore-skills/
        ├── .claude-plugin/
        │   └── plugin.json        # the plugin manifest
        └── skills/                # your skills go here (one folder each)
            └── <skill-name>/
                └── SKILL.md
```

## Use it

Add the marketplace once (per machine / per environment), then install the plugin:

```shell
/plugin marketplace add JeffThorslund/claude-skills
/plugin install deepcore-skills@deepcore-skills
```

Or from the CLI (useful for remote routines / CI):

```bash
claude plugin marketplace add JeffThorslund/claude-skills
claude plugin install deepcore-skills@deepcore-skills
```

Because it's installed at the user/plugin level (not committed into a project's `.claude/`),
the skills work in **every repo** you open and in any remote routine that runs the same install.

## Add a new skill

1. Create `plugins/deepcore-skills/skills/<skill-name>/SKILL.md` with YAML frontmatter:

   ```markdown
   ---
   description: One line telling Claude when to use this skill.
   ---

   Instructions for the skill go here.
   ```

2. Commit and push. No version bump needed — `version` is intentionally omitted from
   `plugin.json`, so **every commit is treated as a new version** automatically.

3. In any session, refresh and you'll get the update:

   ```shell
   /plugin marketplace update deepcore-skills
   ```

Skills are namespaced by the plugin, so they're invoked as `/deepcore-skills:<skill-name>`.

## Validate before pushing

```bash
claude plugin validate .                          # checks marketplace.json
claude plugin validate ./plugins/deepcore-skills  # checks the plugin + skill frontmatter
```
