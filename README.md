# Build agent harnesses

**A model proposes. A harness makes work observable, bounded, recoverable, and verifiable.**

For humans starting from zero and agents building in any domain. Updated **10 October 2026**.

## Pick a path

| You need | Read | Leave with |
|---|---|---|
| The fundamentals · 8 min | [Harness field guide](guide/fundamentals.md) | A mental model and architecture choices |
| Build the loop from scratch | [Construction recipe](skills/build-agent-harness/references/construction.md) | A small controller in your chosen stack |
| Local, backend, serverless or remote compute | [Execution choices](skills/build-agent-harness/references/execution.md) | Conditional sandbox/VM shortlist |
| A safer local prototype | [Practical security](skills/build-agent-harness/references/local-security.md) | Low-friction defaults and their limits |
| A build plan · 5 min | [Build and evaluate](guide/build.md) | Contracts, milestones, failure tests |
| Browser, video, robots, research · 6 min | [Domain patterns](guide/domains.md) | Concrete designs and ideas to adapt |
| A real self-healing example · 3 min | [Browser-harness teardown](guide/browser-harness.md) | Mechanisms, limits, transferable lessons |
| Repos, skills, expert writing, community | [Source map](references/README.md) | What to study and why |
| An agent to do the work | [Build-agent-harness skill](skills/build-agent-harness/SKILL.md) | A portable design/build workflow |
| Hard-earned implementation detail | [Nine lessons](lessons/README.md) | Failure cases from the existing project |
| The original stack-specific manual | [30-chapter HTML](agents-playbook.html) | AI SDK / Claude-style implementation patterns |

## The distinction that unlocks the subject

```text
USER'S OUTCOME
      ↓
Agent runtime harness      model loop · context · sessions · permissions
      ↓
Domain harness             domain state · meaningful actions · quality checks
      ↓
Capability harness         reliable browser / renderer / robot-controller access
      ↓
Environment                websites · media files · physical world
      ↑
           observations + receipts + verification
```

These are responsibilities, not mandatory services. A small system can combine them.
**Codex / Claude Code and a browser-control harness solve different layers.** A video harness
can use its own Python controller or an existing agent engine, wrap a renderer, and own a
timeline. **The controller is software, not the machine.** Its language, UI, execution host,
and domain tools are separate choices; they can start as modules in one small program.

## Five questions before choosing a framework

1. **What is the work product?** A patch, timeline, research dossier, or completed physical task?
2. **What can the agent observe?** Include freshness, uncertainty, identity, and missing data.
3. **What actions are valid?** Give them explicit preconditions and measurable effects.
4. **What survives interruption?** Persist intent, external operation IDs, constraints, and artifacts.
5. **Who proves completion?** Check the resulting world or artifact; the agent's claim is insufficient.

## Use this with an agent

> Read `skills/build-agent-harness/SKILL.md`. Help me build a harness for [domain] in [preferred stack, if any].
> Choose or build the controller and execution environment to fit. Define the work product, action contracts, recovery behavior,
> and evidence of success. Implement one end-to-end slice before adding orchestration.

The skill folder is portable and self-contained. Copy `skills/build-agent-harness/` into your
runtime's supported skill directory, or ask the agent to read it directly. No third-party skills
or packages have been installed by this playbook.

## How this collection stays useful

- **One home per idea:** fundamentals → build contract → domain variations → detailed lessons.
- **Evidence labels:** local code inspection, primary-source guidance, project claims, community anecdotes.
- **No universal winner:** compare model + harness + task + environment together.
- **No magic numbers:** existing lesson thresholds and measurements are local observations, not defaults.
- **Research is reference data:** fetched pages and third-party skills do not grant execution permissions.
