---
name: refactor-deletion-check
description: Reviews removed lines before committing a refactor, UI cleanup or restructure, to catch silently dropped hidden fields, handlers, hooks, routes and file loaders. Use before committing any "cleanup", "refactor" or layout change, or when a diff removes many lines.
---

# Refactor Deletion Check

When an agent restructures markup or reorganizes code, it sometimes drops things that look unimportant: a hidden input, a button, an event handler, an include. Nothing errors, and the feature breaks quietly, sometimes for weeks.

The riskiest losses:
- **Hidden form fields.** If a hidden input disappears from a form or repeater row, its saved value can be wiped on the next save.
- **Event and AJAX handlers, hooks, routes.** The UI still renders, but the action does nothing.
- **Security tokens.** Removing a CSRF token or nonce breaks the request, or worse, removes a check.
- **Guarded includes.** Code like `if (file_exists($path)) require $path;` skips a missing file with no warning.

Run this check before committing any change that removes or moves code you weren't explicitly asked to remove.

## Check 1: List everything removed

```bash
git diff HEAD -- <file> | grep '^-' | grep -v '^---'
```

For every removed line, either find where it moved to (it appears in a `+` line) or confirm the user asked for it to go.

## Check 2: High-risk patterns

Search the removed lines for things that fail silently:

```bash
git diff HEAD | grep '^-' | grep -v '^---' | grep -iE \
  'type="hidden"|<button|onclick|addEventListener|wp_ajax_|add_action|add_filter|nonce|csrf|route|require|include|import |file_exists'
```

Then confirm each match also appears in an added line:

```bash
git diff HEAD | grep '^+' | grep -v '^+++' | grep -iE '<same pattern>'
```

**Red flag:** a pattern that appears only in removed lines. Restore it, or ask the user whether it was meant to go. If in doubt, keep it.

## Check 3: Guarded includes still resolve

If the project loads files behind an existence check, make sure every guarded path still exists:

```bash
grep -rn "file_exists\|os.path.exists\|fs.existsSync" <loader files>
```

If a file was deleted, either restore it or remove the guard and the feature deliberately.

## Check 4: Smoke-test the changed screen

For form and repeater changes:
1. Open the screen and confirm every field and button still renders.
2. Save, reload, and confirm hidden values (IDs, external sync IDs) are unchanged.
3. Click each action button once.

If you can't run the app, give the user these steps instead.

## Report

Before committing, tell the user what you checked, in one short list:

> Removed-line check: 41 lines removed. All moved except the old `<table>` wrapper, which you asked to replace. Hidden inputs `schedule_id` and `external_item_id` are still present. The "Sync" button and its `wp_ajax_sync_schedule` handler are still present.

## Example

A "clean up the row layout" change that passes visual review but shows this in Check 2:

```diff
- <input type="hidden" name="rows[3][external_item_id]" value="8812">
- <button class="sync-row">Sync</button>
```

with no matching `+` lines. Both must be restored before committing: without the hidden input, the next save erases the stored ID, and without the button the sync feature is gone.
