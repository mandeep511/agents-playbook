# The five laws

Five rules that generate most of the specific ones. Each was found repeatedly, in layers that
did not know about each other, after being believed fixed.

---

## 1. Absence is not a determination

Three outcomes need three values, and **the unknown one is never the default.**

A boolean cannot carry *yes*, *no*, and *we could not tell*. When you force it to, the third
state collapses into whichever of the other two is cheaper to write — and that is almost always
the reassuring one. Pass the error, not the absence of data, because `null` means two things.

Any encoding with fewer states than the data has is this defect. That includes visual encodings,
which is where it is hardest to see, because no string is wrong.

> **Observed** · Twelve instances across eleven layers — context assembly, an HTTP API,
> extraction, region resolution, flag rendering. A chart drew a reading with no reference range
> as a filled circle, identical to one confirmed in range, until unknown became dashed and
> unfilled.

**It also arrives inside libraries you do not control.** A *disabled* query reporting
`isPending: true` is indistinguishable from one in flight, so a screen gated on another fetch
renders a skeleton forever with no exit. A dismissed dialog left mounted with `data-closed`
means presence is not openness. When a third-party flag conflates "no information" with "no",
derive the state you need instead of reading theirs.

**The hardest sub-shape: a correct fail-closed normaliser read by a second consumer.** A
normaliser that maps absent → unsupported is right for resolution, because unknown must fail
closed. But ask it *"did the record state a value?"* and the answer can never be no — so
"nothing given" and "explicitly unsupported" become one value and a whole branch goes
unreachable, even with a correct fixture. Nobody suspects code that is correct for its own
purpose. **Read the raw field when the question is whether something was stated.**

Assume another instance exists. Every time we have looked, it has.

---

## 2. A pass is only as strong as its population

A check states a property over a set. A green result is a claim about the set it reached, not
about your system. *"40 of 40 routes observed"* is true and reads as more than it is.

**When a check passes, ask what it could not see.**

Three instruments, three blind spots — not a hierarchy:

| Method | Finds | Blind to |
|---|---|---|
| String/AST sweep | The sites you thought of | Anything spelled differently. A single-line pattern cannot see a call that wrapped across lines. |
| Gate over a derived population | Whatever its population reaches | Everything outside the population — and the population is usually narrower than the name suggests |
| Looking at the rendered result | The rest | Slow, unrepeatable, and no coverage claim |

> **Observed** · Removing one rule from one codebase: a sweep found 13 sites, a gate found 8
> more, and opening a screenshot found a third component the other two structurally could not
> reach. Each miss was structural, not careless.

Three ways a population fails, each needing a different fix:

- **You measured the wrong artifact.** A scan of server-rendered bytes on a client-rendered
  product saw 1 element where 37 rendered. The predicate was right; the population was a loading
  skeleton. *Fix: observe the real output, and refuse to fall back to the cheap proxy.*
- **An instrument page masked it.** A component gallery renders everything, so it contributed
  861 hits and made every total look healthy while real pages contributed one each. **The
  instrument page hid the instrument's blindness.** Same page, two jobs; trustworthy for one.
  *Fix: know which job you are doing.*
- **A dynamic route enters as one instance.** One id per route shape means one instance is
  observable and the rest are not. *Fix: genuinely hard — enumerating instances makes the
  population hand-maintained again, which is what deriving it was for.*

**Corollary: a check that cannot fail is not evidence.** Two fallback tests were quietly
satisfied by the authoritative source, so the leg under test never ran. "It returned the right
answer" does not say which branch produced it. Split the I/O from a pure function over resolved
inputs, so every higher-precedence source can be forced absent *and proven absent in the same
assertion*.

**And absence only means something when the set you are checking is complete.** A check over a
string somebody summarised must exempt what it cannot enumerate, rather than call it wrong.

---

## 3. Enumerate permissions, not prohibitions

**You cannot enumerate the absences, only the permissions.** A guard written as a blocklist is
wrong by construction. A guard derived by subtraction from a known vocabulary cannot drift.

> **Observed** · A database trigger listing the blocked cases let a `NULL` value match no clause
> and land in a gated table — allow-by-omission, which is law 1 again: a boolean standing in for
> three states where the unhandled one is the reassuring answer. Inverted to name what is
> allowed and refuse the rest, it immediately surfaced a hole nobody had reported.

**A negated character class is a blocklist, and it is the form nobody recognises as one.**

```
/<\/strong>[A-Za-z]/     ← "the letters that may not follow a closing tag"
```

`Cancel— the link` ships straight through it, because an em dash is not a letter. Rebuilt as the
property that actually matters — *a character may abut the tag only if it cannot begin the next
word* — the em dash fails without anyone having had to think of it.

Same class, twice more in the same file:

- `<[^>]+>` to strip tags forgets that `>` is legal inside a quoted attribute, so
  `alt="ratio > 1"` ends the tag early and leaks markup into text you then treat as prose.
- A negated class capturing a label stopped at the apostrophe in *"the patient's own data"* and
  silently truncated it, because **prose contains quotes**.

**The tell is grammatical.** If you can only say what a class is *for* by listing what it
excludes, invert it.

