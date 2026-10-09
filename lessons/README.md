# Lessons — what building one actually taught us

[Start with the domain-neutral guide](../README.md). These nine docs preserve the original
project's implementation lessons and reported observations. Their underlying experiments were
not rerun for this edition. Numeric thresholds and runtime-specific choices are local examples,
not universal defaults. Repeated principles now link to a canonical section.

`../references/` holds the primary sources — public engineering posts on context and harness
design. Read those for *what the design space looks like*. Read these for *what breaks anyway*.

## Read in this order

| # | Doc | Read it when |
|---|-----|--------------|
| 01 | [The five laws](01-thinking-framework.md) | Always first. Five rules that generate most of the others. |
| 02 | [System prompts](02-system-prompts.md) | Writing or tuning a prompt. Includes the caution dial, which is the failure nobody warns you about. |
| 03 | [Tools](03-tools.md) | Designing a tool, a result shape, or a refusal. |
| 04 | [Context](04-context.md) | Deciding what the model sees, and what to do when it no longer fits. |
| 05 | [Memory](05-memory.md) | Persisting anything across sessions. |
| 06 | [Automation & notifications](06-automation.md) | Work that starts without a user, and output that arrives without being asked for. |
| 07 | [Fault tolerance](07-fault-tolerance.md) | Failover, degradation, and guards that backfire. |
| 08 | [Measurement](08-measurement.md) | Before quoting any number about agent quality. |
| 09 | [Multi-agent](09-multi-agent.md) | Running more than one agent against one repo. |

## The five laws

Read [the five laws](01-thinking-framework.md) for the canonical principles and their examples.
Use the table above to jump directly to the implementation problem you have.

## How to read the instances

Rules are stated generically. Each one carries the concrete case that produced it:

> **Observed** · A chart drew a reading with no reference range as a filled circle — identical
> to one confirmed in range.

Skip the instances on a first pass. Come back to them when you disagree with a rule — the
instance is the argument.

## One warning about this directory

Scaffolding encodes an assumption about what the model cannot do alone, and those assumptions
expire. Some rules here exist because a model of a particular generation needed help. Remove one
component at a time and measure — see [08](08-measurement.md) for why "we removed it and nothing
broke" usually is not a measurement.
