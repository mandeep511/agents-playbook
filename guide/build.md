# Build and evaluate

[Start](../README.md) · [Fundamentals](fundamentals.md) · [Domain examples](domains.md)

## The first deliverable: one page of decisions

Fill this before implementing. Unknowns stay explicit; do not invent production requirements.

| Decision | Concrete answer must name |
|---|---|
| User and task | One real request and its work product |
| Scope | Runtime, capability, domain, or evaluation harness |
| Build or reuse | Controller, tools, renderer, storage; choose based on the task |
| Placement | Language, product surface, execution host, trust boundary and resource needs |
| Completion | Observable checks and who grades subjective quality |
| Authority | Allowed effects, credentials, destinations, escalation boundary |
| Limits | Time, calls, spend, concurrency, cancellation behavior |
| State | What persists; who owns it; version/concurrency rules |
| Recovery | Failure classes, reconciliation, retry ceiling, stop condition |

## A minimum action contract

Illustrative data contract; adapt fields to the operation. This is not executable code.

```yaml
operation_id: export-017
kind: render_video
input:
  timeline_revision: 8
  source_manifest_hash: sha256:example
  destination: outputs/clip-017.mp4
preconditions:
  expected_project_revision: 8
  source_access: read_only
control:
  deadline_seconds: 120
  cancellation: stop_renderer_and_keep_diagnostics
  authorization_ref: task-041-local-export
retry:
  mode: lookup_before_retry
  idempotency_key: task-041-export-017
result:
  status: pending | succeeded | failed | unknown
  external_job_id: renderer-092
  evidence_ref: artifacts/export-017-validation.json
```

An idempotency key works only if the receiver enforces it. Local bookkeeping alone cannot
prevent a remote service from applying an action twice. Persist the client operation ID before
dispatch; a server-generated job ID can only be saved after acknowledgment. Provide a lookup
path for lost acknowledgments. Bind approvals to the exact action,
arguments, destination, and relevant state version; changed proposals need re-evaluation.

## Durable state, without a framework explosion

Start with files for single-writer artifacts or a transactional store for concurrent operations.
Keep model conversation separate from the records used to authorize work.

| Record | Minimum contents |
|---|---|
| Task | Goal, constraints/version, permitted effects, budget, status |
| Operation | Intent, arguments/hash, target identity, external ID, attempt, outcome |
| Observation | Source, timestamp, object/revision, data, uncertainty |
| Artifact | Path/ID, version/hash, provenance, validation result |
| Event | Run/operation ID, transition, timestamp, error class, cost |

**Crash window:** intent saved → request sent → remote effect → response saved.
A crash between effect and response creates an unknown outcome. On resume, look it up;
retry safely only with a supported deduplication contract, otherwise require reconciliation.
Use one writer or revision checks for mutable domain state; a second worker must not silently
apply edits to an old version.

## Build in slices

| Stage | Build | Exit evidence |
|---|---|---|
| 0 · Baseline | Small custom loop or suitable engine + simplest tools | Saved trace of one realistic task |
| 1 · Vertical slice | Observe → one action → verify | Useful artifact, actual outcome check |
| 2 · Interruption | Persist/reconcile/resume/cancel | Crash tests leave no unexplained effects |
| 3 · Domain quality | Expert rubric + preview/review | Output meets both mechanical and taste criteria |
| 4 · Operation | Budgets, isolation, traces, scoped credentials | Failures are diagnosable and bounded |
| 5 · Optimization | Routing, memory, workers only as justified | Repeated comparison beats baseline on chosen metrics |

Keep the smallest slice deployable. “Can export a verified clip and resume safely” teaches
more than six agents exchanging plans without producing a video.

## The failure suite

| Inject | Required behavior |
|---|---|
| Tool succeeds but response is lost | Reconcile; no duplicate side effect |
| Process killed before/after action | Resume from durable intent and observed outcome |
| Wrong tab / stale project revision | Reject or reacquire correct target before acting |
| Expired credentials | Stop and identify reauthentication need |
| Rate limit / transient read failure | Bounded backoff within shared task budget |
| Malformed arguments | Repair arguments; do not blindly repeat |
| Context reset | Preserve goal, constraints, pending actions, evidence pointers |
| Hostile instructions in a document/tool result | Treat as data; preserve authorization boundaries |
| Two writers | Conflict detection or isolation; no silent overwrite |
| Cancellation during long operation | Propagate cancellation; reconcile residual effects |
| Grader unavailable or fixture corrupt | Incomplete evaluation, never a pass |

## Prove improvement

Use the same tasks, fixtures, model/settings, environment, tool versions, and budgets.
Run repeated trials; separate development cases from held-out cases. Save every trace,
including failures. Report denominators and skipped cases.

- **Verified completion:** passing outcomes / attempted tasks; include unresolved outcomes.
- **Reliability:** recovery success, duplicate effects, constraint violations, interventions.
- **Efficiency:** cost per verified completion, wall time, tool calls, clean-run overhead.
- **Quality:** domain rubric calibrated against human examples; inspect disagreements.

Define thresholds from the task's consequences. Zero observed violations is not proof of zero
risk. A single successful demo is not a reliability estimate. Pair automatic checks with human
review where quality is subjective. [Evaluation methods](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents).

## Diagnose before adding scaffolding

| Symptom | First inspect |
|---|---|
| Wrong action | Observation quality, tool overlap, missing preconditions |
| Repeated failure | Error classification, stale state, retry ownership |
| Premature “done” | Independent completion checks, unresolved work |
| Forgetting | Context assembly and durable constraints |
| Expensive but mediocre | Tool/schema overhead, serial waits, uncalibrated critic |
| Works only on your machine | Dependencies, fixtures, credentials, implicit environment |

Fix one mechanism, replay the failed case, then check regressions. Preserve a rollback path.
