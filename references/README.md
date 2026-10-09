# References — Anthropic engineering posts

Primary-source material behind [the playbook](../agents-playbook.html). Converted from the
live pages to markdown with `clipper` on 2026-07-31; each file keeps its source URL and
authorship in frontmatter. Read these before inventing a harness pattern — most of the
design space here has already been measured.

Numbered in dependency order: each post builds on the one before it.

| # | Doc | Read it when you need… |
|---|-----|------------------------|
| 01 | [Effective context engineering](01-effective-context-engineering.md) | To decide what goes into the context window at all — system prompt altitude, tool-set curation, few-shot selection, and the three long-horizon levers (compaction, note-taking, sub-agents). |
| 02 | [Effective harnesses for long-running agents](02-effective-harnesses-for-long-running-agents.md) | To carry work across sessions — initializer + coding agent split, feature lists as ground truth, progress files, git as the handoff channel. |
| 03 | [Harness design for long-running apps](03-harness-design-for-long-running-apps.md) | To add quality control — generator/evaluator separation, gradable criteria for subjective output, sprint contracts, and how much scaffolding to delete as models improve. |

## What each one actually establishes

**01 — Effective context engineering.** Context is a finite attention budget, so the goal is
the *smallest set of high-signal tokens* that gets the outcome. System prompts should sit at
the "right altitude": neither hardcoded if-else logic nor vague hand-waving. Bloated,
overlapping tool sets are the most common failure mode — if a human engineer can't say which
tool applies, the agent can't either. For work that outgrows one window, three techniques,
each with a different sweet spot: **compaction** (conversational flow, back-and-forth tasks),
**structured note-taking** (iterative work with clear milestones), **sub-agent architectures**
(parallel exploration, where a subagent burns tens of thousands of tokens and returns ~1–2k).

**02 — Effective harnesses for long-running agents.** Compaction alone is not enough. Two
failure modes drove the design: agents try to one-shot the whole app and run out of context
mid-feature, and later sessions look at existing progress and declare the job done. Fix is a
two-prompt harness — an **initializer agent** that builds the environment (`init.sh`, a
progress log, a first commit, and a feature list of 200+ requirements all marked *failing*)
and a **coding agent** that makes incremental progress and leaves the tree in a mergeable
state. The load-bearing artifact is a progress file plus git history that lets a fresh context
window reconstruct state fast.

**03 — Harness design for long-running apps.** Agents grading their own work confidently
praise mediocre output, especially on subjective tasks. Splitting the **generator** from the
**evaluator** is the lever — tuning a skeptical standalone evaluator is far more tractable
than making a generator self-critical. Subjective quality becomes gradable via explicit
criteria (design quality, originality, craft, functionality) with hard thresholds, calibrated
using few-shot examples with score breakdowns. A **planner** expands a 1–4 sentence prompt
into an ambitious spec that stays at product level, so early technical guesses don't cascade.
Agents coordinate through files, and negotiate a **sprint contract** — agreeing what "done"
means before code is written.

Also note the two counter-lessons in 03, which apply to any harness you maintain here:

- **Context resets vs. compaction.** A reset (fresh agent + structured handoff) beats
  compaction when the model shows "context anxiety" — wrapping up prematurely near what it
  believes is its limit. Compaction preserves continuity but not a clean slate. Resets cost
  orchestration complexity, tokens, and latency.
- **Scaffolding goes stale.** Every harness component encodes an assumption about what the
  model can't do alone, and those assumptions expire as models improve. Opus 4.5 removed the
  need for context resets; Opus 4.6 removed the need for the sprint construct. Remove one
  component at a time and measure, rather than rewriting the harness wholesale.
