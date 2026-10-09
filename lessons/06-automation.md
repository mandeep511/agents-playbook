# Automation & notifications

Work that starts without a user, and output that arrives without being asked for. This is where a
chat assistant becomes a product, and where the failure modes stop being cosmetic.

## Two shapes of scheduled work — pick per job

| Shape | Mechanism | Use when |
|---|---|---|
| **Event into the live session** | Inject a system event; the next turn sees it | The user is present and the finding should steer the conversation |
| **Isolated turn** | Run a separate turn with its own context; post a bounded summary back to a queue | The user is absent, or the work is long, or you do not want it in the transcript |

The isolated turn is the safer default. It cannot corrupt the live context, it cannot be
interrupted by the user mid-write, and its output is a **bounded** artifact you control the size of.

Every scheduled job also needs: **active hours** (nobody wants 3 a.m. output), **per-job history**
(so you can answer "did it run?"), **one concurrent run per job**, and an **acknowledgement
sentinel** so a job knows its last result was seen.

## Cron vs heartbeat

Two different questions that look alike:

- **Cron** — *"do this at this time."* The schedule is the trigger.
- **Heartbeat** — *"check whether anything needs doing."* The state is the trigger; the interval
  is just how often you look.

Using cron for a heartbeat gives you work that fires when nothing has changed. Using a heartbeat
for cron gives you work that drifts. Most "notification" features are heartbeats wearing a cron
schedule, and the giveaway is a job whose first action is to check whether it should do anything.

## Never let background output become user input

> **Observed** · A background notice could be laundered into a user-stated fact. The worker wrote a
> finding, the finding was injected, and the next turn treated it as something the user had said —
> including as **consent**.

Every worker-originated injection is framed as a system notification that is **neither user input
nor consent**, and a fact derived from one can never be recorded as user-stated. Enforce it in the
type, not in the prompt.

This is the same boundary as [02](02-system-prompts.md)'s rule that injected text never shares a
block with the user's own words — and the reason both matter is that the *provenance* of a sentence
decides what the system is allowed to do with it.

## Delivery needs a told-state

A notification system without a record of what the user has already been told will repeat itself,
and repetition is how users learn to ignore the channel — including on the day it matters.

Keep a told-state table with three things: what was said, what value it was about, and whether it
was acknowledged. Then:

- **Compare fingerprints, not just subjects.** An acknowledgement of a finding at one severity must
  not silence a worse later value — see [05](05-memory.md).
- **Route by canonical key.** If the key is already in the deep link you were going to send, the
  told-state lookup costs nothing.
- **Suppression is a decision, so it needs the unknown state.** "Not told" and "cannot tell whether
  we told them" are different, and only one of them is safe to suppress on.

## Escalation is exempt from everything

Whatever brevity, density, deduplication or suppression rules you build, **genuine escalation must
be structurally exempt** — enforced so the tuning machinery cannot see it at all.

Two reasons. The obvious one is harm. The subtle one is that it lets you tune everything else
aggressively without a safety argument every time, which is what makes the rest of the system
improvable. A carve-out you have to remember is not a carve-out.

Give criticals no subject line at all if the subject would itself be a suppression key.

## Mid-turn input needs a stated policy

With real users this happens constantly, and having no policy means the behaviour is whatever your
queue happens to do.

Five policies worth naming:

| Policy | Behaviour |
|---|---|
| **collect** | Buffer; deliver with the next turn |
| **steer** | Apply at the next tool boundary |
| **follow-up** | Queue as a separate turn after this one |
| **steer-backlog** | Apply now, keep the rest queued |
| **interrupt** | Stop the current turn immediately |

Map by shape, not by length: **input that looks like an emergency interrupts and leads the next
turn**, a correction steers at the next tool boundary, and everything else collects.

## Lanes, and one run per session

Per-lane semaphores with **different limits** for foreground, subagent and scheduled work. A
background sweep must not be able to starve a live conversation, and it will if they share a pool.

Then: **exactly one run per session key.**

> **Observed** · Nothing enforced this, so two browser tabs or one retried request could corrupt the
> message log, the plan state and the notice drain simultaneously.

## Design for the crash, not the happy path

- **An orphan sweeper.** Runs left marked `running` after a process dies never resolve on their
  own, and every "why is it stuck" report traces back to one.
- **Duplicate suppression should fail open.** A ref-counted claim registry that dedupes concurrent
  escalations must, when it cannot decide, deliver twice rather than zero times. Failing closed on a
  notification path means the guard becomes the outage.
- **A queued job kind with no worker draining it is a silent drop** — the same shape as a refusal
  with no destination ([03](03-tools.md)). Assert that every kind you can enqueue has a consumer.

## Cold paths need the same budget as warm ones

> **Observed** · A page resolved its own data by building a URL from the incoming request's `Host`
> header and fetching it — so the render's read re-entered the single process performing the render,
> on a 700 ms budget. Measured at the same instant: **14.48 s through its own origin, 9–13 ms
> straight to the source.** The data was always 10 ms away; the budget went entirely on the hop.

Two lessons that generalise past the plumbing:

- **A timeout on the same event loop as the work is not a guarantee.** A saturated process blows any
  budget however close the data is.
- **Therefore losing the race must not be spendable as evidence.** Three states — stated,
  consulted-and-silent, never-consulted — and a fallback that treats the last two alike will present
  a confident answer it has no basis for. Resolving a value and *addressing it to a person* are two
  different claims; let the second one fail independently.
