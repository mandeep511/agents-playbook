# Context

Context is a finite attention budget, not a container to fill. Recall degrades as tokens grow, in
every model — so the goal is the **smallest set of high-signal tokens** that gets the outcome, not
the largest set that fits.

## Measure it from the provider, not from a tokenizer

You do not need a tokenizer. Take the authoritative `usage` from the last response, then estimate
only what you have appended since (`chars / 4 × 1.2` is close enough).

Paying for a token-count round trip per turn is paying for a number the previous response already
gave you exactly.

**And count every token class the provider bills** — input, output, cache writes, cache reads. A
cost function that misses a class under-counts exactly the expensive turns.

## Three levers, and they are not interchangeable

| Lever | Sweet spot | Cost |
|---|---|---|
| **Compaction** | Conversational back-and-forth, where continuity matters | 1 LLM call, and everything not in the summary is gone |
| **Structured notes** | Iterative work with milestones — write state to a file, read it back | Discipline; the file goes stale silently |
| **Sub-agents** | Parallel exploration. Burns tens of thousands of tokens, returns 1–2k | Orchestration, and the parent cannot see the reasoning |

Choosing wrong is a common and expensive mistake: compaction on milestone work loses the
milestones, and notes on a conversation read as a stranger's summary of your own chat.

**A fourth option: a context reset.** A fresh agent plus a structured handoff beats compaction
when the model starts showing *context anxiety* — wrapping up prematurely near what it believes is
its limit. A reset costs orchestration, tokens and latency, and buys a clean slate that compaction
cannot give you.

## Two layers, and pruning is not compaction

Run them as separate mechanisms in a fixed order. Pruning is cheap, in-memory and non-persistent;
compaction is expensive, summarising and permanent.

### Layer 1 — pruning (0 LLM calls, does not touch the transcript on disk)

Trim old **tool results** only. Never rewrite conversation text.

1. Gate on both a **time** check and a **size** check. Do not prune while the prompt cache is
   still warm — you would spend cache-write cost to save context you were not paying for.
2. Above a soft ratio (~0.3 of the window), **soft-trim** oversized results to head and tail with
   a marker in between — **and state the original size in the marker.** A trim that hides its own
   extent is a silent cap.
3. Above a hard ratio (~0.5) *and* only if a worthwhile amount of prunable content remains,
   **hard-clear** to a placeholder.
4. Reset the clock only when pruning actually changed something.

Two safety rules regardless of thresholds: never prune the most recent few assistant turns, and
**never prune before the first user message** — that is where bootstrap reads live.

**Give eviction a floor.** Without one you pay a full pass for a trivial saving. And check what
you are *not* clearing: reasoning blocks were the largest thing in our context and were never
touched at all.

> **The dead end to avoid** · Evicting a tool result while keeping your record that it was read is
> unrecoverable — the agent can neither see the data nor know to re-fetch it. **Eviction must drop
> both, and a cleared read must always be re-servable.**

### Layer 2 — compaction (1 LLM call, persistent)

**Trigger on an absolute reserve below the window, not a percentage.** Percentages move when you
change models. Put a warning band well below the trigger, and only evict inside the band.

**Cut point:** walk back a fixed token distance, then snap *forward* to a valid boundary. A tool
result is never a valid cut point — it orphans the call. Snapping to a user message is the
simplest rule that always works.

## The summary needs a format, not a budget

A rolling summary specified as *"about N tokens"* with no format, no preservation rules and no
failure behaviour will drop the one thing that mattered, silently.

> A dropped *"statin intolerance, myalgia, late 2025"* is a safety event, not a quality
> regression.

Use a fixed section template: **goal · constraints · progress · decisions · next steps · critical
context.** Keep a separate *update* variant with explicit preserve rules for re-summarising an
existing summary.

**The section names do the work, not the prose.** The tell: one production harness ships a
numbering bug in its template and it does not matter.

Two rules:

- **Never introduce a new fact.** A summary is lossy compression, not inference.
- **Have a degradation ladder** — full, then partial, then a bare note saying compaction failed.
  A failed summarisation must not look like a successful empty one.

**Carry a read ledger and a failure ledger** — capped in count and length — into every summary
generation. Roughly 200 tokens, instead of re-injecting 25k of documents to remind the model what
it already looked at.

## Make the rebuild reproducible

Persist a **boundary record** at every compaction: sever the parent link, record the logical
parent, and record what was preserved.

Without it, the live context and the context you rebuild on resume are different objects, and
*"what did the model see at turn 212?"* has no answer. That is the question you will need on your
worst day.

On rebuild: the summary re-enters as a flagged message, and the most recent files are re-injected
as synthetic read pairs with **per-item and total caps** — oversize ones become a pointer, not a
truncation.

## Injection discipline

Everything the harness adds between turns is a liability with a token cost.

- **One wrapper kind**, on synthetic meta messages. Name every payload kind.
- **A cadence rule so nudges do not repeat.** A reminder that fires every turn is noise by turn
  three, and the model learns to skip the wrapper — including the times it mattered.
- **Every producer gets a timeout (~1s), swallows errors to empty, and there is a global kill
  switch.** A context assembler with budgets but no timeout and no failure semantics will hang a
  turn on a slow disk.
- **Persist injections with the transcript** rather than stripping them on save. Otherwise the
  replay is not what ran.
- **Cap how much may be force-included.** "Everything is important" is the same failure as marking
  nothing.
- **Conditional injection beats always-on.** Inject a rules file only when a matching predicate
  fires — a touched path, an active topic — with a token budget per turn.

And the framing rules from [02](02-system-prompts.md) apply to all of it: injected text never
shares a block with the user's own words, and worker output is a system notification rather than
user input.

## Build the introspection before you need it

A context report, generated from the last real run:

- per-source raw size **vs** injected size (the gap is where your budget went)
- tool-schema JSON size, and truncation flags
- what the assembler dropped, and why

You cannot tune what you cannot see, and every context bug looks like a model bug until you can
read the prompt the model actually got.
