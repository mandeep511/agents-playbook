# Harness fundamentals

[Start](../README.md) · [Build](build.md) · [Domains](domains.md) · [Sources](../references/README.md)

## 1. Know what you are building

An **agent** chooses its next action from observations. A **harness** supplies the loop,
interfaces, state, boundaries, and feedback that make those choices useful.

| Term | Owns | Does not guarantee |
|---|---|---|
| Model | Predictions and decisions from supplied context | Truth, persistence, or successful execution |
| Agent runtime harness | Model/tool loop, context, sessions, cancellation, budgets | Domain correctness |
| Capability harness | Reliable access to a browser, shell, renderer, or controller | The user's whole task is complete |
| Domain harness | Domain state, valid operations, work products, domain verification | A new model or runtime is necessary |
| Evaluation harness | Fixtures, trials, graders, score aggregation | Production behavior outside its tested population |
| Skill | Discoverable procedural knowledge and optional helpers | Enforcement, durable state, or authorization |
| Tool / MCP integration | An operation or access interface | Recovery, isolation, or task orchestration |

This taxonomy is an editorial design aid; projects use “harness” differently. Separate the
responsibilities before comparing products. [Agent/eval definitions](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents).

## 2. Think in feedback loops

```text
restore → observe → choose → validate → record intent → execute
             ↑                                    ↓
             └──── persist ← verify ← reconcile result

Exit: verified success | needs input | budget exhausted | cancelled | failed
```

The model chooses under uncertainty. Deterministic code handles exact checks. The environment
provides evidence. **A tool returning successfully is only evidence that a tool returned.**

Example: `export_video` returning a job ID proves submission. A completed job proves rendering.
A decodable, reviewed file matching the brief proves delivery quality. Publishing that file is
another action with its own authorization and receipt.

## 3. Eight design surfaces

| Surface | Design question | A useful artifact |
|---|---|---|
| Goal | What counts as success, partial success, or refusal? | Acceptance criteria |
| Observation | Can it see the actual state and what is missing? | Timestamped, identified observations |
| Actions | Can it express the intended operation unambiguously? | Typed tool contracts |
| State | What remains true after process/context loss? | Durable task + operation records |
| Context | What must it know for the next decision? | Small working view + retrieval pointers |
| Control | Who may do what, where, for how long? | Executor-enforced policy and budget |
| Recovery | Is repeating this action safe? | Error taxonomy + reconciliation path |
| Evaluation | What independently demonstrates the outcome? | Artifact checks + calibrated quality rubric |

**Design the observation/action pair together.** An agent cannot reliably edit a timeline it
cannot inspect, or repair a failure whose tool result hides the cause.
[Tool design guidance](https://www.anthropic.com/engineering/writing-tools-for-agents).

## 4. Put each responsibility in the right place

| Put it in… | When it is… | Example |
|---|---|---|
| Code | Exact, repeatable, enforceable | Validate timestamps; bound cost; enforce path scope |
| Tool contract | Necessary to call correctly | Expected timeline revision; units; side effects |
| Skill | A reusable procedure requiring judgment | How to choose a strong opening shot |
| Prompt | Stable role and decision priorities | Ask for evidence before declaring completion |
| Durable state | Needed after restart or compaction | User constraints; unfinished render ID |
| Eval | A claim that the behavior actually works | Restart during export without duplicate publication |

A paragraph saying “never overwrite originals” is weaker than a read-only source mount.
A paragraph saying “be tasteful” is weaker than examples of good and bad cuts plus a rubric.

## 5. Use the least complex control structure that fits

| Situation | Start with | Add complexity when… |
|---|---|---|
| All steps are known | Deterministic workflow | Inputs require judgment at a specific step |
| Next step depends on observations | Single-agent tool loop | A measured failure needs another mechanism |
| Long task crosses sessions | Loop + durable milestones | Reconstructing state becomes unreliable |
| Independent investigations | Bounded workers + evidence handoff | Parallelism or context isolation repays coordination |
| Subjective output | Generator + separate review | Checks alone cannot grade the requested quality |
| Physical actions | Planner above deterministic controller | Never use an LLM as the emergency-stop mechanism |

Multiple agents introduce merge conflicts, stale briefs, duplicated costs, and correlated errors.
A second model is not automatically an independent source of truth.
[Workflow/agent tradeoffs](https://www.anthropic.com/engineering/building-effective-agents).

## 6. Context is a view; state is a record

Keep three things distinct:

- **Authoritative task state:** goal, current constraints, authorized scope, unresolved actions.
- **Working context:** what the model needs now; disposable and reconstructible.
- **Long-term knowledge:** sourced claims/preferences with freshness and correction rules.

Compaction compresses conversation. It must not silently rewrite constraints or settle unknown
operations. A fresh session should reconstruct the next valid action from state and artifacts.
[Long-running harnesses](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents).

## 7. Self-healing means restoring an invariant

| Failure | Restore | Then verify |
|---|---|---|
| Browser session died | Connection and intended target | Identity and fresh page state |
| Element moved | Current observation | Correct element and changed state |
| Render crashed | Valid job/project state | Output integrity and project revision |
| Tool timed out after a write | Knowledge of external outcome | Receipt lookup; deduplication guarantee |
| Context filled | Decision-relevant context | Constraints and pending operations survive |
| Robot lost localization | Safe physical state | Fresh localization before further movement |

**Retrying uncertainty can duplicate reality.** Use an `unknown` outcome when execution may
have happened. Reconcile before repeating non-idempotent work. A successful reconnect does not
prove the original target, permissions, or preconditions still hold. See the [local teardown](browser-harness.md).

## 8. The ideas worth exploring

- **Make expertise executable:** turn a domain expert's checklist into observations, actions, and graders.
- **Build a work product the human can edit:** timelines, CAD plans, experiment logs, and patches beat opaque transcripts.
- **Give the agent cheap rehearsal:** preview renders, browser readbacks, simulation, dry-run plans.
- **Search candidates, then judge:** compare several cuts, layouts, or hypotheses under the same budget.
- **Learn the workflow, preserve the boundary:** improve skills from failed traces; version and evaluate changes before promotion.
- **Delete a component as an experiment:** if held-out performance survives, keep the simpler harness.

These are design hypotheses, not universal performance claims. The useful innovation is often
one better representation or feedback signal—not another agent.
