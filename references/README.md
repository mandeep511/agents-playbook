# Sources worth your time

[Start](../README.md) · [Domain patterns](../guide/domains.md)

Research checked **9 October 2026**; organized **10 October 2026**. Read the first four entries
for foundations; select the rest by the problem you are solving. Project descriptions are
inspected documentation claims, not independently reproduced benchmarks.

## Primary engineering writing

| Source | Read for | Local copy |
|---|---|---|
| [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) | Workflow vs agent; start simple | Link only |
| [Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | Context selection and long-horizon strategies | [01](01-effective-context-engineering.md) |
| [Long-running harnesses](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) | Session handoffs and incremental progress | [02](02-effective-harnesses-for-long-running-agents.md) |
| [Agent evaluations](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) | Outcomes, trials, graders; includes video-editing evaluation | [05](05-agent-evals.md) |
| [OpenAI: Harness engineering — Ryan Lopopolo](https://openai.com/index/harness-engineering/) | Legible environments and architectural invariants | [04 · research note](04-harness-engineering.md) |
| [Long-running app design — Prithvi Rajasekaran](https://www.anthropic.com/engineering/harness-design-long-running-apps) | Planner/generator/evaluator and quality criteria | [03](03-harness-design-for-long-running-apps.md) |
| [Anthropic: Writing tools](https://www.anthropic.com/engineering/writing-tools-for-agents) | Tool interfaces, results, evaluation-driven iteration | [06](06-writing-tools.md) |
| [Mario Zechner: Building Pi](https://mariozechner.at/posts/2025-11-30-pi-coding-agent/) | Minimal core, inspectability, extension tradeoffs | [07](07-minimal-pi.md) |
| [Harrison Chase: Your harness, your memory](https://www.langchain.com/blog/your-harness-your-memory) | Portability and ownership of state/context | [08](08-harness-memory.md) |
| [Anthropic: Managed-agent architecture](https://www.anthropic.com/engineering/managed-agents) | Separate session, harness, and sandbox lifecycles | Link only |

**Read disagreements productively.** Minimalism asks which mechanisms are necessary. Rich
harnesses ask which mechanisms save repeated work. Resolve the tension with your own tasks and
failure traces. Chase's portability argument is a vendor-authored perspective: inspect actual
export/resume guarantees rather than treating all hosted systems as equivalent.

## Repositories: inspect a mechanism

| Repository | Layer | Study this |
|---|---|---|
| [OpenAI Codex](https://github.com/openai/codex) | Agent runtime | Execution boundaries, tools, session behavior |
| [Pi](https://github.com/earendil-works/pi) | Extensible runtime | Small core, extensions, skill loading |
| [Deep Agents](https://github.com/langchain-ai/deepagents) | Runtime/framework | Planning, files, context, delegation |
| [Mini SWE Agent](https://github.com/SWE-agent/mini-swe-agent) | Minimal coding agent | A small complete loop |
| [Karpathy autoresearch](https://github.com/karpathy/autoresearch) | Experiment workflow | Bounded editable surface and metric-driven iteration |
| [Kinewright](https://github.com/CanadaApollo6/Kinewright) | Video domain | Timeline operations shared by humans and agents |
| [Vex](https://github.com/AKMessi/vex) | Video agent/domain | Persistent projects, working copies, validation |
| [Crayotter](https://github.com/idwts/Crayotter) | Video workflow | Traceable intermediate artifacts |
| [OpenTimelineIO](https://github.com/AcademySoftwareFoundation/OpenTimelineIO) | Video representation | Editorial timeline interchange; needs a renderer |
| [Nav2](https://github.com/ros-navigation/navigation2) | Robot navigation | Action/controller boundaries; not an LLM harness |

For local browser-harness, see the [source inspection](../guide/browser-harness.md).
Before adopting code, inspect its current license, dependencies, permissions, maintenance, and
relevant tests. Benchmark scores and repository popularity do not establish production fitness.

## Skills that complement the runtime

| Resource | Useful for | How to use it |
|---|---|---|
| [Agent Skills specification](https://agentskills.io/specification) | Portable skill structure | Small entry point; references loaded on demand |
| [Anthropic skills](https://github.com/anthropics/skills) | Procedural packaging examples | Inspect only relevant skills and their licenses |
| [Remotion skills](https://github.com/remotion-dev/skills) / [official guide](https://www.remotion.dev/docs/ai/skills) | Video composition, captions, preview/render | Pair guidance with a renderer and verification |
| [Build-agent-harness](../skills/build-agent-harness/SKILL.md) | Designing, building, improving harnesses | Read directly or copy the entire skill folder |
| Local browser-harness `SKILL.md` | Browser mechanics and interaction playbooks | Pair caller guidance with executable recovery |

Third-party skills were researched, not installed. Reading one does not authorize its setup
commands, publishing actions, or requested access changes.

## Community signals: hypotheses to test

Small, self-selected samples; votes and promotional posts are not comparative evidence.
Cross-posts of the same project count as one source.

| Discussion | Signal | Experiment suggested |
|---|---|---|
| [LocalLLaMA: harnesses for small models](https://www.reddit.com/r/LocalLLaMA/comments/1vpcj2j/agentic_harness_for_small_models/) | Users discuss prompt/tool overhead and lighter setups on different hardware | Hold model/task fixed; compare schema size, latency, tool correctness |
| [ClaudeCode: harness as environment](https://www.reddit.com/r/ClaudeCode/comments/1tc2av1/the_harness_is_the_environment_rethinking_what/) | Discussion emphasizes environment/context quality and measuring it | Log each context source's contribution; remove stale instructions |
| [Vex author's workflow](https://www.reddit.com/r/AI_Agents/comments/1tbulo6/i_got_tired_of_writing_ffmpeg_commands_so_i_built/) | Author describes FFmpeg editing plus probing/validation | Test interrupted exports, wrong crops, and audio quality |

**A useful conflict:** older Vex Reddit posts call it open source; the inspected README says
source-available, non-commercial. Check the current repository license before reuse.

## X / Twitter: discovery with access limits

- [Karpathy's autoresearch announcement](https://x.com/karpathy/status/2030371219518931079):
  direct X retrieval failed. The [author's repository](https://github.com/karpathy/autoresearch)
  links the announcement and supplies the technical evidence used here.
- [Harrison Chase's harness/memory post](https://x.com/hwchase17/status/2042978500567609738):
  discovered through indexed discussion; the [author's blog](https://www.langchain.com/blog/your-harness-your-memory)
  is the primary text used here. No verbatim tweet claim is relied on.
- Sarah Wooders' memory/harness essay is linked from Chase's article, but direct retrieval
  failed. It remains a research lead, not independently verified evidence in this guide.

Named authors are selected for inspectable work. This is not exhaustive coverage or a claim
of community consensus.

## Archive and deduplication

- **01–03:** original snapshots, retrieved 31 July 2026.
- **05–08:** full snapshots converted with installed `clipper`, 9 October 2026.
- **04:** original short note; web retrieval succeeded, direct clipping failed.
- Preserve source wording in snapshots; put corrections and judgments in authored guides.
- Canonicalize sources by destination URL; collapse tracking variants, mirrors, and cross-posts.
- The Markdown guide is the domain-neutral entry; the HTML is the original stack-specific manual.

[Machine-readable source manifest](manifest.json) records URL, retrieval date, snapshot/note type,
size, and SHA-256 for the eight saved sources.


## Construction and execution research · 10 October 2026

The [construction recipe](../skills/build-agent-harness/references/construction.md),
[compute comparison](../skills/build-agent-harness/references/execution.md), and
[local security guide](../skills/build-agent-harness/references/local-security.md) are original
synthesis with direct primary citations. They are not provider benchmarks or full-page snapshots.

| Read | What to learn |
|---|---|
| [Mini SWE Agent](https://github.com/SWE-agent/mini-swe-agent) | Small Python core with model/environment separation |
| [smolagents](https://github.com/huggingface/smolagents) | Code and tool-call styles; local executor is not automatically a security sandbox |
| [Armin Ronacher on Pi](https://lucumr.pocoo.org/2026/1/31/pi/) | Minimal extensible foundations, an author's perspective |
| [Anthropic: brain and hands](https://www.anthropic.com/engineering/managed-agents) | Separate session, controller and sandbox responsibilities |
| [AlphaFold 3 installation](https://github.com/google-deepmind/alphafold3/blob/main/docs/installation.md) | Real compute/setup requirements; local orchestration does not make every tool lightweight |

Additional community evidence, **anecdotal and sometimes promotional**:

- [Do I need E2B/Daytona?](https://www.reddit.com/r/AI_Agents/comments/1unl2r3/how_are_you_actually_using_agent_sandboxes_like/)
  raises container-versus-service setup and state-management tradeoffs. Do not repeat commenters'
  latency figures or vendor isolation claims as verified facts.
- [Unified sandbox interfaces](https://www.reddit.com/r/AgentsOfAI/comments/1s2lxv4/unified_interface_for_ai_sandboxes/)
  illustrates why a portable adapter still needs provider-specific lifecycle capabilities.
- [Local code sandboxing](https://www.reddit.com/r/LocalLLaMA/comments/1ubp4q3/sandboxing_code_execution_for_ai_agents/)
  surfaces setup friction and divergent security advice. The official Node documentation explicitly
  rejects treating `node:vm` as a security mechanism.

Fresh X searches did not yield sufficiently verifiable primary posts for additional technical
claims. We used official documentation, repos, author blogs and accessible Reddit threads instead
of presenting mirrored posts as verified tweets. Three research agents investigated independently;
recommendations remain editorial judgments, not consensus or product testing.
