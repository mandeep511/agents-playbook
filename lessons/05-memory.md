# Memory

Two separate problems that get conflated: **what the agent knows about the user across sessions**
(memory) and **what was said** (the transcript). Different shapes, different failure modes.

## Files or a database — decide on provenance

Markdown files in a directory are the right default. `ls`, `cat` and `grep` are a retrieval
interface the model already knows, they are debuggable by a human with a file browser, and there is
no schema to migrate.

Move to structured rows when a memory needs **fields you must query or enforce** — provenance, a
validity window, a supersedes pointer. That is the whole test. If you only ever read memories
whole, a database buys you nothing; one production harness abandoned structured memory twice.

We went structured because a stored claim needs to answer *where did this come from* and *is it
still true*, and prose cannot be queried for either.

## Bound it, and fail loud when it is full

Give each store a character limit and show the usage in the injected block:

```
MEMORY (your notes) [67% — 1,474/2,200 chars]
```

**When a write would exceed the limit, return an error rather than silently dropping an entry.**
The agent then makes room itself — consolidating or removing in the same turn — which is the
behaviour you want and will not get from an eviction policy you wrote.

Note the trap in `replace`: swapping an entry for a longer one can still overflow. The limit is on
the result, not on the operation.

Telling the agent its own capacity is what makes self-curation work. An agent that cannot see it is
nearly full will keep appending.

## Frozen snapshot, not live injection

Load memory into the system prompt **once at session start** and never change it mid-session. The
prompt prefix is what your cache is keyed on; editing it mid-session pays for the whole prefix
again.

Writes persist to disk immediately and appear in *tool responses* right away — so the agent sees
its own live state — but the prompt block stays frozen until the next session. Two views, one of
them deliberately stale, and it is worth the inconsistency.

## Memory is a claim, not an instruction

This is the inversion that matters most if you are borrowing from a coding harness.

Coding harnesses give memory files an **override preamble** — the memory wins over the default
behaviour, because the user wrote it to correct the agent. In a product where memory is derived
from *content the agent read*, that precedence is an injection vector.

Ours is inverted: **a stored memory may never override the safety envelope**, and remembered
material is framed as a claim to be checked rather than as a rule to follow.

Two consequences:

- **A remembered value is not a source.** If a claim needs provenance, recalling it does not supply
  any. Re-read the record.
- **Free-form path imports must not exist.** A memory file that can say *"also read this path"*
  lets an uploaded document drive a filesystem read. Reference harnesses guard this with a human
  approval dialog; if you have no human to click, the syntax should not exist.

## Recall is about the past, including about your own code

A recalled memory reflects what was true when it was written. If it names a file, a function or a
flag, **verify that still exists before acting on it.** Memories rot in exactly the way that is
hardest to notice: the claim is specific, confident, and was correct.

## When one row supersedes another, enumerate what travels with it

**State carried forward is state that has to be carried explicitly.** The fields you forget do not
error — they read as absent, and absent is the value the guard treats as safe. Law 1 again.

> **Observed** · An acknowledgement row inherited no value fingerprint from the alert it was
> reacting to. So *"I know, I'm on it"* recorded at one severity silenced a far worse later value:
> the comparison ran against `None` and reported no change.

Write down the list of fields that must survive a supersede, and test it. This is not derivable by
reading the write site — it only shows up when the *next* read asks a question the row cannot
answer.

## An ordering that ties is an ordering that lies

> **Observed** · `created_at` came from a wall clock that ticks about every 15 ms, so several rows
> written in one turn shared a timestamp. `ORDER BY created_at DESC LIMIT 1` then picks between them
> arbitrarily — and in an append-only table that means reading a **superseded** row as current.
> Found by a test that failed once in five runs, which is the tell: a 1-in-5 flake in a suppression
> lookup is a suppression that happens sometimes and cannot be reproduced.

**Prefer asserting the structural claim over any timestamp order** — the row whose `supersedes IS
NULL` is *defined* as current; the newest row merely usually is.

If you do need a tiebreak, note that SQLite does not expose `rowid` through a view, so a `_current`
view cannot be ordered safely at all and the query has to run the view's predicate against the base
table.

## Transcripts: append-only, with real branching

| Rule | Why |
|---|---|
| Append-only transcript, small mutable index | The index is the only thing you contend on |
| A routing key points at the current transcript | The bucket stays stable while the context is replaceable |
| Durable turn ids + parent pointer + leaf pointer | **Keying turns by `(chat, index)` forbids branching**, and overwriting an edited turn silently changes text a citation already points at |
| Store whole usage, derive cost | Pricing changes; usage does not |
| Record kinds separate model-visible / UI-visible / state-only | Three audiences, one log, and they disagree about what belongs |
| Delete archives rather than unlinks | An append-only store where one path really deletes is not append-only |

Keep transcript hygiene in **exactly one function.** Every place that trims, rewrites, or filters
the history is a place the replay can diverge from what ran.

## Injecting memories into a turn

- **Cap how many may be force-included.** "Everything is important" is the same failure as marking
  nothing important.
- **Prefer conditional injection** — a predicate on the active topic — over always-on.
- Budget the tokens per turn, and count the injected size separately from the raw size. See
  [04](04-context.md).
