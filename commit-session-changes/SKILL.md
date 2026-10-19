---
name: commit-session-changes
description: Commits the changes from the current session to git with a clear, descriptive message, staging only the changes (down to individual hunks) that belong to the work. Use when the user asks to commit, save progress, or checkpoint changes.
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
git diff --cached --stat   # already staged, possibly by the user
git diff --stat            # unstaged
```
- Compare the list with the files you changed this session. Files you didn't touch may be the user's own work in progress.
- If something is **already staged** that you didn't stage, ask before including it or unstaging it.
- For each file you changed, read its diff. If it also contains edits you didn't make (the user's, or another tool's), the file has **mixed ownership** and must be staged by hunk, not as a whole file.

### 3. Stage deliberately
- Stage files that contain only your changes by path: `git add path/one path/two`.
- For mixed-ownership files, stage only your hunks. Interactive `git add -p` needs a terminal, so agents usually build a patch of just their hunks and apply it to the index: `git apply --cached my-changes.patch`. If a hunk mixes your lines with someone else's, stop and ask the user rather than guessing.
- Use `git add -A` only when every change in the working tree is yours and belongs in this commit.
- Never stage secrets, `.env` files, credentials, large build output or local config. If one shows up, leave it unstaged and tell the user.
- If the changes cover unrelated pieces of work, suggest splitting them into separate commits.

### 4. Check the staged diff
Immediately before committing, read the whole staged diff:
```bash
git diff --cached
```
Every line in it should be yours and belong to this piece of work. If anything else is there, unstage it (`git restore --staged <path>`) or ask.

### 5. Write the message
- Follow the repo's existing style (check `git log --oneline -10`), for example Conventional Commits like `feat:` and `fix:` if the history uses them.
- Subject line: under about 72 characters, imperative mood, saying what changed. Add a body for the why when it isn't obvious.
- Avoid vague messages such as "Updated files" or "WIP".

```
fix: keep schedule IDs when saving repeater rows

The row layout cleanup dropped the hidden schedule_id input, so every
save cleared the stored ID. Restores the field and adds it to the
row template test.
```

### 6. Commit and confirm
```bash
git commit -m "<subject>" -m "<body>"
git log --oneline -1
```
Tell the user the commit hash, the message and the files included. Don't push unless asked.

## Common mistakes
- Committing from the wrong repository in a project with nested repos.
- Sweeping in unrelated or secret files with `git add .`.
- Staging a whole file that also holds the user's unrelated edits.
- Committing on top of changes someone else had already staged.
- Messages that describe the files instead of the change.
