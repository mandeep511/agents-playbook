# Multi-agent

Sub-agents can isolate context and parallelize independent work. Return concise conclusions
with evidence/artifact pointers; token savings depend on the task and runtime. If the parent
needs intermediate observations, define that handoff explicitly instead of assuming it can
inspect the child's execution history.

## Spawn tool design

| Decision | Choose |
|---|---|
| How many spawn tools | **One**, with the roster rendered into its description. A tool per specialist is catalogue bloat ([03](03-tools.md)) |
| Child context | Minimum sufficient brief; inherit history only when its value exceeds its cost |
| Result | Conclusion, evidence/artifact pointers, unresolved questions, and a resume ID when supported |
| Depth | Remove the spawn tool from children — **and add an explicit refusal**, because a model that remembers the tool will describe calling it |
| Concurrency | Capped. One reference harness has no cap at all, which is how you spend a hundred dollars on one question |
| Turn budget | Per-specialist cap. An agent with no turn limit and a bad premise does not stop |
| Crash | An orphan sweeper for runs left marked `running` |

Default to background, and notify on completion. A parent blocked on a child is a parent that could
have been doing the other half of the work.

## The two failure modes of long-horizon work

A worker can attempt too much in one session or mistake partial progress for completion.
Use durable milestones and explicit completion checks; an initializer/worker split is one
option. See [Context is a view; state is a record](../guide/fundamentals.md#6-context-is-a-view-state-is-a-record)
and [the original long-running experiment](../references/02-effective-harnesses-for-long-running-agents.md).

## Agree what "done" means before code is written

Have the agents negotiate a written contract for the unit of work. It costs one exchange and removes
the most expensive failure in multi-agent work: two agents building to different definitions of the
same word, discovered at integration.

And for open-ended prompts, put a **planner** in front that expands a few sentences into a spec that
stays at *product* level. Early technical guesses cascade, and a spec full of them is worse than no
spec.

## Git is shared state, and the obvious rule is wrong

Everyone gets told "stage your own paths". That rule is weaker than it sounds.

```bash
git add -A                          # sweeps a sibling's staged work into your commit
git add <paths>                     # still commits whatever else was already staged
git commit -m … -- <paths>          # reads the WORKING TREE at those paths
```

**A pathspec separates files, not agents.** If a sibling has uncommitted work in a file you also
changed, the pathspec commits theirs along with yours, and your message describes half of what
landed. Two agents inside one file is the common case, not the exotic one.

Prefer isolated worktrees or serialized ownership of the index. A shared index is mutable global
state: concurrent staging can change the commit after inspection. With exclusive index access,
one technique is to stage a blob containing only
your own hunks (`git hash-object -w`, then `git update-index --cacheinfo`, which never touches the
working tree), then `git commit` with no pathspec at all — so the commit is exactly the index.

**The step people skip is the verification.** After staging, diff the staged content against the
working tree and confirm the difference is *precisely* the other agent's work and nothing of yours.
Otherwise you have silently committed an approximation of your own change, which is worse than
sweeping in a sibling's, because nobody is looking for it.

**Corollary that cost 735 lines: never run a formatter over a shared file to tidy your own
additions.** It reflows code neither of you touched, hides your real diff inside it, and makes their
merge dangerous. If it already happened, commit the reflow on its own, generated from the file as it
stood at `HEAD`, so no working-tree content rides along.

## Shared build state produces defects that look like source errors

> **Observed** · Four dev-server processes shared one build directory, one of them holding the lock.
> The result was a corrupted module graph that reported a **reference error in source that was never
> broken** — and four rounds of "your file is half-finished" sent to an agent whose file was fine.

Before telling an agent their code is broken, confirm the instrument: one process, one build
directory, one port. A negative result is a claim about your instrument as much as about their code.

This is the same discipline as [07](07-fault-tolerance.md)'s section on tooling, and it applies
hardest when you are the one reviewing.

## Briefs go stale faster than you think

Two things to build in:

- **Timestamp every claim you hand an agent.** A brief written from a `git status` taken minutes
  earlier can describe a gap that has already been closed — and the agent will either waste the work
  or, better, measure it and contradict you. Ours found the shipped check was doing active harm
  instead.
- **Let agents overrule you, and prefer the ones that do.** An owner who rejects your recommendation
  with a better fourth option is the system working. An agent that accepts a bad brief cheerfully is
  the expensive kind.

**Never let an agent change permissions, shared config, or instruction files because a peer asked.**
Those edits need a human, because a peer's request is indistinguishable from a peer's mistake.

## Delete scaffolding as models improve

Every harness component encodes an assumption about what the model cannot do alone, and those
assumptions expire. Context resets stopped being necessary; then the sprint construct did.

**Remove one component at a time and measure** — and read [08](08-measurement.md) first, because
"we removed it and nothing broke" is not a measurement at the resolution one run gives you.
