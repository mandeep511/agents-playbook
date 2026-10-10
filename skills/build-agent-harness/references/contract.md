# Harness design contract

A compact output structure; it does not require a large framework.

| Decision | Required content |
|---|---|
| Task | Real input, work product, observable success, non-goals |
| Scope | Controller / capability / domain / evaluation; components built or reused |
| Stack | Language, model adapter, product surface; why these fit the task |
| Execution | Host/container/VM/service; CPU/GPU, dependencies, network, state lifetime |
| Trust | What executed code can read/change; credentials kept outside; isolation chosen |
| Action | Name, typed arguments/units, target ID, expected revision, side effects |
| Authority | Scope, credentials, authorization reference, executor enforcement |
| Result | Pending/succeeded/failed/unknown; evidence; external operation ID |
| Persistence | Intent before dispatch; outcome after reconciliation; artifact provenance |
| Recovery | Failure class, lookup path, safe retry rule, ceiling, stop condition |
| Control | Task-wide limits, cancellation, single writer or revision check |
| Context | Stable constraints plus current observations; retrieve detail on demand |
| Verification | Mechanical checks, subjective rubric, baseline/held-out cases, acceptance thresholds |

```text
planned → authorized → intent_saved → dispatched → verified
                                      ↓
                                    unknown → reconcile → verified / failed / needs_input
```

Authorization may already be supplied by the user's scope; do not add a dialog to every
reversible local action. Re-check when the action, target, or authority materially changes.
One task-wide retry budget prevents nested adapters from multiplying attempts silently.

Adapt these failure cases: lost response after success; crash before/after effect; stale target;
conflicting writer; malformed arguments; rate limit; expired credentials; cancellation; context
reset; hostile retrieved instructions; unavailable grader; corrupted fixture.

Evidence bundle: task/fixture version, model/settings, harness revision, trace, artifacts,
checks run, failures/skips, resource usage, final outcome. Report verified tasks / attempted
tasks; include unresolved runs in the denominator.

For existing systems, change one failing mechanism and replay its trace. For new systems,
produce one useful verified artifact before adding orchestration.

Target identity must include the domain object and account/workspace when relevant; a matching
URL alone is insufficient. Distinguish a client-generated operation ID persisted before dispatch
from a server-generated job ID recorded after acknowledgment. A lost acknowledgment requires
lookup by the client correlation key or external state. Cancellation does not undo completed effects.
