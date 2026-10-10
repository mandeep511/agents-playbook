# Domain patterns

Read only the relevant section. These are architecture sketches, not ready-to-deploy systems.

## Browser

Outer runtime owns the task; capability layer owns browser/session mechanics. Record target ID,
origin, expected state, and observation time. After recovery, verify identity before repeating.
A working tab may be the wrong tab. Verify actual downloads/receipts. Unknown submission status
needs lookup, not another click.

## Video

Keep sources immutable. Store a versioned timeline, stable clip IDs, source hashes, explicit
timebase, and edit history. Tools inspect, apply revision-checked edits, preview, start/inspect/
cancel exports, and validate. Render temporary output; promote after checks. Reconcile existing
jobs on restart. Check decoding, duration, dimensions, audio, captions, transitions, and fidelity
to the brief. Probing cannot judge pacing; sampled frames miss defects. Publishing is separate.

## Robotics

LLM plans; deterministic local controller owns motion safety and emergency stops. Preconditions
include sensor freshness, pose uncertainty, map version, command expiry, actuator state, and
allowed regions. Network loss must not disable local safety. Reconcile physical state before
replay; database rollback cannot undo movement. Start in simulation and use domain-qualified
hardware validation. Acknowledgment differs from completed placement.

## General work / coding

Preserve constraints, patches/files, test results, and unresolved operations. Choose or build
the controller and execution environment to fit the project. Isolate workers or use ownership/version checks. Completion means observed
behavior and valid deliverables, not a polished message. External submission differs from preparation.

## Research

Attach sources and uncertainty to individual claims. Preserve provenance and open questions.
Bound experiment parameters and runtime; record failed trials as well as wins. Protect held-out
data from optimization. Validate the metric against the user's actual purpose.

## Scientific research: local first, remote compute when needed

A small Python harness can combine model calls, file editing, terminal access, literature/data
connectors, and simulation jobs. A server is optional; add one for a UI, collaboration or remote
access. Keep data acquisition, prediction, simulation and interpretation as distinct operations.

| Tool | Contract and evidence |
|---|---|
| Retrieve an existing structure | Database accession/version, URL, file hash, confidence metadata |
| Submit a prediction | Supported service/local installation, terms/access, input/config hash, job ID |
| Run a simulation/analysis | Tool version, units, parameters, seed, resource budget, logs and artifacts |
| Compare/report | Source-linked observations, uncertainty, failed jobs, reproducible commands |

“AlphaFold connection” needs clarification: downloading an existing database prediction,
running a local inference pipeline and using a hosted service have different APIs, terms and
compute requirements. Do not invent a universal endpoint. A scientific model's output is a
prediction, not proof of experimental or clinical validity.

First slice: retrieve a public reference structure, inspect metadata, run a small bounded analysis,
and write a report linked to its inputs. Use local resources for lightweight work; dispatch heavy
GPU/long-running jobs to suitable compute. Export artifacts before deleting remote environments.
Treat this as research infrastructure, not an autonomous wet-lab or clinical decision system.

Sources: [AlphaFold DB FAQ](https://alphafold.ebi.ac.uk/faq),
[AlphaFold 3 repository](https://github.com/google-deepmind/alphafold3). Check current terms,
installation requirements and supported access before integration.
