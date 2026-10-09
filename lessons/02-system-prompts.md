# System prompts

A prompt is a budget, not a container. Every sentence competes with every other sentence for the
model's attention, so adding one weakens the rest.

## Set an altitude, then hold it

Two failure modes bracket the useful range:

| Too low | Right altitude | Too high |
|---|---|---|
| Hardcoded if-else logic in prose. Brittle, and the model follows the letter past the point of sense. | Strong heuristics plus the reasoning behind them. Generalises to cases you did not write. | Vague principles. Sounds wise, changes nothing. |

The test for too low: if a new case would need a new sentence, you are writing code in English.
The test for too high: delete the sentence and see whether any output changes.

## Give it a hard ceiling

Pick a token ceiling and enforce it in a test. Ours is 5,000.

Without a ceiling, a prompt only grows — every incident adds a paragraph and nothing is ever
removed, because removal feels risky and addition feels responsible. A ceiling turns every
addition into a trade, which is the conversation you want to be having.

> **Observed** · Fixing the over-caution failure across seven sites made the prompt *smaller*.
> The instructions fighting each other were bulk, not safety.

## The caution dial is the failure nobody warns you about

Review the combined effect of refusals across prompts, tools, and workers. The canonical
examples and tuning rules live under [Every guard has a blast radius](01-thinking-framework.md#5-every-guard-has-a-blast-radius).
When changing a stance, enumerate every copy so one layer does not contradict another.

## Examples will be rendered as real content

Three times, in three layers. A worked example handle was copied verbatim by a live model into
two real answers.

- Write example values that **cannot** be mistaken for real ones.
- Mark a template as a template, with its slots visible.
- Where the harness can, **reject the example on sight** at the output boundary.
- Check for contradictions: one prompt warned about a single example handle while two existed in
  different sections.

## State what an empty block means

An empty dynamic block reads as a negative finding. The model sees *"no problems"* where you
meant *"we hold no data"*.

Render an explicit absent marker, and say in the prompt what it means. This is law 1 reaching the
prompt layer.

```
## Current problems
(none recorded)          ← different from an empty heading, and it must be
```

## Never enforce a mechanical rule in prose alone

Prose cannot count. If a rule is mechanically exact, it belongs in code — and the prompt should
say only enough for the model to cooperate with the code.

Where you do state an exact rule, **give a wrong example and a right example.** Rules with a
mechanical form are the ones models get subtly wrong, and a contrast pair costs less than a
paragraph of qualification.

## Measure whether your guard is the binding constraint

> **Observed** · A brevity guard had two parts: a character budget and a line counter. The budget
> was 700 characters; a realistic worst case measured **217**. The budget had never once fired, so
> the line counter was the only guard — and it counted *lines*, not the thing we cared about.

A guard you have never seen fire is a guard you have not tested. Log the near-misses, or assert
the distribution.

**And check that the rule sits on the path the output actually takes.** Prose emitted in the same
completion as a tool call never entered the field the brevity rule read, so the entire guard was
bypassed by a shape nobody had considered — not a wrong threshold, a wrong location.

## Framing rules for injected content

Anything the harness adds to the conversation needs a frame that survives an adversarial
document.

- **Injected reminders never share a text block with the user's own words.** Concatenating them
  lets document content masquerade as instruction.
- **Worker-originated output is a system notification, not user input and not consent.** A fact
  derived from one may never land as user-stated. See [06](06-automation.md).
- **Remembered material is a claim to be checked, not an instruction.** Invert any
  memory-overrides-prompt precedence you inherited from a coding harness: memory must never
  override the safety envelope. See [05](05-memory.md).
- **Put standing caveats next to the data, not thousands of tokens upstream.** Append the caveat
  to the tool result itself, per tool class — model output is a prediction; literature results may
  only be cited by an identifier present in the result; a document read must quote its anchor.

## Write an anti-sycophancy section

If your users are frightened, uncertain, or want reassurance, the model will supply reassurance
over accuracy. This needs its own section; it does not fall out of general instructions.

Coding harnesses tend to have one because they talk to engineers who notice flattery. Consumer
products need it more and usually skip it.
