# Measurement

Read this before quoting any number about agent quality, including your own.

## One run is not a measurement

Same cases, same prompt, same code, same model, re-scored under one grader:

```
24, 25, 27, 22   of 27
mean 24.5 · standard deviation 2.08 cases
```

A single run therefore resolves nothing finer than about **4 cases — 15 percentage points.**

| Difference you want to detect | Runs needed (~95%) |
|---|---|
| 1 case · 3.7 points | 17 |
| 2 cases · 7.4 points | 4 |
| 3 cases · 11 points | 2 |

**Two models within three cases of each other cannot be ranked by a set this size at any feasible
spend — and a tie is not evidence of equality, it is absence of resolution.**

Anything quoted to two significant figures from one run is quoting the noise. A headline of "93%"
was one run's draw from a range of 89–100%.

### So decide on the axes that *are* precise

When the quality signal cannot resolve your choice, stop trying to make it. Decide on things
measured exactly:

- cost per unit of work
- input modality and capability
- **observed failure modes** — a model that fails by returning `inconclusive` is survivable where one
  that fails by inventing an answer is not

That is how we picked a vision model whose miss rate on identical inputs was 25%: a miss degraded to
"inconclusive", and the confident-wrong bucket was empty across every call.

## Your fixture is part of the measurement

Worse than variance, because variance moves the score while this moves the **answer key**.

> **Observed** · Two runs scored 18 and 21 of 27 — outside the variance range above — and the model
> was not why. Unrelated ingest work had written 23 rows into the eval fixture mid-session, touching
> the spine of **13 of the 27 cases**. A case expecting "6.1% on 20 June" now faced a record holding
> a different unit and a later date, so the cases became *wrong about the fixture* and the
> record-reading tests failed for being right. **It read exactly like a model regression.**

**Only ten of the twenty-three came from a deliberate action, and that is the real defect.** The
ingest path wrote rows as unconfirmed, and the read path never mentioned confirmation — so merely
*inspecting* a document's extraction queue wrote rows the agent then read as fact. "I only looked at
it" was not true of that pipeline, and an unconfirmed extraction was indistinguishable from a
confirmed one at read time, which makes review-and-confirm decorative as far as reads are concerned.

### Rules

- **A pass rate is comparable only to another taken against the same fixture state.** A run that
  spans a write to the fixture is **unmeasured, not bad** — the same category as a run that ran out
  of credit.
- **Fingerprint the rows the cases are graded against**, record it beside a code-provenance hash, and
  compare to the previous run. On a change, put **"DO NOT COMPARE THIS RUN TO THE PREVIOUS ONE"** in
  the *written report* — not on a terminal that scrolled past.
- Comparability is relative, so compare to the last run rather than to a pinned constant. And **an
  unreadable fixture is a third state that must never read as unchanged** (law 1).

### What we rejected, and why it is the interesting part

Two obvious fixes were wrong:

- **Superseding the bad rows** — in an append-only store each supersede is itself a permanent row, and
  a visible correction inside the series changes the story the trend cases read.
- **Re-baselining the expectations** — every expected fact names a source a human can open. An answer
  key re-derived from whatever the database happens to hold turns all of those into lies.

Decoupling the expectations from the record is the right end state. **Detection came first
deliberately**, because it is cheap, it protects whichever fixture you end up on, and it turns a
silent false signal into a loud refusal.

## Separate generation from evaluation when needed

Agents grading their own work confidently praise mediocre output, and the effect is strongest exactly
where you need judgement — subjective quality.

**Tuning a skeptical standalone evaluator is far more tractable than making a generator
self-critical.** You can iterate on the evaluator's calibration without touching the thing that
produces the work.

For subjective output, make it gradable: explicit criteria with hard thresholds, calibrated using
few-shot examples that include the score breakdown, not just the verdict.

For an independent review pass, keep the critic separate from the builder and return findings
before edits. This is useful for subjective review; deterministic tests do not require another
agent. Separate agents can share biases, so calibrate their judgments against external evidence.

> **Observed** · A layer passed its own full test suite with four critical defects live.

A critic that retracts a finding after checking properly is doing the job, not failing at it. Budget
for retractions or you will train the critic to be confident.

## Measure a gate's precision before you ship it

Especially a gate that blocks or rewrites output. This is the single highest-leverage measurement in
this document.

> **Observed** · A citation gate was extended from "the reference exists" to "the value stated before
> the reference appears in that row's text". Replayed over the archive — **259 real answers — it
> flagged 179 pairs on 87 of them, and 160 of those were wrong. 11% precision, on a gate that strips
> the citation after two failed rewrites.**

Why it failed is the generalisable part: the field it judged was **prose** — sometimes a truncated
excerpt, sometimes only the latest of several values — so a number missing from it said nothing about
the source. **And the model could not even comply**, because the tool minted one reference for a whole
series, so there was no per-value reference to move to. The cheapest way to pass was to stop quoting
history.

The fix was to stop judging prose: resolve the reference to the actual row and compare against **what
the source holds** — both unit systems, the reference band, every series point, and any difference or
percentage change between them. Arithmetic over the row's own numbers, instead of a word list for
"down" or "by".

**179 flags became 19, every one a real defect, with zero answers newly accused.**

### It was not a latent risk — it had already reached users

Five archived answers carried **19 markers the gate had written in** after failed rewrites, with zero
invented references among them: nineteen real, resolvable citations destroyed on a false accusation,
in output a user read.

### Two rules out of it

- **When one gate has both a decidable and an undecidable leg, measure them separately.** 92 of the
  raw flags were on sources no comparison could ever have decided — and nothing about those flags
  looked different from a true one.
- **One exemption is a false negative bought on purpose.** A bare `%` beside a row not measured in
  percent is read as a *change* rather than a value, because `%` is both a unit and the universal unit
  of change. "Your level fell 74%" must not cost the citation.

## Check the denominator before a zero means anything

**"The gate never fires" and "there is nothing to gate" are the same reading.**

So count the pairs, not the flags. Ours: citation density **4.0 references per answer against 1.8**
before, with **247 checkable pairs against 174**, and zero flags. Denser output *and* more to check is
what makes the zero a result — and one archived answer really did cite nothing at all while holding 55
references, which is what the check was for.

## A check that cannot fail, or never runs

- **A test satisfied by the wrong branch is not evidence.** Two fallback tests were quietly satisfied
  by the authoritative source, so the leg under test never ran. Split the I/O from a pure function over
  resolved inputs, so a higher-precedence source can be forced absent **and proven absent in the same
  assertion**.
- **A check that never executes is the purest form.** An async assertion the runner silently skipped is
  how a threshold breach went unnoticed. **Verify the guard runs before trusting what it guards** — the
  cheapest version is to break it on purpose and watch it go red.
- **Watch the property, not the spelling.** A rule about a minimum size that greps one class name misses
  the same value written three other ways — the difference between 81 violations and the real 87.
