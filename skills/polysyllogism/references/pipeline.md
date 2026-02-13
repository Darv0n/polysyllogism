# Execution Pipeline

Step-by-step protocol for analyzing and optimizing agentic architectures.
Designed for Claude Code: uses native tools, applies fixes directly, verifies closure.

## Phase 1: SCAN — Topology Discovery

### 1.1 Detect Framework

Determine what kind of agentic system we're analyzing.

**Code Pipeline Detection** — use Grep tool:

```
Pattern: "StateGraph|add_node|add_edge|CompiledGraph"
Type: py
→ LangGraph detected

Pattern: "AssistantAgent|UserProxyAgent|GroupChat|ConversableAgent"
Type: py
→ AutoGen detected

Pattern: "class.*Agent.*:|Task\(|Crew\("
Type: py
→ CrewAI detected

Pattern: "from anthropic|client\.messages|tool_use"
Type: py, ts
→ Anthropic Agent SDK detected
```

**Claude Code Plugin Detection** — use Glob tool:

```
Pattern: "**/.claude-plugin/plugin.json"
→ Plugin manifest found

Pattern: "**/agents/*.md"
→ Agent definitions found

Pattern: "**/skills/*/SKILL.md"
→ Skill definitions found

Pattern: "**/commands/*.md"
→ Command definitions found

Pattern: "**/hooks/hooks.json"
→ Hook definitions found
```

### 1.2 Extract Agent Definitions

**For code pipelines** — use Grep tool:

```
Pattern: "def\s+\w+.*state.*:"    # Function-based agents
Pattern: "class\s+\w+Agent"        # Class-based agents
Pattern: "@tool|def.*_tool"        # Tool definitions
```

Read each matched file to extract:
- Agent name and role
- What state keys it reads
- What state keys it writes
- What tools it has access to

**For Claude Code plugins** — use Read tool:

For each agent .md:
- Parse frontmatter: name, description, tools, model, color
- Parse system prompt: what does it expect? what does it produce?
- Note: `tools:` field IS the capability manifest

For each SKILL.md:
- Parse frontmatter: name, description
- Identify trigger phrases in description
- Assess reference loading strategy

For each command .md:
- Parse frontmatter: description, argument-hint
- Check $ARGUMENTS handling
- Identify assumed context

### 1.3 Extract State Schema

**For code pipelines** — use Grep tool:

```
Pattern: "class\s+\w+.*TypedDict|class\s+\w+.*BaseModel"
Type: py
→ Read matched files for field definitions

Pattern: "state\[.+\]|state\.get\(|state\.\w+"
Type: py
→ Map actual state access patterns vs. declared schema
```

**For Claude Code plugins:**

State is implicit. The "schema" is:
- What the skill description promises to provide
- What the agent prompt assumes is available
- What the command $ARGUMENTS contains
- What the hook event payload includes

### 1.4 Output: Topology Map

```
┌─────────────────────────────────────────────────────────┐
│  TOPOLOGY: [System Name]                                │
│  Framework: [LangGraph | Plugin | etc.]                  │
├─────────────────────────────────────────────────────────┤
│  AGENTS/COMPONENTS:                                      │
│  1. [Name] — Role: [role]                                │
│     Reads:  [state keys / context requirements]          │
│     Writes: [state keys / outputs]                       │
│     Tools:  [list]                                       │
│                                                          │
│  STATE:                                                  │
│  - [field]: [type] (required/optional)                   │
│                                                          │
│  FLOW:                                                   │
│  [Entry] → Component_1 → Component_2 → [Exit]            │
│                                                          │
│  TRANSITIONS TO ANALYZE:                                 │
│  T1: Component_1 → Component_2                            │
│  T2: Component_2 → Component_3                            │
└─────────────────────────────────────────────────────────┘
```

**CHECKPOINT: Present topology to user before proceeding.**

---

## Phase 2: ANALYZE — Chain Decomposition

### 2.1 Model Each Transition

For every A → B transition identified in the topology:

```
TRANSITION: [A] → [B]
══════════════════════════════════════════════════════════

A PROVIDES: [list of state keys / context / outputs]
B REQUIRES: [list of what B needs to reason correctly]

MATCH:
  □ [requirement_1] — source: [key/context] ✓
  □ [requirement_2] — source: [key/context] ✗ MISSING

GAP: [REQUIRED] - [PROVIDED]
```

### 2.2 Classify Gaps

For each gap found, classify against the anti-patterns catalog:

