---
name: time-logger
description: Logs time spent on features or tasks to a CSV ledger for billing. Use when the user asks to log time or track hours, or after finishing significant feature work they want to bill for.
---

# Time Logger

Append one row per piece of work to a CSV time ledger, so billable hours are recorded while the details are still fresh.

## Configuration

**Ledger path:** `time-log.csv` in the project root, unless the project says otherwise.

To use a different location, add a line like this to the project's agent instructions (`CLAUDE.md`, `AGENTS.md` or equivalent):

```
Time log: docs/billing/time-log.csv
```

Relative paths are resolved from the project root. If the user names a path in the conversation, that wins.

## Workflow

### 1. Find or create the ledger
- Read the ledger file to check whether it exists.
- If it doesn't, create it with exactly this header row:
  `Date,Feature/Task,Time Spent (Hours),Description`

### 2. Determine time spent
- If the user gives the time ("log 2 hours for the checkout fix"), use it as given.
- If they don't, **estimate** from the work done in this session and ask the user to confirm the estimate before writing anything. Never log an unconfirmed guess.

### 3. Format the row
| Column | Format |
|---|---|
| `Date` | Today's local date, `YYYY-MM-DD` |
| `Feature/Task` | Short, high-level name, e.g. `Stripe Webhook Handler` |
| `Time Spent (Hours)` | Decimal, e.g. `1.5`, `0.25` |
| `Description` | One or two sentences on what was actually done |

Always wrap `Description` in double quotes, and double any quote characters inside it (`"` becomes `""`). Wrap `Feature/Task` in quotes too if it contains a comma.

### 4. Append the row
- Add the row to the end of the file. Never rewrite or reorder existing rows.
- If you append from a shell, use a method that preserves the quoting above, and make sure the file ends with a newline before you append.

### 5. Confirm
Show the user the row you recorded as a markdown table, plus the ledger path.

## Example

**User:** "Log 1.5 hours for the subscription bundle work."

Row appended to `time-log.csv`:

```csv
2026-06-09,Subscription Bundle Builder,1.5,"Built recurring pricing hooks for bundled products and documented the approach."
```

Reply to the user:

> Logged to `time-log.csv`:
>
> | Date | Feature/Task | Hours | Description |
> |---|---|---|---|
> | 2026-06-09 | Subscription Bundle Builder | 1.5 | Built recurring pricing hooks for bundled products and documented the approach. |

See [`examples/time-log.csv`](examples/time-log.csv) for a ledger with several entries.

## Common mistakes
- **Logging a guess.** Estimated hours must be confirmed by the user first.
- **Overwriting the ledger.** Always append; read the file first if unsure.
- **Broken CSV.** An unquoted comma in the description shifts every column after it.
