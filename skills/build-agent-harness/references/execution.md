# Choose where tools execute

**Harness/controller = software. Execution environment = where a tool runs.** Choose independently.
The controller can run on a laptop while simulation runs on a GPU service and data retrieval
runs through a typed API. A sandbox is unnecessary when all tools are trusted bounded APIs.

Recommendations below are editorial starting points, researched **10 October 2026**, not
comparative benchmarks. Verify current APIs, plans, regions, limits and costs before provisioning.

## Choose by workload

| Need | First option to investigate | Why / tradeoff |
|---|---|---|
| Personal prototype, trusted local tools | Python process + disposable run directory | Least setup; direct host privileges unless separately isolated |
| Local generated scripts, reproducible dependencies | Container executor if Docker is already available | Practical boundary with scoped mounts; you maintain image and policy |
| Simple managed Linux execution from Python/TS backend | [E2B](https://docs.e2b.dev/) | Direct SDK workflow for commands/files; account, network and lifecycle costs |
| Existing Cloudflare application | [Cloudflare Sandboxes](https://developers.cloudflare.com/sandbox/concepts/) | Containers for native Linux tools; Dynamic Workers for code calling scoped methods |
| Remote development computer / richer OS needs | [Daytona](https://www.daytona.io/docs/en/sandboxes/) | Multiple sandbox classes; select OS/resources and verify class-specific lifecycle |
| Python scientific compute / GPUs | [Modal](https://modal.com/docs/guide/sandboxes) | Custom dependency images and GPU options; check runtime restrictions/preemption |
| Own long-lived Linux service | Dedicated VM or [Fly Machines](https://fly.io/machines/) | Familiar machine/process model; you own patching, cleanup and access policy |

**A useful default:** local trusted toy → local process; arbitrary generated code → a disposable
container/VM or managed sandbox; web app with multiple users → a per-trust-boundary executor.
No universal winner. Test time-to-first-real-task, dependency setup, idle cleanup, file transfer,
resume semantics and total cost. “Fast startup” without your dependencies is an incomplete metric.

## Serverless does not mean every tool runs in the request

```text
web request → authenticated task record → coordinator/queue
                                      → remote command or job
                                      → durable artifacts + completion event
web UI      ← status/events/artifact references
```

Native binaries, long simulations and renderers often belong in a separate executor. A Worker
or function can coordinate them. Keep owner identity, task state and credentials in the control
layer. Use stable job IDs and deadlines so request termination does not create orphaned work.
Do not run arbitrary user/model code alongside a backend's application secrets or shared database.

## Four meanings of persistence

| State | Owner / recovery question |
|---|---|
| Conversation and constraints | Application store; can a fresh controller reconstruct the task? |
| Workspace files | Volume/artifact store; survive stop, expiry, deletion, or only pause? |
| Processes and memory | Provider/runtime; resume in place or restart from a checkpoint? |
| External jobs and effects | External service + operation ledger; can status be reconciled? |

E2B documents memory/filesystem pause-resume. Cloudflare's workspace snapshots have different
semantics from a process snapshot. Daytona varies by sandbox class. Modal offers snapshot
mechanisms with constraints. **Never translate “persistent” into “everything survives.”** Read
[E2B persistence](https://docs.e2b.dev/sandbox/persistence),
[Cloudflare lifetime](https://developers.cloudflare.com/sandbox/concepts/lifetime/),
[Daytona lifecycle](https://www.daytona.io/docs/en/sandboxes/) and
[Modal snapshots](https://modal.com/docs/guide/sandbox-snapshots).

## Before selecting a service

Check language/native dependencies, CPU/RAM/disk/GPU, startup with your image, maximum job life,
idle timeout, egress/private-network access, scoped credentials, cancellation, artifact export,
region/data handling, tenant isolation, and billing while idle. Read the current SDK example and
run one disposable smoke task; do not provision paid resources without the user's authorization.

Cloudflare's current docs distinguish 1.0 from legacy 0.x; old `getSandbox` examples may target
a different API. GPU support is provider/runtime-specific: verify [Modal resources](https://modal.com/docs/guide/sandbox-resources)
and the selected Daytona class; do not assume every sandbox product has a GPU tier.

A VM or managed sandbox still needs scoped mounts, network and credentials. Prefer typed
service adapters or credential brokering outside generated code. See
[Cloudflare security](https://developers.cloudflare.com/sandbox/concepts/security/) and
[practical local security](local-security.md).
