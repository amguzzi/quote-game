# Redacted

A play-once web game. You read real historical reactions to new
technologies with the innovation blacked out, and guess what they were
afraid of. Then you meet the author — and the year, which is usually the
punchline (one of them is Plato).

**Everything is two files:**

| File | What it is |
|------|------------|
| `index.html` | The whole game — folio look, no build step, no dependencies |
| `quotes.js`  | The quote deck. Edit this to change the game; no code changes needed |

Serve the two files from anywhere static (Vercel, Netlify, GitHub Pages)
and it works. Opening `index.html` locally also works, though guess
tracking may be blocked from `file://`.

## How it plays

Title → passages one at a time. Type a guess, press **Reveal**: the
redaction bars dissolve in place, a ✓/✗ lands in the guess box, and the
answer appears with author, year, and a one-line note. Enter reveals;
Enter again advances. The final Reckoning leads with your *finest
misjudgement* — if you ever guessed "AI," the oldest thing you said that
about; otherwise the oldest thing you got wrong — then your score and a
copy-to-share summary.

Answer matching is loose: case, punctuation and articles are ignored, and
each quote lists `accept` aliases.

## The quote deck (`quotes.js`)

Each entry:

```js
{
  id: "writing-plato",          // stable key, used for guess tracking
  text: "… their trust in {{!writing}}, produced by external characters …",
  answer: "writing",            // shown on reveal
  accept: ["the alphabet"],     // extra answers counted correct
  author: "Plato",              // shown only on reveal
  year: -370,                   // number; negative = BC (sorts the "oldest miss")
  yearLabel: "c. 370 BC",       // optional display override
  note: "…",                    // one-line blurb shown after the reveal
  source: "…", confidence: "…"  // provenance, not shown in game
}
```

`{{!…}}` marks the innovation (underlined on reveal); `{{…}}` marks other
identifying details to redact. Nothing outside the reveal should hint at
the era. Array order is play order. All five current quotes are verified
against the sourced deck; an unverified entry was excluded.

## Guess tracking

Every reveal fires an insert into the `redacted_guesses` table of the
owner's Supabase project (`session_id` per playthrough, `quote_id`,
`guess`, `correct`). The embedded key is the publishable anon key and the
table is insert-only under RLS — players can write a guess, never read
one. Tracking is fire-and-forget and can never break the game; blank out
`TRACK.url`/`TRACK.key` in `index.html` to disable it.

```sql
-- read results in Supabase:
select quote_id, guess, correct, count(*)
from redacted_guesses group by 1,2,3 order by 1, count(*) desc;
```

---

The design exploration that led here (four visual directions, a reveal-
pattern lab, two flow studies) lives in this repo's git history, before
the "Just the game" commit.
