---
name: update-deployment-changelog
description: Records the files changed, database changes and manual deployment steps from the current session at the top of a deployment changelog. Use when the user asks to update the changelog or deployment notes, or before shipping work to staging or production.
---

# Update Deployment Changelog

Keep a running deployment checklist in the project. Each session's work becomes one entry listing what changed in code, what changed in the database, and, most importantly, **what someone has to do by hand** on staging or production for the change to work.

Code is the easy part of a deploy because git already tracks it. What gets forgotten is the coupon that has to exist in the admin panel, the plugin setting that has to be toggled, or the environment variable that has to be added. The **Manual Actions** checklist exists to catch those.

## Configuration

**Changelog path:** `DEPLOYMENT-CHANGELOG.md` in the project root, unless the project says otherwise. (It is deliberately not `CHANGELOG.md`, which many projects already use for release notes.)

To use a different file, add a line like this to the project's agent instructions (`CLAUDE.md`, `AGENTS.md` or equivalent):

```
Deployment changelog: docs/deploy-log.md
```

Relative paths are resolved from the project root. If the user names a path in the conversation, that wins.

## Workflow

### 1. Collect what changed
- Review this session's conversation and actions for every file created, edited, renamed or deleted.
- If the project is a git repo, cross-check with `git status` and `git diff --stat` so nothing is missed.
- Note any database changes: migrations, schema edits, seed data, option or settings rows.
- List every **manual action**: anything that has to be done on the target environment outside the code deploy. Typical examples:
  - creating records in an admin UI (coupons, products, users, roles)
  - changing plugin, app or CMS settings
  - adding or changing environment variables and secrets (name them, never paste values)
  - running a migration, cache clear, reindex or one-off script
  - DNS, cron, webhook or third-party dashboard changes

### 2. Write the entry
Use this structure:

```markdown
## YYYY-MM-DD: <Short summary from the user's request>

### Description
<Two or three sentences: what changed and why.>

### Modified Files
- `path/from/project/root.php`: <what changed>

### Database Changes
- <change>, or "None"

### Manual Actions
- [ ] **<Action>** on <staging / production>: <why it's needed>
```

Rules for **Manual Actions**:
- Write each item as an unchecked box so it can be ticked off during the deploy.
- Bold the action, and say which environment it applies to and why the code depends on it.
- Put them in the order they must be done.
- If there are none, write `- None`. Never leave the section out, so a reader knows it was checked.

### 3. Insert it at the top
- Read the changelog file first.
- If it doesn't exist, create it with a `# Deployment Changelog` heading followed by the entry.
- If it exists, insert the new entry directly below the top-level heading, above older entries. Leave every existing entry untouched.

### 4. Confirm
Tell the user the entry was added and repeat the Manual Actions checklist in your reply, since that is the part they must act on.

## Example

**User:** "We just added the sibling discount. Update the deployment changelog."

Entry inserted at the top of `DEPLOYMENT-CHANGELOG.md`:

```markdown
## 2026-06-12: Sibling discount at checkout

### Description
Applies a 10% discount automatically when a cart contains enrolments for two or more children on the same account, using a WooCommerce coupon applied in code.

### Modified Files
- `wp-content/themes/site-child/inc/sibling-discount.php`: new file with the cart rule that applies the coupon
- `wp-content/themes/site-child/functions.php`: loads the new file

### Database Changes
- None

### Manual Actions
- [ ] **Create the `SIBLING10` coupon** (10% off, percentage discount) in WooCommerce on staging, then production: the code applies it by code name and does nothing if it's missing.
- [ ] **Clear the page cache** after deploying so the cart shows the discount line.
```

See [`examples/DEPLOYMENT-CHANGELOG.md`](examples/DEPLOYMENT-CHANGELOG.md) for a file with several entries.

## Common mistakes
- **Forgetting manual actions.** Admin and UI changes are invisible in the diff, so they're the easiest to miss and the most likely to break a deploy.
- **Overwriting the history.** Always read the file and insert; never rewrite it from scratch.
- **Pasting secrets.** Name the environment variable or credential that must be set, never its value.
