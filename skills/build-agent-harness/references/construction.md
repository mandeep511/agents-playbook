# Construct a harness from first principles

Use for a new implementation, regardless of stack. This is a design recipe; the sketch below
is not an SDK or a runnable production implementation.

## Start with the world, not a framework

Complete: **The agent observes ___, changes ___ through ___, and proves success using ___.**
Choose a work product the human can inspect: files, timeline, experiment ledger, CAD document,
or task record. Missing observations cannot be repaired with a more forceful prompt.

## Six small parts

| Part | Responsibility | Minimal local implementation |
|---|---|---|
| Model adapter | Request/response, tool-call IDs, usage, stop reasons | One provider SDK wrapped in a function |
| Controller | Next turn, dispatch, budgets, cancellation | Python loop / async function |
| Tool registry | Names, argument schemas, effects, handlers | Dictionary of functions plus validation |
| Execution adapter | Run commands/jobs and retrieve outputs | Local worker initially; remote adapter later |
| State/artifacts | Intent, outcomes, constraints, files, provenance | JSON files for one writer; SQLite for transactions |
| Verifier | Check the user's actual outcome | File/schema checks plus domain review |

A CLI invokes the controller. A web route starts a job and streams/polls events. Neither is the
agent's reasoning loop. A serverless request can coordinate a remote Linux worker without
running native tools inside the request process.

## The loop, with the important branches visible

```text
load task constraints and unresolved operations
reconcile interrupted work before starting new mutations
while task budget remains and cancellation is not requested:
    assemble relevant observations, state and tool descriptions
    ask model for next action or proposed completion
    if completion proposed:
        run independent acceptance checks
        finish if verified; otherwise return actionable failures to model
    else:
        validate tool name, arguments, authorization and current preconditions
        persist operation intent with a client-generated ID
        execute via the selected adapter under a bounded deadline
        persist result or unknown outcome, evidence pointers and usage
        append the matching tool result to model context
record explicit final status and remaining work
```

Preserve the provider's tool-call/result pairing; handle malformed responses and multiple calls.
Serialize conflicting writes. Spill large results to artifacts with bounded summaries and a way
to retrieve the rest. Do not interpret a provider stop reason as proof that the task succeeded.

## Tool design: domain operations plus escape hatches

A typed `inspect_asset`, `fetch_structure`, or `submit_simulation` is easier to validate than
an unrestricted shell command. A terminal adds flexibility when the task needs coding or new
compositions. Both can coexist; neither is mandatory. Tool names should describe outcomes,
with units, object IDs, and machine-readable errors.

For a long job, expose `submit → status → cancel → collect`. Save the client operation ID before
submission, server job ID after acknowledgment, input hash, configuration, logs and output
references. On restart, look up the job instead of blindly submitting again. Status polling is
usually deterministic code, not a new model decision every second.

## Three equally legitimate starting points

| Situation | Starting architecture | First useful proof |
|---|---|---|
| One researcher on a laptop | Python CLI, model adapter, typed tools, local workspace and job ledger | Fetch a public reference artifact, run a bounded local analysis, save a sourced report |
| Web application with backend | HTTP API, queued worker, task database, object storage; isolated execution when needed | Two sessions keep separate workspaces and resume an interrupted job |
| Serverless application | Request/event coordinator, durable task state, external execution service for native jobs | Request returns job ID; worker completes and artifacts survive sandbox teardown |

Use existing software when it saves work; write the controller yourself when a small loop is
clearer. Avoid both framework worship and recreating infrastructure without a reason.

## Move compute without rewriting the agent

Keep a narrow adapter around operations actually needed: run, inspect job, cancel, read/write
artifact. Advertise capabilities such as GPU, network access and resumable process support.
Do not pretend all providers implement the same lifecycle. Switching a local worker to remote
compute requires explicit artifact transfer, environment packaging and failure semantics.

**Design exercise:** run one task manually through the proposed tools. Then give the model those
same tools. Fix the interface where both human and model struggle before adding more orchestration.


## One tool-result shape

Illustrative schema; choose a typed equivalent in your language:

```text
request: {operation_id, tool, arguments, expected_revision, deadline}
result: {
  operation_id,
  status: running | succeeded | failed | unknown,
  job_handle?, artifacts: [{id, hash, media_type}],
  error?: {category, message, next_action},
  observed_at, usage
}
```

Keep job handles in an executor-owned registry. A local PID can be reused: associate it with
launch identity/start time and the task, or use a supervisor/container handle. After controller
restart, verify ownership and status before signalling or claiming a process is still running.
An error envelope is distinct from successful output containing the word “error.”
