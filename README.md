# Agent Skills

Practical skills for AI coding agents, developed from real failures in client projects (PHP, WordPress, WooCommerce, AWS), focused on preserving behavior and capturing deployment dependencies. Each skill is a folder with a `SKILL.md` that tells the agent when to use it and what to do.

They use the `SKILL.md` skill-folder format that Claude Code reads. The instructions use plain wording ("read the file", "append a row") rather than tool names from any one agent, so they should port to other agents that read skill folders, but that hasn't been tested.

**Status: early release.** These skills encode safeguards that helped on real projects. They haven't yet been measured with repeated with-and-without runs, so treat them as checklists that reduce risk, not guarantees.

## Skills

| Skill | What it does |
|---|---|
| [evidence-first-debugging](evidence-first-debugging/) | Has the agent label hypotheses and confirm a cause with evidence before fixing it, and check caches, dead duplicate files and every code path. |
| [refactor-deletion-check](refactor-deletion-check/) | Before a refactor is committed, maps each removed hidden field, handler, hook and include to where it went and how that was verified. |
| [update-deployment-changelog](update-deployment-changelog/) | Writes each session's changes into a deployment changelog, with a checklist of the manual steps a deploy needs. |
| [commit-session-changes](commit-session-changes/) | Commits only this session's changes, down to individual hunks in shared files, with a message in the repo's style. |
| [time-logger](time-logger/) | Appends billable time entries to a CSV ledger, confirming estimates with you before writing. |

Several of these came from real incidents: a debugging session that lost hours to an unproven theory, and a "UI cleanup" commit that silently dropped a hidden field and a sync feature.

## Featured: the Manual Actions checklist

Git tracks code changes. It doesn't track the coupon you created in the admin panel, the plugin setting you toggled, or the environment variable you added on staging. Those are the steps that get forgotten when the change goes to production, and the feature quietly breaks.

`update-deployment-changelog` makes the agent list them every time, as a checklist you tick off during the deploy:

```markdown
### Manual Actions
- [ ] **Create the `SIBLING10` coupon** (10% off, percentage discount) in WooCommerce on staging: the code applies it by code name and does nothing if it's missing.
- [ ] **Create the `SIBLING10` coupon** on production, with the same settings.
- [ ] **Verify:** whether the page cache needs clearing after deploy for the cart to show the discount line.
```

Each environment gets its own item, and anything nobody has confirmed is marked **Verify:** so the agent doesn't present a guess as a requirement. If a session needed no manual steps, the entry says `- None`, so you know it was checked rather than skipped. See a [full example changelog](update-deployment-changelog/examples/DEPLOYMENT-CHANGELOG.md).

## Install

**Claude Code.** Copy a skill folder into your personal skills directory, or into a project's `.claude/skills/` to share it with everyone on that repo:

```bash
git clone https://github.com/sustaintopervail/agent-skills.git
cp -r agent-skills/update-deployment-changelog ~/.claude/skills/
cp -r agent-skills/time-logger ~/.claude/skills/
# or all of them:
# cp -r agent-skills/*/ ~/.claude/skills/
```

**Other agents.** Put the folder wherever your agent loads skills from, or paste the body of `SKILL.md` into its project instructions.

## Configure

Both skills write to a file in the project root by default:

| Skill | Default file |
|---|---|
| update-deployment-changelog | `DEPLOYMENT-CHANGELOG.md` |
| time-logger | `time-log.csv` |

To change the location for a project, add a line to its agent instructions (`CLAUDE.md`, `AGENTS.md` or equivalent):

```
Deployment changelog: docs/deploy-log.md
Time log: docs/billing/time-log.csv
```

## Usage

Just ask in plain language:

- "Update the deployment changelog with what we just did."
- "Log 1.5 hours for the subscription bundle work."
- "Log time for this." (the agent estimates and asks you to confirm)

## Contributing

Issues and pull requests are welcome. A good skill does one job, says when it should trigger, and includes a worked example.

## License

[MIT](LICENSE)
