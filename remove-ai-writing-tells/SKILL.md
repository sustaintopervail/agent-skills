---
name: remove-ai-writing-tells
description: Rewrites a draft so it stops reading as AI-generated, without changing its facts. Use before publishing a post, article, email or README, or when the user says "make it sound human", "remove AI markers", "de-AI this" or "sounds like ChatGPT".
---

# Remove AI Writing Tells

Readers spot AI drafts from a handful of habits: arrows for bullets, dramatic one-word fragments, "it's not X, it's Y" lines, tidy closing aphorisms. Once spotted, they stop trusting the rest, even the true parts. This skill does a rewrite pass that removes those habits and keeps everything the draft says.

## Ground rules

- **Keep every fact.** Numbers, times, names, quotes, code and links stay exactly as they were. This is a style pass, not an edit of content.
- **Never invent.** Don't add anecdotes, feelings, quotes or details to make it sound more human. If a sentence needs a detail you don't have, simplify the sentence instead.
- **Real quotes stay verbatim.** If the author quoted someone (or themselves), don't polish the quote. If a quote has to go, retell it without quotation marks.
- **Match the author's voice.** If there are earlier posts or messages from the author, copy their sentence length, spelling (British or American) and level of formality.
- **Leave code blocks, commands and log output alone.**

## Step 1: Scan for the tells

Read the whole draft and mark each of these:

| Tell | Example | Fix |
|---|---|---|
| Arrow or emoji bullets | `→ It edited the wrong file.` | Plain `-` bullets, or prose |
| Bold label at the start of every bullet | `- **Theory before evidence.** It guessed.` | Drop the label or fold it into the sentence |
| Dramatic fragments | `Carefully. With logging.` / `0 rows deleted.` on its own line in bold | Join into a full sentence |
| "Not X. Y." / "It's not X, it's Y" | `They were students. Not products.` | `They were students, not products.` |
| Mirrored pairs | `Ten minutes with evidence. Hours without it.` | Say it once, plainly |
| Aphorism endings | `It didn't need to be smarter. It needed a protocol.` | Cut, or replace with what actually happened |
| "The point is / The worst part / Here's the thing:" | `The worst part: the rule existed.` | `What annoyed me most was that the rule existed.` |
| Groups of three for rhythm | `fast, focused and fearless` | Keep only the items that carry information |
| Em-dashes everywhere | `the cache — of course — was Redis` | Commas, brackets or two sentences |
| Inflated words | `delve, crucial, seamless, robust, game-changer, testament, landscape, journey` | The ordinary word |
| Hedge-then-hype openers | `In today's fast-paced world...` | Start with the first real fact |
| Empty feelings | `humbling for both of us`, `I'm thrilled to share` | Cut, or say what happened |
| Engagement bait | `Agree? 👇`, `Thoughts?` | One specific question, or none |
| Headings that sound like slides | `The takeaway`, `Why this matters` | A plain heading, or no heading |
| Perfectly parallel lists | five bullets that all start `**Not ...**` | Vary them, or turn them into a sentence |

Not every instance is wrong. One short sentence for emphasis is fine; five in a row is a tell. Fix the pattern, not every occurrence.

## Step 2: Rewrite

Work paragraph by paragraph:

1. Join fragments into full sentences.
2. Replace symbols (`→`, `—`, emoji bullets) with plain punctuation.
3. Cut closing lines that restate the paragraph as a slogan.
4. Turn bold-labelled lists into plain bullets, a numbered list, or a sentence when the items are short.
5. Swap inflated words for ordinary ones.
6. Let sentence length vary. Real writing has some long sentences with a clause or two in them, and some short ones.
7. End the piece on something concrete (what the author does now, a link, one specific question) rather than a moral.

## Step 3: Check

Before handing the draft back:

- [ ] Search for `→`, `—`, `**` at the start of bullets, `Not ` at the start of sentences, and the inflated words above.
- [ ] Every number, time, name, link and quote matches the original.
- [ ] Nothing new was invented.
- [ ] Read the first and last paragraph aloud. If either sounds like a keynote, rewrite it.

Then give the user a short list of the kinds of change you made (for example "removed arrow bullets, joined fragments, cut the slogan ending"), not a line-by-line diff.

## Example

Before:

> 0 rows deleted.
>
> Ten minutes once there was evidence. Hours while there wasn't.
>
> → It edited a PHP class that was never loaded. Carefully. With logging.
>
> The agent didn't need to be smarter. It needed a protocol.

After:

> I ran it and got 0 rows deleted.
>
> So about ten minutes of real debugging, after a few hours of guessing.
>
> - It edited a PHP file that was never loaded, added logging to it, and then we both waited for log lines that were never going to appear.
>
> Afterwards I wrote the post-mortem up as five rules the agent has to follow.

## Common mistakes
- Polishing a real quote until it no longer matches what was said.
- Adding a made-up personal detail to make the text "warmer".
- Removing every short sentence, which reads just as artificial.
- Swapping the arrows for em-dashes.
- Rewriting code blocks or log output.
