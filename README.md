# Polysyllogism

Chain analysis engine for multi-agent architectures. Treats every agentic pipeline as a formal inference chain, finds the missing premises, and fixes them.

## What It Does

Polysyllogism analyzes multi-agent systems by decomposing them into chains of syllogisms. Where one agent's output feeds another agent's input, it checks whether the downstream agent has everything it needs to reason correctly. Gaps between agents -- missing state fields, insufficient prompt context, mismatched event hooks -- are classified against a catalog of 14 named anti-patterns and then fixed directly in the codebase.

The output is corrected code, not a report.

## Installation

Polysyllogism is a Claude Code plugin. Load it with the `--plugin-dir` flag:

```
claude --plugin-dir /path/to/polysyllogism
```

Or add it to your Claude Code configuration for persistent loading.

## Usage

### The `/chain-audit` Command

The primary entry point. Invoke it from within Claude Code:

```
/chain-audit /path/to/project
```

If no path is provided, the command will ask what you want to analyze.

The command runs a six-phase pipeline:

1. **SCAN** -- Spawns the `chain-analyzer` agent to discover the topology (agents, state schemas, transitions, tools).
2. **ANALYZE** -- Decomposes every transition into its logical chain and classifies gaps against the 14 anti-patterns.
3. **OPTIMIZE** -- Generates fixes using reference templates.
4. **DELIBERATE** -- Phronesis gate. Questions each fix from the original engineer's perspective before applying. Catches Procrustean Fixes (pattern #14) where the optimizer removes legitimate capability along with the problem.
5. **APPLY** -- Writes fixes with user approval.
6. **VERIFY** -- Re-runs analysis to confirm all gaps are closed.

### Skill Trigger Phrases

The polysyllogism skill activates on natural language requests containing:

- "chain analysis"
- "polysyllogism"
- "prompt optimization"
- "agent audit"
- "pipeline analysis"
- "chain audit"
- "enthymeme"
- "context starvation"

It also triggers on requests to fix or optimize agentic workflows, diagnose agent failures, audit Claude Code plugins, or analyze LangGraph/AutoGen/CrewAI/Agent SDK systems.

### The `chain-analyzer` Agent

A read-only autonomous scanner spawned during the SCAN phase. It uses Glob, Grep, Read, and Bash to discover the full topology of an agentic system. It has no write access. It detects the framework type, extracts agent definitions, maps state schemas, identifies transitions, and returns a structured topology map.

## Anti-Patterns Quick Reference

| # | Name | Description |
|---|------|-------------|
| 1 | Phantom Disambiguation | Agent requests clarification where none is needed (cardinality = 1) |
| 2 | Capability Hallucination | Agent claims abilities outside its tool manifest |
| 3 | Context Starvation | Downstream agent lacks premises that upstream produced but state dropped |
| 4 | Vacuum Reasoning | Agent makes domain assertions without retrieval grounding |
| 5 | Eristic Critique | Validator objects based on abstract possibility, not ground truth |
| 6 | Provenance Loss | Cannot trace which agent or premise caused an error |
| 7 | Schema Optionality Mismatch | Optional field in schema treated as required by downstream agent |
| 8 | Circular Reference Collapse | Validation against own output rather than external ground truth |
| 9 | Description Starvation | Skill or agent description too vague or too broad to trigger correctly |
| 10 | Agent Context Amnesia | Subagent spawned via Task tool without sufficient context to operate |
| 11 | Hook Event Mismatch | Hook registered for an event that does not carry the state it needs |
| 12 | Reference Overflow | Skill loads disproportionate reference material relative to task needs |
| 13 | Command Enthymeme | Slash command assumes context that will not be present at invocation |
| 14 | Procrustean Fix | Optimizer closes gap by destroying capability the system legitimately needs |

Patterns 1-8 are framework-agnostic (classic chain failures). Patterns 9-14 are specific to Claude Code and LLM-native architectures.

## Architecture

```
polysyllogism/
  .claude-plugin/
    plugin.json              # Plugin manifest (name, description, author)
  commands/
    chain-audit.md           # /chain-audit slash command -- primary entry point
  agents/
    chain-analyzer.md        # Read-only topology scanner (spawned by chain-audit)
  skills/
    polysyllogism/
      SKILL.md               # Skill definition -- core framework and execution phases
      references/
        anti-patterns.md     # 14 named failure modes with diagnostics and fixes
        pipeline.md          # Step-by-step execution protocol for all phases
        templates.md         # Reusable schemas, manifests, and prompt injections
        lexicon.md           # Precision vocabulary (Aristotelian terminology)
```

### How the Components Relate

1. The **command** (`/chain-audit`) is the entry point. It accepts a path argument or prompts the user for one.
2. The command invokes the **skill** (polysyllogism), which defines the six-phase execution pipeline and loads references on demand.
3. During the SCAN phase, the command spawns the **agent** (`chain-analyzer`) via the Task tool to perform autonomous topology discovery.
4. The skill's **references** are loaded progressively by phase -- anti-patterns during ANALYZE, templates during OPTIMIZE, lexicon only when terminology precision is needed.

## Supported Domains

### Domain 1: Code Pipelines (Explicit State)

Traditional agentic frameworks where state is typed and transitions are graph edges:

- **LangGraph** -- StateGraph, add_node, add_edge, TypedDict
- **AutoGen** -- AssistantAgent, UserProxyAgent, GroupChat
- **CrewAI** -- Agent, Task, Crew
- **Anthropic Agent SDK** -- agent definitions, tool_use

### Domain 2: Claude Code Plugins (Implicit State)

Plugin architectures where state is conversation context and transitions are semantic matches:

- **Skills** -- SKILL.md with description-based trigger matching
- **Agents** -- Markdown files with frontmatter, spawned via Task tool
- **Commands** -- Slash commands with argument hints
- **Hooks** -- Event-driven (PreToolUse, PostToolUse, Stop, etc.)

## License

MIT License. See [LICENSE](LICENSE).

## Credits

Created by TrashWizard / Darv0n.
