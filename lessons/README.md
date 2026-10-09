# Lessons — what building one actually taught us

Nine docs on making an agent robust, fault-tolerant and well-tuned. Everything here was
measured in working, tested code, not reasoned about in advance.

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

Every specific rule in these docs is one of these five wearing different clothes. If you
remember nothing else:

1. **Absence is not a determination.** Three outcomes need three values, and the unknown one is
   never the default.
2. **A pass is only as strong as its population.** When a check goes green, ask what it could
   not see.
3. **Enumerate permissions, not prohibitions.** You cannot enumerate absences, so a guard
   written as a blocklist is wrong by construction.
4. **Guard the value where it lands, not where you hope it was made.** An assertion one boundary
   away from the decision is decoration.
5. **Every guard has a blast radius.** A gate can cause the harm it exists to prevent, and
   caution compounds into uselessness.

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
