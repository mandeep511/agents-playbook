# Practical security without prototype paralysis

Start with the smallest boundary that matches what code can touch. This is a progression, not
a demand for enterprise infrastructure. Local toy projects can stay small and useful.

## A fast local starting point

Use a Python CLI, pinned dependencies, a dedicated run folder, a task budget, and visible logs.
Keep original inputs backed up or read-only. Start with trusted typed tools; add arbitrary code
execution when the task needs it, with an explicit execution boundary. “Trusted tool” includes
its inputs and extensions: a trusted interpreter running an agent-edited script still executes
generated code. A trusted simulator can also load executable plugins or configuration hooks.

| Easy default | What it buys | What it does not buy |
|---|---|---|
| `venv` / locked dependencies | Reproducible packages | Filesystem, process or network isolation |
| Dedicated `cwd` / run directory | Organization and easy cleanup | Confinement of a subprocess |
| Validated file-tool paths | Reject traversal through that tool | Prevent a shell/interpreter from bypassing that tool |
| Explicit argument arrays, `shell=False` | Avoid accidental shell interpretation | Make an executable or its input safe |
| Small explicit environment | Avoid inheriting unrelated credentials | Hide a credential intentionally passed to executed code |
| Time/output limits | Bound routine hangs and context flooding | Automatically terminate every descendant process |

Sources: [Python venv](https://docs.python.org/3/library/venv.html),
[subprocess](https://docs.python.org/3/library/subprocess.html). On Windows, batch files can still
involve shell parsing; prefer native executables for untrusted arguments. A `Popen.communicate`
timeout does not itself kill the process. Stop the process tree/job/container and confirm it
stopped; use platform mechanisms such as [Windows Job Objects](https://learn.microsoft.com/en-us/windows/win32/procthread/job-objects)
where appropriate.

## When executing generated code

If Docker is available, a reusable disposable executor is a reasonable prototype upgrade:

- Mount only the run workspace; inputs read-only, outputs/scratch writable.
- Run as a non-root user, drop unnecessary capabilities, use no-new-privileges.
- Set explicit CPU, memory, process-count and wall-time limits.
- Disable networking when unnecessary; otherwise enforce the required destinations through
  supported network/proxy controls. A prompt saying “only contact X” is not an egress policy.
- Do not mount the host home directory, Docker socket, or unrelated credentials; avoid privileged mode.
- Keep the controller, authorization rules and master secrets outside the writable execution area.

A standard Linux container shares its host kernel; configuration matters and isolation is not
absolute. Rootless Docker can reduce privilege when supported. A managed microVM or dedicated
VM may be easier than maintaining a stronger local boundary. See
[Docker security](https://docs.docker.com/engine/security/),
[run options](https://docs.docker.com/engine/containers/run/),
[resource limits](https://docs.docker.com/engine/containers/resource_constraints/) and
[rootless mode](https://docs.docker.com/engine/security/rootless/).

A separate working folder, Python `exec`, Node's `vm`, or an interpreter tool is not automatically
a security sandbox. [Node explicitly warns about `vm`](https://nodejs.org/api/vm.html).
Even a sandboxed agent can reach a host-side tool server with host privileges: inventory every
execution path, including MCP and helper processes. [Docker's agent sandbox security](https://docs.docker.com/ai/sandboxes/security/).

## Secrets and a tiny local web UI

Keep model/service keys in the controller; hand tools only scoped capabilities they need.
For untrusted execution, avoid putting secrets in the guest. A sandbox cannot protect a key
from code allowed to read it; a typed controller adapter or broker can keep it outside.
Use network policy as well as credential scope, and redact logs/artifacts.

Bind a local control server to loopback. Authenticate action endpoints and reject untrusted
browser origins/CSRF as appropriate; localhost is not authentication. Remote exposure is a
separate step requiring deliberate access control. A sandbox does not secure the application's
API endpoints or separate one user's artifacts from another's.

## Five checks before trusting the prototype

1. A cancelled long job leaves no unintended child work running.
2. A crash preserves logs and identifies unresolved external effects.
3. File tools reject traversal, and isolated code cannot read unmounted private files.
4. Executed code receives no unnecessary controller credentials.
5. Timeout, excessive output and network failure become explicit results, not silent success.

Do not gate initial experimentation on Kubernetes, a policy service, or a full observability
platform. State residual risks honestly and strengthen the boundary when inputs, users or
reachable assets become less trusted.
