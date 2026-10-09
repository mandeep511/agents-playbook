# Browser-harness: recovery in a capability layer

[Start](../README.md) · [Domain patterns](domains.md)

**Evidence:** local source inspection, 9 October 2026. Repository:
`C:/Projects/terminal-agents/browser-harness`. No browser integration tests were run for this
teardown. Function names below are navigation anchors, not a claim about every version.

## What it is

A CLI/Python interface to a persistent daemon connected to Chrome through CDP (Chrome DevTools
Protocol). An outer agent supplies the decisions. The harness supplies durable browser access,
helpers, and selected recovery mechanisms. Its `SKILL.md` teaches the caller how to use them.

## Mechanisms worth learning from

| Mechanism | Inspected code | Why it matters |
|---|---|---|
| Automatic daemon setup | `src/browser_harness/run.py: main` invokes `ensure_daemon` | Callers do not recreate lifecycle plumbing |
| Stale session recovery | `daemon.py: handle` catches “Session with given id not found”, calls `attach_first_page`, retries once | Repairs a specific failed invariant rather than every error |
| Browser-level routing | `daemon.py: handle` sends `Target.*` without a page session | Recovery/discovery can work when page state is stale |
| Real-tab fallback | `helpers.py: ensure_real_tab` filters internal tabs, then switches if needed | Internal Chrome targets do not become ordinary work pages |
| Navigation race avoidance | `helpers.py: new_tab` creates blank, attaches, then navigates | Prevents a premature ready-state check before navigation starts |
| Bounded session setup | `daemon.py: handle` bounds old-session disable and parallelizes independent domain enables | Recovery itself does not add unbounded serial delay |
| Browser identity probe | `run.py: _local_chrome_listening` probes `/json/version` | An open TCP port alone is insufficient evidence of Chrome |
| Explicit cloud opt-in | `run.py: main` checks `BU_AUTOSPAWN`; explicit CDP configuration blocks auto-spawn | Recovery respects chosen infrastructure and spending boundaries |

## The limit that matters

`attach_first_page` and `ensure_real_tab` can select a different page. A working connection does
not prove the original target survived. Retrying a command after reattachment can therefore act
on a new page. **Transfer the recovery mechanism together with an identity check**, especially
for state-changing commands. This is an inferred design risk from the inspected branch, not a
reported production incident or a tested exploit.

Recommended caller contract:

```text
record intended target + expected origin/state
→ attempt operation
→ on stale session: reacquire connection
→ verify target identity and action preconditions
→ repeat only if effect is known absent or safely deduplicated
→ capture fresh evidence of the outcome
```

For a lost response to a submission, the right next action may be looking up the receipt, not
clicking again. Transport healing and business-operation reconciliation are separate concerns.

## Code, skill, and human judgment cooperate

- **Code:** connection/session mechanics, protocol routing, bounded waits.
- **Skill:** screenshot-first observation, `new_tab` before navigation, post-action verification,
  interaction-specific references, stop at authentication barriers.
- **Outer runtime:** task progress, context, permissions, budget, user interaction.
- **Domain verification:** whether the intended download, edit, or submission actually happened.

The general lesson: hide recoverable plumbing, expose unresolved meaning. A useful tool makes
routine failure cheap without making uncertainty invisible.
