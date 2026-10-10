---
name: build-agent-harness
description: Design, build, and improve agent harnesses from first principles in any language or deployment, including local research tools, web backends, browser automation, video editing, and robotic task planners.
---

# Build an agent harness

A harness turns model decisions into useful work: observe, choose, act, check, remember, repeat.
Teach and implement that system for the user's domain. Python on a laptop, a web backend, and a
serverless coordinator with remote compute are all valid starting points. No agent product,
framework, provider, or programming language is required.

## Separate the decisions

| Decision | Meaning |
|---|---|
| Agent controller | Software calling the model, dispatching tools, managing state and stopping |
| Language/runtime | Python, Node.js, Rust, or another implementation choice |
| Execution environment | Where commands run: host process, container, VM, managed sandbox |
| Domain interface | Work products, observations, meaningful actions and quality checks |
| Product surface | CLI, notebook, desktop app, web UI or API |

Build a small controller directly, use an SDK, or extend an existing agent when that fits.
Compare these choices against the actual task; do not assume an existing coding agent. Reuse is
an option, not the objective. One Python process can initially hold several responsibilities.

## Design from one real task

Define the input, editable work product, observations, tools, and independent evidence of success.
For a new harness, read [construction](references/construction.md). For deployment and compute,
read [execution choices](references/execution.md); for shell/files, read [practical security](references/local-security.md).
Use only the relevant [domain pattern](references/domains.md).

Produce a compact [contract](references/contract.md): component ownership, tool inputs/results,
durable state, failure behavior, budgets, smallest useful slice, and acceptance checks. Ask only
for facts that materially change the design. State reversible assumptions and keep progressing.

## Build the smallest useful loop

Implement model adapter → context assembly → tool dispatch → execution adapter → result/evidence
→ next turn. Return errors and missing data explicitly. Keep UI and provider details at the edges.
An API-only harness may need no shell or sandbox at all.

For implementation requests, deliver the working slice and relevant checks. For design requests,
deliver implementation-ready decisions; do not invent completed code or evaluations. Prioritize
fast setup for pet projects and add isolation proportionate to what executed code can touch.

Persist constraints, operation IDs, and artifact provenance outside lossy chat summaries. Long
jobs need submit/status/cancel/results, not one blocking request. Recovery must re-check target
identity and preconditions. A timeout after a possible write is **unknown**: reconcile before
retrying; receiver-side deduplication is required for an idempotency guarantee.

## Improve through evidence

Bound total time, spend, retries and concurrency. Check actual artifacts/world state, not the
model's completion claim. Test interruption, cancellation, stale targets and malformed results.
Compare changes on fixed tasks and repeated trials; report counts, skips, cost, latency and
interventions. Calibrate subjective quality against examples or expert judgment.

Add workers, memory services and routing when failures justify them. Keep security enforcement
outside agent-modifiable code; physical safety stays in a local deterministic controller.
Report what works, evidence, limitations and the next useful increment. Provider recommendations
are conditional: verify current APIs, lifecycle, network policy, resource fit and costs before setup.
