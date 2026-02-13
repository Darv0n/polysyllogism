---
name: polysyllogism
description: >-
  Polysyllogistic Chain Optimization engine for agentic architectures.
  Analyzes multi-agent pipelines as formal inference chains, identifies
  enthymeme gaps (missing premises), validates context propagation, and
  generates optimized fixes — then APPLIES them directly.
  Invoke with: "chain analysis", "polysyllogism", "prompt optimization",
  "agent audit", "pipeline analysis", "chain audit", "enthymeme",
  "context starvation", or requests to fix/optimize agentic workflows,
  diagnose agent failures, audit Claude Code plugins, or analyze
  LangGraph/AutoGen/CrewAI/Agent SDK systems.
---

# Polysyllogistic Chain Optimization

*Every multi-agent system is a chain of inference. Every failure is a missing premise.*

```
╔══════════════════════════════════════════════════════════════════╗
║  This is a LIVE ANALYSIS ENGINE, not a reporting framework.     ║
║  It SCANS codebases, IDENTIFIES gaps, APPLIES fixes,           ║
║  and VERIFIES closure. The output is corrected code,            ║
║  not a document you act on later.                               ║
╚══════════════════════════════════════════════════════════════════╝
```

## Core Principle

Every multi-agent pipeline is a **polysyllogism** — a chain where each agent's output becomes the next agent's premise:

```
Agent₀: P₀ → C₀
Agent₁: C₀ + Context → C₁
Agent₂: C₁ + Context → C₂
```

Failures occur at **enthymeme gaps** — suppressed premises not propagated through state.

This holds whether state is a Python TypedDict, a JSON schema, or natural language context passed via a prompt. The substrate changes. The logic doesn't.

## Two Domains of Application

### Domain 1: Code Pipelines (Explicit State)

Traditional agentic frameworks with inspectable state:
- **LangGraph**: StateGraph, add_node, add_edge, TypedDict
- **AutoGen**: AssistantAgent, UserProxyAgent, GroupChat
- **CrewAI**: Agent, Task, Crew
- **Anthropic Agent SDK**: agent definitions, tool_use

State is typed. Transitions are edges. Gaps are missing fields.

### Domain 2: Claude Code Plugins (Implicit State)

Plugin architecture where "state" is conversation context:
- **Skills**: SKILL.md with description trigger matching
- **Agents**: .md files with frontmatter, spawned via Task tool
- **Commands**: slash commands with argument hints
- **Hooks**: event-driven (PreToolUse, PostToolUse, Stop, etc.)

State is soft-typed. Transitions are semantic matches. Gaps are insufficient context in prompts or descriptions.

| Pipeline Concept       | Code Pipeline              | Claude Code Plugin              |
|------------------------|----------------------------|---------------------------------|
| Agent node             | Python/TS function         | Agent .md / Skill SKILL.md      |
| State schema           | TypedDict / Pydantic       | Prompt context + frontmatter    |
| Transition edge        | Graph edge / function call | Description match / Task spawn  |
| Enthymeme gap          | Missing state field        | Missing context in prompt       |
| Capability manifest    | Tool registration          | `tools:` frontmatter field      |
| Validation closure     | All fields present         | Prompt carries sufficient info  |
| Provenance             | Reasoning trace dict       | Agent return value to parent    |

## Execution Phases

### Phase 1: SCAN — Topology Discovery

Use Grep and Glob to identify agent definitions, state schemas, and transitions.

**For code pipelines**: Search for framework-specific patterns.
**For Claude Code plugins**: Search for `.claude-plugin/plugin.json`, agent `.md` files, `SKILL.md` files, `hooks.json`, command `.md` files.

Output: **Topology Map** — every agent, what it reads, what it writes, what tools it has.

### Phase 2: ANALYZE — Chain Decomposition

For each A → B transition:
1. What does B REQUIRE to reason correctly?
2. What does A PROVIDE?
3. GAP = REQUIRED - PROVIDED
4. Classify gap using anti-patterns catalog

For Claude Code plugins, also check:
- Does the skill description trigger precisely? (not too broad, not too narrow)
- Does the agent prompt carry enough context for autonomous operation?
- Does the hook event match the state it needs to inspect?
- Do references load proportionally to their utility?

Output: **Gap Report** with anti-pattern classification and severity.

### Phase 3: OPTIMIZE — Fix Generation