**Mechanical corollary: an unbounded substring is not a word.** `"ast"` matched *iodinated
contrast*; `"alt"` matched *salt* and *altered*; `"ring "` with a trailing space matched
*offering* and suppressed a real finding. Bind every literal with `\b` and keep stems as
deliberate stems.

---

## 4. Guard the value where it lands

An assertion that sits one boundary away from where the decision is made is decoration.
Classify what a value *is*, wherever it is written — not what container it went into.

> **Observed** · A gate checking table names let "anaphylaxis to penicillin" land as a *problem* —
> visible in one view, invisible to the allergy query. A reader that equates a table with a
> category inherits the same bug.

**An unenforced documented promise is worse than an undocumented one.** A type comment carried
the guarantee *"composed from our own words and a position — never the filename"*, enforced
nowhere, and the client rendered the raw value on the one channel that is spoken aloud,
unskippable, and invisible to visual review.

The comment is not neutral. **It is load-bearing in the wrong direction: it makes the code look
audited.** A reader opens the type, sees the guarantee, believes the invariant holds, and stops —
so the documented promise actively suppresses the check that would have enforced it. Unenforced
and undocumented is a hole. Unenforced and *documented* is a hole with a sign over it saying the
floor is solid.

**When a value crosses a trust boundary, the assertion has to live on the far side of it.** Two
verifiers did assert the invariant — both over values *our own code composed from our own mock's
data*, so neither could ever see a server-supplied one. The tests were right and the population
was wrong, which is law 2 in different clothes.

**Corollary: if a shared component states a fact about its caller, make the caller supply it as
a required field with no default.** Then the compiler asks the question instead of a reviewer
noticing.

```
Record<Kind, Copy>                    // a seventh kind is a compile error
Record<string, Copy> ?? fallback      // a seventh kind silently inherits a lie
```

The tell that finds these: **a shared string sitting beside two or three maps keyed off the same
discriminant.** The maps got the treatment and the sentence did not, which is exactly why it
drifted.

But the rule is not "tighten every map". **Tighten when the key space is a closed discriminant
and a miss would render something false. Stay loose when the key space is open and a miss
correctly renders nothing.** A loose type is a bug only when the fallback is a claim.

---

## 5. Every guard has a blast radius

### A gate can cause the harm it exists to prevent

> **Observed** · A banned-word check over device-claim verbs would fail the page that disclaims
> them — *"does not diagnose, treat, cure, or prevent any disease"* is the standard form and
> cannot be written without them. A literal check there pressures someone into softening the most
> load-bearing sentence in the product.

Word gates must be polarity-aware or scope-exempt. **Prose about a hazard is not exempt from the
hazard** — the same shape took down every dev server on the project when a comment documenting a
gitignore glob ended in `*/` and closed its own block comment early, while warning about a
different way to break the build.

### Never condition a penalty on the presence of an honest disclosure

A gate fired when an answer omitted a verified rule **and** admitted it could not verify
something. The cheapest way to pass was to delete the *"what I could not check"* paragraph — and
an answer that omitted the rule *and hid the gap* passed clean.

**A gate whose easiest satisfaction is less honesty is worse than no gate.**

### Never make a gate go red when someone improves something

A ratchet whose count may only fall should print the command to lower its own baseline, not fail
on a decrease. A gate that punishes progress is a gate people switch off.

### Caution compounds into uselessness

This is the failure nobody warns you about, and it does not look like a bug in any single place.

Every individual refusal is defensible. Stacked, they produce a product that cannot do the thing
it exists to do — and because each one reads as responsible in isolation, code review will not
catch it. The dial gets raised once per feature and never lowered.

> **Observed** · A document-reading product told users *"Nobody here has looked at these images.
> Interpreting a medical image is not something this system does."* Seven separate sites — the
> prompt, a subagent brief, a tool description, an extractor — each independently cautious, all
> collectively describing a tool that refused its own purpose. Fixing it made the prompt
> *smaller*.

**The test: read your refusals end to end, as one document, and ask whether what remains is the
product you set out to build.** Then ask which refusals need to be *warnings* instead. A warning
preserves the capability and informs the user; a refusal removes it. Most over-caution is a
warning that got promoted.

Two structural defences:

- **Keep genuine safety output exempt from every brevity, density and suppression rule**, and
  enforce the exemption structurally so the tuning machinery cannot reach it. Then tuning
  everything else is safe, and you can tune aggressively.
- **Measure precision before shipping a gate that blocks.** See [08](08-measurement.md) — one
  citation gate replayed over 259 real answers flagged 179 pairs at **11% precision**, and it was
  already live and already degrading real output.

---

## Two more that did not fit a law

**Fix the mechanism, not the site.** A truncation fix wired into three of nine sections left the
same bisected name in the other six. When changing a mechanism: enumerate every call site, fix
all or none, **and state the count.** The enumeration has to cross layers.

**An example in a spec will be rendered as real content.** Three times, in three layers. A
prompt's worked example handle was copied verbatim by a live model into two answers. A units
dialog rendered *the copy deck's illustration* to a real user, where the illustrative number
happened to be that user's critical value from a different test — and read as normal.

Write examples with values that **cannot** be mistaken for real ones, mark a template as a
template with its slots visible, and where the harness can, reject the example on sight.
