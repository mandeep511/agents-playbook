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

Preserve constraints, patches/files, test results, and unresolved operations. Reuse runtime and
execution environment. Isolate workers or use ownership/version checks. Completion means observed
behavior and valid deliverables, not a polished message. External submission differs from preparation.

## Research

Attach sources and uncertainty to individual claims. Preserve provenance and open questions.
Bound experiment parameters and runtime; record failed trials as well as wins. Protect held-out
data from optimization. Validate the metric against the user's actual purpose.