```
GAP #N:
  Location:    [A] → [B]
  Missing:     [what's not propagated]
  Pattern:     [anti-pattern name from catalog]
  Severity:    CRITICAL | HIGH | MEDIUM | LOW
  Impact:      [what goes wrong]
```

Severity guide:
- **CRITICAL**: System produces wrong outputs silently
- **HIGH**: System crashes or fails visibly
- **MEDIUM**: System works but with degraded quality
- **LOW**: Cosmetic or edge-case issue

### 2.3 Check Validation Closure

For any component that validates, critiques, or checks other components' work:

```
VALIDATOR: [Name]
─────────────────
Checks: [what it validates]

  [Check_1]: needs [premises]
             has   [what's available]
             CLOSED ✓ | OPEN ✗

  [Check_2]: needs [premises]
             has   [what's available]
             CLOSED ✓ | OPEN ✗ — missing [what]
```

### 2.4 Output: Gap Report

```
┌─────────────────────────────────────────────────────────┐
│  GAP REPORT                                              │
├─────────────────────────────────────────────────────────┤
│  CRITICAL:                                               │
│  1. [description] — Pattern: [anti-pattern]              │
│  2. [description] — Pattern: [anti-pattern]              │
│                                                          │
│  HIGH:                                                   │
│  3. [description] — Pattern: [anti-pattern]              │
│                                                          │
│  VALIDATION STATUS:                                      │
│  □ [Validator_1]: CLOSED                                 │
│  □ [Validator_2]: OPEN — needs [what]                    │
│                                                          │
│  MANIFEST STATUS:                                        │
│  □ [Agent_1]: Complete                                   │
│  □ [Agent_2]: Missing — RISK                             │
└─────────────────────────────────────────────────────────┘
```

**CHECKPOINT: Present gap report to user. Confirm which gaps to fix.**

---

## Phase 3: OPTIMIZE — Fix Generation

For each confirmed gap, generate the specific fix:

### 3.1 State Schema Fixes

For code pipelines: corrected TypedDict/Pydantic with required fields enforced.
For plugins: corrected frontmatter or enriched prompt context.

### 3.2 Capability Manifests

For each agent/component, generate CAN/CANNOT list.
See [templates.md](templates.md) for format.

### 3.3 Prompt Corrections

For code pipelines: system prompt injections (grounding, disambiguation, elenchus).
For plugins: rewritten description fields, enriched agent system prompts.

### 3.4 Structural Fixes

- Hook event corrections (move to correct event stage)
- Reference loading directives (progressive disclosure)
- Command argument validation (handle empty $ARGUMENTS)

---

## Phase 4: APPLY — Write Fixes

Use Edit tool for surgical changes. Use Write tool only for new files.

**Protocol:**
1. Show user the proposed change (old → new)
2. On approval, apply with Edit tool
3. Log each applied change for Phase 5 verification

**For code pipelines:**
- Edit state schema files
- Edit agent function files to add contract checks
- Write capability manifest files
- Edit system prompt strings

**For plugins:**
- Edit SKILL.md frontmatter descriptions
- Edit agent .md frontmatter and system prompts
- Edit hooks.json event registrations
- Edit command .md argument handling
- Add loading directives to SKILL.md reference sections

---

## Phase 5: VERIFY — Closure Check

### 5.1 Re-run Analysis

Execute Phase 2 again against the modified codebase:
- Are all previously identified gaps now closed?
- Did any fixes introduce new gaps?
- Are all validation closures achieved?

### 5.2 Cross-Reference

For Claude Code plugins specifically:
- Do skill descriptions still trigger correctly after edits?
- Do agent tool lists match their actual capabilities?
- Do hook events match the state they inspect?
- Is reference loading proportional?

### 5.3 Report

```
┌─────────────────────────────────────────────────────────┐
│  VERIFICATION REPORT                                     │
├─────────────────────────────────────────────────────────┤
│  GAPS CLOSED:     [N] / [total]                          │
│  NEW GAPS:        [N] (if any)                           │
│  VALIDATION:      [ALL CLOSED | X OPEN]                  │
│                                                          │
│  CHANGES APPLIED:                                        │
│  1. [file:line] — [what changed]                         │
│  2. [file:line] — [what changed]                         │
│                                                          │
│  STATUS: [COMPLETE | NEEDS ITERATION]                    │
└─────────────────────────────────────────────────────────┘
```

If NEEDS ITERATION → return to Phase 3 with remaining gaps.
