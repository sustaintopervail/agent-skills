---
name: commit-session-changes
description: Commits the changes from the current session to git with a clear, descriptive message, staging only the files that belong to the work. Use when the user asks to commit, save progress, or checkpoint changes.
---

# Commit Session Changes

Commit often, commit only what belongs together, and write messages that a reviewer can understand without the conversation.

## Workflow

### 1. Find the right repository
- Run `git rev-parse --show-toplevel` from the folder you changed files in. In projects with nested repositories (for example a plugin or package with its own `.git`), this tells you which repo the files belong to.
- If the user named a specific folder or repo, use that.

### 2. Review what changed
```bash
git status --short
git diff --stat
```
Compare the list with the files you changed this session. Files you didn't touch may be the user's own work in progress.

### 3. Stage deliberately
- Stage the files from this session by path: `git add path/one path/two`.
- Use `git add -A` only when every change in the working tree is yours and belongs in this commit.
- Never stage secrets, `.env` files, credentials, large build output or local config. If one shows up, leave it unstaged and tell the user.
- If the changes cover unrelated pieces of work, suggest splitting them into separate commits.

### 4. Write the message
- Follow the repo's existing style (check `git log --oneline -10`), for example Conventional Commits like `feat:` and `fix:` if the history uses them.
- Subject line: under about 72 characters, imperative mood, saying what changed. Add a body for the why when it isn't obvious.
- Avoid vague messages such as "Updated files" or "WIP".

```
fix: keep schedule IDs when saving repeater rows

The row layout cleanup dropped the hidden schedule_id input, so every
save cleared the stored ID. Restores the field and adds it to the
row template test.
```

### 5. Commit and confirm
```bash
git commit -m "<subject>" -m "<body>"
git log --oneline -1
```
Tell the user the commit hash, the message and the files included. Don't push unless asked.

## Common mistakes
- Committing from the wrong repository in a project with nested repos.
- Sweeping in unrelated or secret files with `git add .`.
- Messages that describe the files instead of the change.
