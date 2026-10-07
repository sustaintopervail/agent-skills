---
name: evidence-first-debugging
description: Debugging protocol that requires proof before naming a root cause. Use for any bug, missing or wrong data, or silent failure, and whenever the user says "not showing", "not saving", "not working", "investigate" or "debug".
---

# Evidence-First Debugging

AI agents are quick to announce a plausible cause and start fixing it. When the theory is wrong, hours go into fixing the wrong thing. This protocol makes the agent prove what is happening before it changes code.

Apply it from the start of any investigation, without waiting to be asked.

## Rule 0: No theory without evidence

Don't state a root cause until you have a log line, a query result, a stack trace or a search match that shows it. If you catch yourself writing "the issue is probably X", stop and gather evidence first.

When you report a cause, quote the evidence next to it.

## Rule 1: Instrument the code path first

When data is missing, wrong or not appearing:

1. Add logging **inside the function that should produce the data**, at its entry and at each branch that matters.
2. Trigger the action (page load, form submit, API call, job run).
3. Read the log.
4. Only now decide what's wrong and write a fix.

Log to a file or the server log, not to the page. Output printed into HTML can be swallowed by page caches, redirects or output buffering, so its absence proves nothing. Answer "does this function even run?" before asking what it returns.

Remove or disable the temporary logging once the bug is fixed.

## Rule 2: Know which layer you're looking at

Before changing anything, note what sits between the code and what you're seeing. Caches stack, and each needs a different way to clear:

| Layer | Typical examples | How to bypass or clear |
|---|---|---|
| Browser | HTTP cache, service worker | Hard reload, private window |
| CDN / page cache | Cloudflare, Varnish, WordPress page-cache plugins | Purge, or a cache-busting query string |
| Object cache | Redis, Memcached | The framework's cache-delete API, or flush the key |
| Database | Stored options, sessions, rows | Direct query, which only helps if no layer above caches it |

Work down the stack. If deleting a row in the database changes nothing, or deletes zero rows, you're probably in the wrong layer. Read screenshots and admin toolbars for clues (for example an "Object Cache" or page-cache indicator) before suggesting a fix.

## Rule 3: Confirm a file is actually loaded before editing it

Projects often contain dead copies, backups or same-named files in different folders.

```bash
# Is the file included/imported anywhere?
grep -rn "filename" --include="*.php" --include="*.js" --include="*.ts" --include="*.py" .

# Are there several files with this name?
find . -name "filename.ext" -not -path "*/node_modules/*" -not -path "*/vendor/*"
```

Edit only the file that the include, import or autoloader actually resolves to. If more than one candidate exists and you can't tell which is live, show the user both paths and ask before editing either.

## Rule 4: Fix every code path, not just one

When a fix adds a cache clear, a log call, a sync trigger or a validation, search for every place the same operation happens:

```bash
grep -rn "save_order\|update_settings" .
```

Confirm each one has the fix, and list them in your summary. A fix on one path that leaves another broken is a bug waiting to come back.

## Example

**User:** "The class schedule isn't showing on the site after I save it."

Bad start: "This is probably because the post type is wrong. Let me change it."

Good start:

> I'll check whether the schedule function runs at all before guessing. I've added logging at the start of `render_schedule()` and after the query. Please reload the page once.
>
> The log shows the function runs and the query returns 6 rows, so the data is fine. The page still shows the old version, and your toolbar shows a page cache is active. Purging that cache should show the new schedule; if it doesn't, I'll look at the object cache next.

## Common mistakes
- Explaining a cause before any log or query backs it.
- Debugging with on-page output when a page cache is active.
- Running database deletes when the value lives in an object cache.
- Editing a dead duplicate file.
- Fixing the one code path you looked at and missing the others.