Generate corrected artifacts:
- Amended state schemas (enforce required fields)
- Capability manifests (CAN/CANNOT lists)
- Grounding requirements injected into prompts
- Reasoning trace logging added
- Plugin frontmatter corrections

### Phase 4: DELIBERATE — Phronesis Gate

Before applying any fix, question it from the perspective of the original engineer's intent.

For each proposed fix:
1. **What does this remove or change?** Name the concrete thing being cut.
2. **Why was it there?** Reconstruct the original engineer's probable intent. Charitable interpretation, not dismissal.
3. **Does the fix preserve the intent while closing the gap?** If yes, proceed. If it closes the gap by destroying legitimate capability, it is a Procrustean Fix — redesign it.
4. **Could this be redistribution, not removal?** If a capability is on the wrong component rather than a wrong capability, move it rather than delete it.

This phase exists because OPTIMIZE has the razor but not the pause. The razor finds what to cut; DELIBERATE asks whether cutting is the right action or whether reshaping serves better.

### Phase 5: APPLY — Write Fixes

This is what makes the Claude Code version different. Use Edit/Write tools to:
- Patch state schemas directly
- Inject capability manifests into agent prompts
- Fix skill descriptions
- Add missing context to agent system prompts
- Correct hook event registrations

Present changes to user for approval before writing.

### Phase 6: VERIFY — Closure Check

Re-run Phase 2 against the corrected codebase:
- All gaps closed?
- All validation closures achieved?
- No new gaps introduced by fixes?

If gaps remain, iterate OPTIMIZE → DELIBERATE → APPLY → VERIFY.

## Quick Diagnosis

| Symptom | Anti-Pattern | Fix |
|---------|-------------|-----|
| Agent asks "which X?" when only 1 X | Phantom Disambiguation | Cardinality check |
| Agent promises screenshots/attachments | Capability Hallucination | CANNOT manifest |
| Validator can't verify claims | Context Starvation | Propagate retrieval docs |
| Agent asserts facts not in retrieval | Vacuum Reasoning | Require grounding citation |
| Critique without ground truth | Eristic Critique | Elenchus check |
| Can't trace how error was produced | Provenance Loss | Reasoning trace |
| Optional field crashes downstream | Schema Optionality Mismatch | Contract enforcement |
| Validation against own output | Circular Reference Collapse | External ground truth |
| Skill triggers on wrong inputs | Description Starvation | Precise trigger phrases |
| Subagent lacks context to operate | Agent Context Amnesia | Enrich spawn prompt |
| Hook fires but can't access state | Hook Event Mismatch | Correct event type |
| 285KB loaded when 5KB needed | Reference Overflow | Progressive disclosure |
| Command assumes unavailable context | Command Enthymeme | Explicit argument requirements |
| Fix removes capability along with problem | Procrustean Fix | Deliberate: redistribute, don't amputate |

See [references/anti-patterns.md](references/anti-patterns.md) for the full catalog.

## Key Terms

- **Polysyllogism**: Chain where each conclusion becomes next premise
- **Enthymeme**: Syllogism with suppressed premise — the gap
- **Hypokeimenon**: The substrate that persists through transitions (state object / conversation context)
- **Verification Closure**: Validator has all premises to complete proof
- **Apodeictic**: Necessarily true, cannot be otherwise — how manifests should be written
- **Elenchus**: Socratic refutation — testing claims against ground truth
- **Phronesis**: Practical wisdom — the capacity to discern right action in context, not just apply rules
- **Prolepsis**: Defensive prompting — anticipating failure modes

See [references/lexicon.md](references/lexicon.md) for the complete terminology.

## References — Load on Demand

Load ONLY the reference matching the current phase. Do NOT load all at once.

- **SCAN phase** → [references/pipeline.md](references/pipeline.md) §1 (Topology Discovery)
- **ANALYZE phase** → [references/anti-patterns.md](references/anti-patterns.md) (14 failure modes) + [references/pipeline.md](references/pipeline.md) §2
- **OPTIMIZE phase** → [references/templates.md](references/templates.md) (schemas, manifests, plugin templates)
- **DELIBERATE phase** → [references/anti-patterns.md](references/anti-patterns.md) §14 (Procrustean Fix) + [references/pipeline.md](references/pipeline.md) §4
- **APPLY phase** → [references/templates.md](references/templates.md) (schemas, manifests, plugin templates)
- **Terminology questions** → [references/lexicon.md](references/lexicon.md) (precision vocabulary)
- **Full pipeline walkthrough** → [references/pipeline.md](references/pipeline.md) (all phases)
