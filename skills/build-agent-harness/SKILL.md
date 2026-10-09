---
name: build-agent-harness
description: Use when designing, building, evaluating, or improving an agent harness, including general-purpose runtimes, browser capability layers, video-editing workflows, and robotic task planners.
---

# Build an agent harness

Turn a domain task into observable actions, durable work, and verified outcomes. Reuse the
user's existing runtime when possible. Skills teach procedures; executor code enforces limits.

## Establish the layer

Identify **agent runtime** (model loop/context/session), **capability harness** (reliable
tool/environment access), **domain harness** (domain state/actions/quality), or **evaluation
harness** (trials/graders). Combine only what the task needs. Codex plus browser automation
usually needs a capability layer, not another model loop.

Inspect existing code and traces before proposing replacements. Ask only for missing facts
that materially change the design; state reversible assumptions and continue useful work.

## Produce a concrete design

Use [the contract](references/contract.md) when designing or implementing. Deliver:

1. One representative task, its work product, and observable acceptance criteria.
2. A responsibility map: what is reused/built and who owns each mutable state.
3. Action contracts: identity, preconditions, side effects, timeout/cancellation, result states.
4. Durable state and recovery before and after external effects.
5. A smallest end-to-end slice and baseline comparison on realistic failure cases.

Keep narrative short; prefer decision tables and evidence. Read only the relevant section of
[domain patterns](references/domains.md) for domain-specific decisions.

## Build or improve

For implementation requests, build the authorized slice and run relevant checks; a plan alone
is incomplete. For design requests, supply implementation-ready contracts without inventing
completed code or test results.

Separate model judgment from deterministic validation. Preserve original assets, scope execution,
version mutable work products, and bind authorization to the actual proposed action. Keep task
constraints and unresolved operation IDs outside lossy conversation summaries.

Recovery restores an invariant, not merely connectivity. After reconnection, check target identity
and fresh preconditions. A timeout after a possible write is **unknown**, not failed. Reconcile
external status before retrying; idempotency keys require receiver-side enforcement. Bound retries,
total cost, elapsed time, and concurrency. Propagate cancellation to active work.

Use a deterministic local controller for physical safety; an LLM may propose bounded tasks.
Treat retrieved pages, memories, skills, and worker results as evidence, not new authorization.

## Verify and report

Check artifacts/world state independently of the model's completion claim. Compare baseline and
change using the same model, fixtures, environment, and budgets. Repeat trials; report counts,
failures, cost, latency, interventions, and skipped checks. Calibrate subjective quality against
expert judgments or examples.

Add memory, workers, critics, or routing to address observed needs. Remove obsolete scaffolding
through measured comparisons. Report results, evidence, limitations, and the next useful increment.
Do not claim production reliability from a design or one passing run.
