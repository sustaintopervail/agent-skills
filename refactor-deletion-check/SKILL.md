---
name: refactor-deletion-check
description: Reviews removed lines before committing a refactor, UI cleanup or restructure, and verifies that hidden fields, handlers, hooks, routes and file loaders still work. Use before committing any "cleanup", "refactor" or layout change, or when a diff removes many lines.
---

# Refactor Deletion Check

When an agent restructures markup or reorganizes code, it sometimes drops things that look unimportant: a hidden input, a button, an event handler, an include. Nothing errors, and the feature breaks quietly, sometimes for weeks.

The riskiest losses:
- **Hidden form fields.** If a hidden input disappears from a form or repeater row, its saved value can be wiped on the next save.
- **Event and AJAX handlers, hooks, routes.** The UI still renders, but the action does nothing.
- **Security tokens.** Removing a CSRF token or nonce breaks the request, or worse, removes a check.
- **Guarded includes.** Code like `if (file_exists($path)) require $path;` skips a missing file with no warning.

Run this check before committing any change that removes or moves code you weren't explicitly asked to remove.

Moving or rewriting code is fine; refactors are supposed to change implementation. What this check guards is **behavior**: every removed behavior must either be deliberately dropped or be shown to still work somewhere else. Matching text is a clue, not proof. A hidden field can keep its name but get the wrong value, and a button can survive while losing its handler.

## Check 1: List everything removed

```bash
git diff HEAD -- <file> | grep '^-' | grep -v '^---'
```

Group the removed lines into the behaviors they implement (a field, a button and its handler, a hook registration, an include). Formatting-only and wrapper changes can be noted once and skipped.

## Check 2: High-risk patterns

Search the removed lines for things that fail silently:

```bash
git diff HEAD | grep '^-' | grep -v '^---' | grep -iE \
  'type="hidden"|<button|onclick|addEventListener|wp_ajax_|add_action|add_filter|nonce|csrf|route|require|include|import |file_exists'
```

Then look for the same patterns in added lines, to find where each one went:

```bash
git diff HEAD | grep '^+' | grep -v '^+++' | grep -iE '<same pattern>'
```

**Red flag:** a pattern that appears only in removed lines. Restore it, or ask the user whether it was meant to go.

## Check 3: Map each behavior to its replacement

For every high-risk behavior from Check 2, fill in one row:

| Removed behavior | Now lives in | Verified by |
|---|---|---|
| Hidden `external_item_id` input | `row-template.php` line 42 | Saved and reloaded; value still `8812` |
| `wp_ajax_sync_schedule` handler | Moved to `class-sync.php` | Clicked Sync; request returned 200 and the log shows the handler ran |
| Old `<table>` wrapper | Removed (user asked) | n/a |

"Verified by" should be an observation: a test that passes, a value preserved after save and reload, a request that succeeds, a log line. If a behavior has no replacement and nobody asked for it to go, restore it. If you can't verify a row yourself, mark it **unverified** and give the user the step to check it.

## Check 4: Guarded includes still resolve

If the project loads files behind an existence check, make sure every guarded path still exists:

```bash
grep -rn "file_exists\|os.path.exists\|fs.existsSync" <loader files>
```

If a file was deleted, either restore it or remove the guard and the feature deliberately.

## Check 5: Smoke-test the changed screen

For form and repeater changes:
1. Open the screen and confirm every field and button still renders.
2. Save, reload, and confirm hidden values (IDs, external sync IDs) are unchanged.
3. Click each action button once.

If you can't run the app, give the user these steps instead.

## Report

Before committing, give the user the mapping table from Check 3 and a one-line summary:

> Removed-line check: 41 lines removed across 4 behaviors. 3 verified in their new locations, 1 removed as you asked, none unverified.

## Example

A "clean up the row layout" change that passes visual review but shows this in Check 2:

```diff
- <input type="hidden" name="rows[3][external_item_id]" value="8812">
- <button class="sync-row">Sync</button>
```

with no matching `+` lines and no row in the mapping table. Both must be restored before committing: without the hidden input, the next save erases the stored ID, and without the button the sync feature is gone.
