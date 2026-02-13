# Templates

Reusable schemas, manifests, and prompt injections for both code pipelines and Claude Code plugins.

## Part I — Code Pipeline Templates

### State Schema Template

```python
from typing import TypedDict, List, Optional, Required

class RetrievalMetadata(TypedDict):
    """Metadata for downstream validation"""
    source_doc_ids: List[str]
    query_entity: str
    entity_cardinality: int
    disambiguation_required: bool
    answer_completeness: str  # 'full' | 'partial' | 'none'

class ReasoningStep(TypedDict):
    """Provenance trace entry"""
    agent: str
    timestamp: str
    decision: str
    reasoning: str
    premises_used: List[str]
    grounding_docs: List[str]

class AgentState(TypedDict):
    # === CARDINAL (must persist through all transitions) ===
    query: str
    retrieval_docs: List[dict]
    retrieval_metadata: RetrievalMetadata
    detected_language: str

    # === DERIVED (produced by agents) ===
    response: str
    validation_result: dict

    # === PROVENANCE (append-only) ===
    reasoning_trace: List[ReasoningStep]
```

### Capability Manifest Template

```markdown
## CAPABILITY MANIFEST — [Agent Name]

### IDENTITY
Role: [gatekeeper/retriever/generator/validator]
Purpose: [one sentence]

### CAN (apodeictic)
- [Action 1 with scope]
- [Action 2 with scope]
- [Tool 1]: [what it does]

### CANNOT (apodeictic)
- Send images, screenshots, or attachments
- Access systems not in CAN list
- Assert domain facts not in retrieval context
- Remember previous sessions
- [Pipeline-specific constraint]

### GROUNDING REQUIREMENTS
Before [common action]:
- Verify [check 1]
- Verify [check 2]
- If fails → [alternative]

### STATE CONTRACT
Reads:  [keys consumed]
Writes: [keys produced]
```

### Prompt Injection Templates

**Capability Boundaries:**
```markdown
## CAPABILITY BOUNDARIES

You CANNOT:
- Send images, screenshots, or attachments
- Access external systems not listed in your tools
- [Pipeline-specific forbidden action]

When user requests forbidden capability:
  Respond: "I'm not able to [X]. I can [alternative]."

Do NOT:
- Promise future capability
- Suggest workarounds using forbidden actions
```

**Grounding Requirements:**
```markdown
## GROUNDING REQUIREMENTS

For ANY domain-specific claim:
1. Identify retrieved document supporting claim
2. If no document supports → claim FORBIDDEN
3. Say: "Available information doesn't specify [X]"

PATTERN:  "According to [doc_id], [claim]"
FORBIDDEN: "[claim]" without citation
```

**Disambiguation Protocol:**
```markdown
## DISAMBIGUATION PROTOCOL

BEFORE requesting clarification:
1. Check retrieval_metadata.disambiguation_required
2. If FALSE → clarification FORBIDDEN
3. If TRUE → clarification REQUIRED

BEFORE asking "which X?":
1. Check retrieval_metadata.entity_cardinality
2. If cardinality = 1 → question FORBIDDEN
3. Proceed with single-option response
```

**Elenchus Protocol (for validators):**
```markdown
## ELENCHUS PROTOCOL

BEFORE flagging an issue:
1. Identify specific ground truth document
2. Verify issue EXISTS in that document
3. If not in ground → critique FORBIDDEN

VALID critique:   "Response says X, but [doc_id] says Y"
INVALID critique: "Response doesn't address [thing not in any doc]"

Principle: Critique against RETRIEVED TRUTH only.
```

**Reasoning Trace:**
```python
trace_entry = {
    "agent": "validator",
    "decision": "approve",
    "reasoning": "Response matches FAQ content",
    "premises_used": ["response", "retrieval_docs", "query"],
    "grounding_docs": ["faq_parking_main_st"],
    "capability_check": None
}
# Append to state['reasoning_trace'] before returning
```

---

## Part II — Claude Code Plugin Templates

### Agent Definition Template

```markdown
---
name: [agent-name]
description: [Precise description of what this agent does and when it should be spawned. Include specific capability boundaries.]
tools: [Comma-separated list matching ACTUAL capabilities. Do not list tools the agent doesn't need.]
model: [sonnet | opus | haiku — match to task complexity]
color: [green | blue | yellow | red]
---

You are a [role] that [specific purpose].

## What You Receive

You will be given [specific inputs]. These are your premises. Do not assume information beyond what is provided.

## What You Produce

Return [specific outputs]. Every claim must trace to your inputs.

## Constraints

- You have access to: [tools from frontmatter — restate for clarity]
- You do NOT have access to: [explicitly state what's excluded]
- If you need information not in your inputs, say so — do not fabricate

## Process

1. [Step with specific action]
2. [Step with specific action]
3. [Step with specific action]
```

**Anti-pattern prevention built in:**
- "Do not assume information beyond what is provided" → prevents Vacuum Reasoning
- Explicit tool restatement → prevents Capability Hallucination
- "If you need information not in your inputs, say so" → surfaces Context Starvation

### Skill SKILL.md Template

```markdown
---
name: [skill-name]
description: >-
  [What the skill does — one sentence]. [When to invoke — specific phrases].
  Invoke with: "[phrase 1]", "[phrase 2]", "[phrase 3]", or requests to
  [action 1], [action 2], or [action 3].
---

# [Skill Title]

[One-line purpose statement]

## When This Applies

[Precise activation conditions — not vague categories]

## Core Process

[The actual methodology]

## References — Load on Demand

Load ONLY the reference matching the current need:

- [Phase/need 1] → [references/file1.md]
- [Phase/need 2] → [references/file2.md]
- Terminology → [references/lexicon.md]

Do NOT load all references at once.
```

**Anti-pattern prevention built in:**
- Quoted trigger phrases → prevents Description Starvation
- "Load on Demand" → prevents Reference Overflow
- Precise activation conditions → prevents phantom triggering

### Command Template

```markdown
---
description: [What the command does]
argument-hint: [What to pass as argument]
---

# [Command Name]

## Entry Point

Target: $ARGUMENTS

**If $ARGUMENTS is empty:**
Ask the user what they want to [action]. Provide specific options:
1. [Option with description]
2. [Option with description]
3. [Option with description]

**If $ARGUMENTS is provided:**
Validate that the target exists and is appropriate before proceeding.

## Execution

[Steps that don't assume prior context]
```

**Anti-pattern prevention built in:**
- Empty $ARGUMENTS handling → prevents Command Enthymeme
- Validation step → prevents runtime failures from bad input
- No assumptions about prior context → self-sufficient entry point

### Hook Template

```json
{
  "hooks": {
    "[Event — choose correctly]": [
      {
        "description": "[What this hook checks — be specific]",
        "command": "[script or command]",
        "matcher": {
          "tool_name": "[specific tool, not wildcard]"
        }
      }
    ]
  }
}
```

**Event selection guide:**
| You want to... | Use event... | NOT... |
|---|---|---|
| Block a tool call before it runs | PreToolUse | PostToolUse |
| Check tool output after execution | PostToolUse | PreToolUse |
| Validate final response | Stop | PostToolUse |
| Check user input before processing | UserPromptSubmit | PreToolUse |
| Run setup at session start | SessionStart | UserPromptSubmit |

**Anti-pattern prevention built in:**
- Event selection guide → prevents Hook Event Mismatch
- Specific matcher → prevents over-broad hook firing
