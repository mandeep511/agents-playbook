# Domain patterns

[Start](../README.md) · [Build contract](build.md) · [Repos and skills](../references/README.md)

**Same loop, different world model.** These are proposed architectures, not claims that the
listed projects implement every safeguard. Repository features below are README/documentation
claims inspected on 9 October 2026; the projects were not executed.

## The translation table

| Domain | Durable work product | Useful action | Independent evidence |
|---|---|---|---|
| General-purpose work | Task ledger + files | Read, transform, export, submit | Artifact opens; requirements met; receipt |
| Coding | Patch + tests + decision record | Edit, run, inspect, review | Relevant tests and observed behavior |
| Browser | Target-bound task + receipts | Observe, navigate, interact, download | Fresh page state and external record |
| Video | Timeline + media manifest | Trim, reorder, caption, mix, preview | Decodable export, audiovisual review |
| Robotics | Mission + world state | Navigate, inspect, pick, place | Controller telemetry and physical observation |
| Research | Evidence graph + experiment log | Search, extract, compare, test | Source support and reproducible results |

## Video editing: a timeline is the agent's workspace

```text
brief + immutable media → inspect/transcribe → candidate edit plan
→ versioned timeline → cheap preview → review → full render → validate → deliver
```

Use stable clip IDs and rational/frame-based time when precision matters. Declare frame rate,
source-versus-timeline coordinates, audio units, and inclusive/exclusive trim boundaries.
A sentence like “cut from 10 to 15” is ambiguous after earlier edits.

**Example brief:** one 30-second vertical clip, opening claim preserved, readable captions,
no clipped speech, source media unchanged. Agree duration tolerance and audio targets.

| Layer | What to build |
|---|---|
| Controller | Build a small model/tool loop or use an agent engine for planning and dispatch |
| Domain | Versioned timeline, asset hashes, edit operations, undo/history |
| Capability | Renderer with job IDs, progress, cancellation, temporary outputs |
| Review | Technical checks plus pacing, continuity, framing, caption readability |

On crash: inspect the recorded render job. Validate completed temporary output before promotion;
restart partial exports only under the renderer's actual guarantees. Avoid two jobs publishing
the same destination. A successful encoder exit does not detect a bad cut or wrong spoken claim.
Sampled frames can miss defects; inspect transitions and listen to audio as well.

### Study these distinct approaches

| Project | Type | What to inspect | Caveat |
|---|---|---|---|
| [Kinewright](https://github.com/CanadaApollo6/Kinewright) | Desktop editor + agent integration | Human and agent operations on the same timeline | Project describes rough edges; test your formats |
| [Vex](https://github.com/AKMessi/vex) | Terminal editing agent | Stateful edits, working copies, output QA | README says source-available, non-commercial; inspect license before reuse |
| [Crayotter](https://github.com/idwts/Crayotter) | Multimodal video workflow | Intermediate planning and production artifacts | Treat as a research/project reference; validate independently |
| [Remotion skills](https://www.remotion.dev/docs/ai/skills) | Skills for programmatic video | Compositions, timing, captions, previews and renders | Skills + renderer; not a complete editing harness |
| [OpenTimelineIO](https://github.com/AcademySoftwareFoundation/OpenTimelineIO) | Editorial interchange | Portable timeline representation | Interchange component; rendering needs another system |

**Ideas:** dialogue-preserving rough cuts; three opening variants with blind review; podcast-to-shorts
with source-time citations; automatic caption-safe reframing; alternative music mixes with loudness
checks. Generate candidates cheaply, then spend render budget on the best.

## Browser control: recovery must preserve target identity

The outer agent decides the task. The browser capability harness owns connection/session state,
observations, browser actions, and recoverable mechanics. The domain layer knows whether a
report was downloaded or a form was submitted. Read the [local code teardown](browser-harness.md).

**Design pattern:** observe target ID + origin + page state → act → inspect changed state.
Reacquire observations after navigation or recovery. A stale screenshot is not an action plan.
An authentication wall requires the supported authentication path, not speculative clicking.

**Ideas:** export reconcilers, accessibility auditors, evidence collectors, reproducible browser
bug reports. Share domain knowledge as short skills; keep connection repair in code.

## Robotics: separate planning from control

An LLM can propose missions and bounded symbolic actions. A deterministic local controller
owns motion constraints, interlocks, watchdogs, and stopping. The physical world is not undone
by rolling back a database entry.

| Persist | Validate before action | Observe after action |
|---|---|---|
| Mission, command IDs, pose uncertainty, load state, map version | Fresh sensor data, allowed region, actuator/controller health, action expiry | Physical pose, item identity, placement, released reservations |

Network/model loss must leave the local safety system working. Resume only after reconciling
physical state; do not replay “pick” because an acknowledgment was lost. Begin in simulation,
then use a domain-qualified validation process for hardware. [Nav2](https://github.com/ros-navigation/navigation2)
is a navigation-system reference, not an LLM harness or a safety certification.

**Ideas:** inspection missions that request closer views when uncertain; warehouse exception
triage; simulation-generated failure cases; a planner that trades travel time against observation
quality. Keep hard real-time control outside the model loop.

## General work and research: evidence is the work product

Store deliverables, sources, decisions, and open questions independently of chat. Give documents
stable IDs and attach citations to each extracted claim. Separate research from external actions
such as sending, purchasing, or publishing.

[Karpathy's autoresearch](https://github.com/karpathy/autoresearch) illustrates a deliberately narrow
experiment loop: change a bounded program, run under a fixed training budget, compare a metric,
keep or discard. The transferable idea is a controlled search space plus feedback—not an
instruction to let every task run forever. Protect held-out evaluation from optimization.

**Ideas:** literature claim-checking; experiment schedulers; spreadsheet reconciliation; CAD
variants checked against geometric constraints; sound-editing candidates judged on loudness and
listening quality. Every new niche starts by defining its representation, actions, and evidence.

For a laptop-based scientific harness combining structure data, simulations, script editing and
remote jobs, see [the scientific pattern](../skills/build-agent-harness/references/domains.md#scientific-research-local-first-remote-compute-when-needed).
