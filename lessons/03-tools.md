# Tools

Tool-set bloat is the most common agent failure mode. **If a human engineer cannot say which tool
applies, the model cannot either.**

Test your catalogue by handing the descriptions to an engineer who did not write them and asking
which tool they would use for five real requests. Overlapping tools do not degrade gracefully —
the model picks one, and which one is effectively random.

## Never hand the model two parallel lists

This is the most expensive tool-design defect we shipped, and it is entirely structural.

A tool returned rendered lines *and*, on one trailing line, every source handle in its own order.
Nothing kept them in step. The model has to zip them, and any answer that regroups the rows into
its own order breaks the zip **silently**.

> **Observed** · One answer regrouped eleven results into categories and attached handles in the
> tool's order: **eleven resolvable citations on the wrong rows.** Twelve of the nineteen real
> mis-citations in the archive are this one shape.

It is more dangerous than a wrong value. A misstated number can be caught by a user who knows
their own data. **A correctly-stated value citing the wrong source cannot be caught by anyone** —
the link resolves, the document opens, and it is the wrong document.

Three rules out of it:

- **Attach the reference to the row it belongs to**, and do it at the one chokepoint every tool
  reaches the model through. The fix existed and had been wired to a single surface while every
  other producer still emitted the flat list.
- **A gate against this must be structural, not ordinal.** Match on the row's leading name token
  and every number in its label — that is provably permutation-invariant. Then permute the inputs
  in the test and assert one pairing. Under the old mechanism, 24 permutations gave 24 different
  pairings, which is the whole defect in one number.
- **A producer may not state a value it cannot source.** One producer joined several values onto
  one line under a single summary handle, so they were uncitable by construction — and the model
  reached for whatever handle was nearby.

## Two channels, always

One channel for the model, one for the harness, UI and logs. Plus partial-progress updates for
anything slow.

**Read failure from the envelope, never by sniffing the text for the word "error".** A document
containing the word `error` is not a failed call, and a failed call may not contain it.

## Errors are prompts

An error message is the only instruction the model gets at the moment it is most likely to act on
one. Spend tokens here.

- Machine-readable prefix, then prose.
- **Name the next call.** A truncation notice that does not say how to get the rest teaches the
  model that the data does not exist.
- Enforce prerequisites twice — as prose in the description, and as a validator that fires.
- Two-tier descriptions: a short line for everything, plus a long prompt **only** for the handful
  of tools where misuse is expensive. Long descriptions everywhere is bloat again.

## A refusal needs a destination

A gate that correctly refuses something and has nowhere to put it silently drops it — which is
unreviewable, and the exact opposite of the gate's purpose.

> **Observed** · A validation gate refused extracted facts with no staging table behind it. Same
> shape as queueing a job kind no worker drains.

**Ask of every gate: where does the refused thing go?**

## A silent cap is a lie

Truncating without setting the flag tells the model it has the whole picture.

- **Order matters more than the limit.** A cap that keeps *"substrate of CYP3A4"* and drops the
  interacting drug names inverts the clinical value of the result. Truncate the least useful end.
- **Never bisect a structured token.** Half an identifier looks real, gets cited, and resolves to
  nothing.
- **Spill to a file** and hand back a path when the result is genuinely large.
- **A filter is a cap too.** A library view matched its "in your record" filter on one status, so
  a whole class of documents appeared under *All* and under **no other filter** — the list was not
  wrong, it was quietly short. **Assert that no value falls into zero buckets.**

## Flags the registry should carry

| Flag | Why the harness needs it |
|---|---|
| `read_only` | Lets you gate execution separately from reading, and parallelise safely. A harness without this can only gate execution. |
| `concurrency_safe` | A mutating call blocks the queue; a read does not. |
| tool group | Permissions resolve over groups, deny-wins. Per-tool ACLs do not scale. |

## Capability checks need three states

A model-capability table goes stale. So a check against it has three answers, not two:

```
True   → call it
False  → refuse before spending the call, and name the slug, its real
          modalities, and the env var that overrides
None   → attempt it
```

**`None` is attempted, not refused.** Refusing on unknown means a stale table silently breaks
every model it has not heard of — the guard becomes the outage. This is law 1 with the safe
default on the *permissive* side, which is the right call whenever the failure is a recoverable
error rather than a harm.

## Growing the catalogue without breaking the cache

Prefer a mechanism that **appends** schemas on demand over one that swaps the tool array, because
a swap invalidates the prompt prefix and you pay for the whole cache again.

And keep procedures on disk rather than in the prompt: an index entry of ~45 tokens each, bodies
read on demand, one per turn, with a mandatory selection protocol. That also makes them
reviewable by a domain expert who should never have to touch the harness.
