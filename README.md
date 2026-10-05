# Redacted

A play-once web game. You read real historical reactions to new
technologies with the innovation blacked out, and guess what they were
afraid of. Then you meet the author — and the year, which is usually the
punchline (one of them is Plato).

**The whole thing:**

| File | What it is |
|------|------------|
| `index.html` | The whole game — folio look, no build step, no dependencies |
| `quotes.js`  | The quote deck, encoded (see "Editing the deck") |
| `favicon.svg`, `apple-touch-icon.png`, `og-image.png` | Icon + social preview card |

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

## Editing the deck

`quotes.js` is **encoded** (XOR + base64) so answers don't show up in
view-source. This deters casual cheating only — anyone can decode it in a
console; real secrecy would need a server.

**To edit the quotes:**

1. Open the game in a browser, open the console, run
   `copy(JSON.stringify(QUOTES, null, 2))` — the decoded deck is now on
   your clipboard. Save it as `deck.json` and edit it.
2. Re-encode with Python:

   ```python
   import json, base64
   raw = json.dumps(json.load(open('deck.json')), ensure_ascii=False,
                    separators=(',',':')).encode('utf-8')
   key = b"redacted"
   print(base64.b64encode(bytes(b ^ key[i % len(key)]
         for i, b in enumerate(raw))).decode())
   ```

3. Paste the printed blob into the `atob("…")` string in `quotes.js`.

Entry fields: `id` (stable key, used for guess tracking) · `text` (the
passage; `{{!…}}` marks the innovation, `{{…}}` other redacted details;
nothing outside the reveal should hint at the era) · `answer` · `accept`
(aliases counted correct) · `author` · `year` (number, negative = BC) ·
`yearLabel` (optional display override like "c. 370 BC") · `note`
(post-reveal blurb) · `source`/`confidence` (provenance, not shown).
Array order is play order.

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
