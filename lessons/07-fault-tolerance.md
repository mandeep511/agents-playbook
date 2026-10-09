# Fault tolerance

Agents fail in the middle of things. The question is never *"did it work"* — it is *"what is true
now, and can the next turn find out."*

## Classify before you retry

A single `retryable` boolean is the most common failure-handling bug, because it puts a malformed
request in the same bucket as a rate limit. Retrying a bug is just a slower bug.

Map provider status to a **reason**, and give each reason its own behaviour:

| Reason | Behaviour |
|---|---|
| Rate limit | Jittered backoff, retry |
| Timeout | For a safe read, retry within budget. After a possible write, mark outcome unknown and reconcile before repeating |
| Billing / quota | Stop. Cooldown, surface it. Never retry — you will burn the remaining balance faster |
| Auth | Stop. No amount of retrying fixes a key |
| Malformed request | Stop. **This is your bug**, and the retry hides it |
| Output overflow | Parse the error, retry with a reduced budget |

For transient model/read failures, use bounded jittered backoff. Fall back only when the
alternative preserves required capabilities, data handling, and task semantics. Auth, malformed
requests, and uncertain writes need their own paths. Share one task-wide retry budget so nested
layers cannot multiply attempts. See [the crash window](../guide/build.md#durable-state-without-a-framework-explosion).

## Models are config, so a defect can hide behind whichever one a role points at

> **Observed** · Sending no tool array while assistant tool calls remained in history is a hard 400
> on one model tier and silently tolerated by another. The bug was in our request builder the whole
> time; which model the role pointed at decided whether we saw it.

Name **roles**, not SKUs — `MAIN`, `CHEAP`, `HARD`, `VISION` — so swapping a model is a config
change. Then the rule that follows: **anything depending on which model backs a role must be tested
against every tier you might point it at.**

And keep the reasoning for each choice next to the config, not in a document. The next person to
change it is reading the config.

## A guard in three places is enforced at the strength of its weakest copy

> **Observed** · Three separate definitions of "is this a live test that spends money" existed in
> three files, agreeing on the primary check and nothing else. The suite billed a real account on
> every run for weeks, and the weakest copy is the one that decided.

Collapse duplicated guards to one, then **add a test that fails if any file re-declares its own.**
The duplication is not the bug; the ability to re-introduce it silently is.

Related, and worth its own line: **a spend guard that cannot see the real balance will authorise
more than exists.** State that limitation where the limit is defined, or the number reads as a
guarantee.

## Put hard invariants where a sibling cannot reach around them

Application code cannot be the only guard for something another module can write directly. If the
invariant matters, it belongs in the database — a constraint or a trigger — because that is the one
layer every writer goes through.

Two corollaries:

- **Enumerate permissions, not prohibitions** ([01](01-thinking-framework.md)). A trigger listing
  blocked cases lets an unhandled value through by omission.
- **Guard the value, not the destination.** Classify what a string *is*, wherever it is written.
  Checking which table a row went into is checking the wrong thing.

## Your own tooling is the least trustworthy instrument you own

This section exists because every item in it produced a **false negative about a safety mechanism.**

**`$?` after a pipeline is the status of the last command, not of the one you care about.**

```bash
cmd | tail; echo $?          # measures tail, which succeeds essentially always
cmd >/dev/null 2>&1; echo $?  # measures cmd
```

A gate that correctly exited non-zero reported 0. A gate that crashed at import reported 0. Two
engineers hit this on the same gate on the same day, in opposite directions. **The trap is triggered
by its own preconditions** — you reach for a pipe precisely because the output is long, which is the
same condition that makes you want to check an exit code. So it will keep happening. Set `pipefail`,
and treat any exit code read through a pager as unmeasured.

**A summary line is an instrument too.**

> **Observed** · A safety gate printed `SAFETY GUARD: PASS` on a run that executed 9 of its 17
> sections. It took no arguments, and the absence of a flag was read as a decision that an offline
> run was wanted — so the whole rendered half never ran, including assertions that exist because of a
> P0. The result had already been reported upward twice as evidence.

Four rules, and the middle two generalise furthest:

1. **The undetermined state was in the entry point of the tool whose job is to catch undetermined
   states.** Nobody chose offline; a flag was missing. Defaulting to strict would have been the same
   defect in better clothes — so *neither* flag is now an error that names both runs and exits
   non-zero. **The shortest command must be the complete run**, because a repo where the thorough
   variant has the longer name has a nine-section gate.
2. **A qualifier inside the matched string is not a qualifier.** The incomplete verdict is spelled
   `INCOMPLETE`, never `PASS (incomplete)` — people and scripts grep for `PASS`.
3. **A skip announcing itself in the grammar of a result is how a skipped test becomes a passing
   one.** The skip line was shaped exactly like a section header, so the run's own observer counted
   the section as having run. The *skip's* grammar had to change, not the parser.
4. **Report the skipped set by observing which sections printed**, not by listing which ones need a
   server. A list goes stale the first time a section moves, and observation also catches a section
   that throws before printing its header. **Print what ran against the population** — a zero beside
   a forty is legible where a missing line is not.

**And make sure the gate can run at all.** Ours invoked a runner it never declared, so a clean
checkout could not execute the check that protects the safety rules.

## Degrade to one source, never merge two

> **Observed** · A stale but plausible region cookie supplied the *content set* while the record
> supplied the *numbers*, so an emergency page rendered a UK emergency number beside a live, tappable
> US poison-control line, with the UK alternates gone.

The instructive part: **a garbage cookie degraded cleanly. Only the plausible one blended two
sources into a page that looked authoritative and was internally inconsistent.** Your fallback
testing probably uses garbage inputs.

Resolve such a value **atomically from one source**, and never merge fields from sources that
disagree. Then derive the control-flow decision **once** and export it — two derivations of one
decision is the same defect waiting to reappear.

## Make the environment reproducible, with a test

> **Observed** · Seven third-party packages were imported and never declared. A fresh clone answered
> a user with *"DOCX would not open: ModuleNotFoundError: No module named 'docx'"* — and the package
> that turns a scanned document into pixels was one of them, so a whole input class silently did not
> work.

The fix is not a seventh entry in the manifest. It is **a test that walks every source file, resolves
distribution names to import names, and fails on any undeclared third-party import.** Verify it by
removing a declaration.

Mark genuinely optional heavy dependencies as extras rather than omitting them: **invisible and
optional are different claims, and only one of them is true.**

## Remove the affordance, do not disable it

A control that cannot be flipped survives every attack on it. Prefer absence over `disabled`.

**Corollary: a disabled control must name the step that would enable it, as an action and not as a
count.** "Choose a unit for X and Y", never "2 units need choosing". A correct gate with an
uninformative message is still a dead end.

## Read provenance before you transform

> **Observed** · A file's source format was read *after* an orientation transform that returns a
> **copy** — and the copy carries no format. That provenance field was empty for every upload the
> parser had ever handled, and nothing ever errored.

Any transform that returns a new object may drop metadata. Capture provenance at the boundary, from
the original, before anything touches it.

## Miscellaneous, each found the hard way

- **A comment can break the build it describes.** Documenting a glob ending in `*/` closed the
  surrounding block comment early and took down every dev server — while warning about a *different*
  way to break the build. Prose about a hazard is not exempt from the hazard.
- **A high-resolution render can exceed a provider's image size limit** and fail the whole request,
  not just the image.
- **Atomic file replacement raises on Windows** while a reader holds the file.
- **Keyword search with AND-joined tokens returns nothing** for a natural-language question.
