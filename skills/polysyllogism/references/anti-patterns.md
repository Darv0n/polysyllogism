# Anti-Patterns Reference

Catalog of polysyllogistic chain failures with diagnostic criteria and remediation.

## Part I — Classic Patterns (Framework-Agnostic)

### 1. Phantom Disambiguation

Agent requests clarification where none is needed.

```
PREMISE (hallucinated): "Cities can have multiple locations"
PREMISE (retrieved):    "FAQ mentions {city}"
CONCLUSION:             "Ask: which location in {city}?"

ERROR: Major premise GENERATED, not RETRIEVED
       Actual cardinality = 1
```

**Diagnostic:** Agent asks "which X?" but retrieved context shows cardinality(X) = 1

**Fix:**
```python
# Retrieval metadata
{"entity_cardinality": 1, "disambiguation_required": False}

# Prompt injection
BEFORE requesting clarification:
  Check retrieval_metadata.disambiguation_required
  If False → clarification FORBIDDEN
```

---

### 2. Capability Hallucination

Agent claims abilities outside tool manifest.

```
PREMISE (hallucinated): "I have screenshot capability"
PREMISE (actual):       Tools = [search, retrieve, respond]
CONCLUSION:             "I can send you a screenshot"

ERROR: Premise contradicts manifest
```

**Diagnostic:** Agent offers action X; X not in tool manifest

**Fix:**
```markdown
## CAPABILITY MANIFEST

### CANNOT (apodeictic)
- Send images, screenshots, attachments
- Access systems outside tool list
- Remember previous sessions

When user requests forbidden capability:
  "I'm not able to [X]. I can [alternative from CAN list]."
```

---

### 3. Context Starvation

Downstream agent lacks premises for valid inference.

```
Agent_Retrieval outputs: {response, docs}
State captures:          {response}  # docs DROPPED
Agent_Validator needs:   {response, docs}

ERROR: Enthymeme gap — suppressed premise
```

**Diagnostic:** Validator makes grounding claims but lacks retrieval docs in state

**Fix:**
```python
class AgentState(TypedDict):
    retrieval_docs: Required[List[Document]]  # Not Optional

def validate_transition(prev, next):
    if 'retrieval_docs' in prev and 'retrieval_docs' not in next:
        raise EnthymemeError("Required context dropped")
```

---

### 4. Vacuum Reasoning

Agent reasons about domain without retrieval grounding.

```
PREMISE (training): "Parking garages typically have multiple levels"
PREMISE (unchecked): Retrieved FAQ silent on levels
CONCLUSION:          "The garage has multiple levels"

ERROR: Domain assertion without retrieval citation
```

**Diagnostic:** Agent makes domain claim not traceable to retrieved document

**Fix:**
```markdown
## GROUNDING REQUIREMENT

For ANY domain-specific claim:
1. Identify retrieved document supporting claim
2. If no document → claim FORBIDDEN
3. Say: "Available information doesn't specify [X]"

PATTERN:  "According to [doc_id], [claim]"
```

---

### 5. Eristic Critique

Validator objects for process, not truth.

```
RESPONSE:     "Parking is at 123 Main St"
CRITIQUE:     "Doesn't specify which entrance"
GROUND TRUTH: FAQ mentions one entrance
RESULT:       Unnecessary revision triggered

ERROR: Critique from abstract possibility, not ground
```

**Diagnostic:** Critique flags issue not reflected in retrieved ground truth

**Fix:**
```markdown
## ELENCHUS PROTOCOL

BEFORE flagging issue:
1. Identify specific ground truth document
2. Verify issue EXISTS in that document
3. If not in ground → critique FORBIDDEN

VALID:   "Response says X, but [doc_id] says Y"
INVALID: "Response doesn't address [thing not in any doc]"
```

---

### 6. Provenance Loss

Cannot trace how conclusion was derived.

```
INPUT:  User query
OUTPUT: Response with error
TRACE:  ???

ERROR: No reasoning trace
       Cannot identify failing agent
       Cannot identify wrong premise
```

**Diagnostic:** Error occurs; cannot determine which agent or premise caused it

**Fix:**
```python
class ReasoningStep(TypedDict):
    agent: str
    decision: str
    reasoning: str
    premises_used: List[str]
    grounding_docs: List[str]

# Every agent appends
state['reasoning_trace'].append({
    "agent": "critique",
    "decision": "approve",
    "grounding_docs": ["faq_parking_123"]
})
```

---

### 7. Schema Optionality Mismatch

Optional field treated as Required downstream.

```
STATE SCHEMA:   language: Optional[str]
AGENT_REVISOR:  Assumes language present
RUNTIME:        language is None
RESULT:         Crash or incorrect default

ERROR: Contract mismatch
```

**Diagnostic:** Agent accesses field without null-check; field is Optional in schema

**Fix:**
```python
# Option 1: Make required
class AgentState(TypedDict):
    language: str  # Remove Optional

# Option 2: Explicit contract
class RevisorContract:
    required = ['response', 'language', 'retrieval_docs']

def revisor_entry(state):
    for field in RevisorContract.required:
        if state.get(field) is None:
            raise ContractViolation(f"Requires '{field}'")
```

---

### 8. Circular Reference Collapse

Validation against own output. Echo chamber.

```
ITERATION 1: Agent produces R1
ITERATION 2: Agent validates R1 against R1
RESULT:      R1 "validated" by matching itself

ERROR: No external ground truth in loop
```

**Diagnostic:** Iterative loop where validation references previous output, not retrieval

**Fix:**
```markdown
## LOOP INVARIANT

In any iterative refinement:
1. Validation MUST reference EXTERNAL ground truth
2. External = retrieval docs, user query, manifest
3. Previous output is NEVER ground truth

VALID:   Refine → Check against FAQ → Iterate
INVALID: Refine → Check against previous response → Iterate
```

---

## Part II — Claude Code / LLM-Native Patterns

These patterns emerge in systems where "state" is natural language context and "transitions" are semantic matches rather than explicit edges.

### 9. Description Starvation

Skill or agent description is too vague to trigger correctly, or too broad to trigger precisely.

```
DESCRIPTION: "Helps with code"
RESULT:      Triggers on everything involving code
             Competes with 12 other skills
             User gets wrong skill half the time

ERROR: Dihairesis failure — no natural joint in trigger space
```

**Diagnostic:**
- Description uses generic terms without specific trigger phrases
- No quoted phrases for exact-match triggering
- Overlaps with other skill/agent descriptions in same plugin set

**Detection in codebase:**
- Read SKILL.md or agent .md frontmatter `description:` field
- Check for: quoted trigger phrases, specific keywords, bounded scope
- Cross-reference against other installed skill descriptions for collision

**Fix:**
```yaml
# BAD — starved
description: Helps analyze code quality

# GOOD — nourished
description: >-
  Analyzes multi-agent pipelines as formal inference chains.
  Invoke with: "chain analysis", "polysyllogism", "agent audit",
  "pipeline analysis", or requests to diagnose agent failures.
```

Principle: Description is a **contract**, not a summary. It defines the trigger boundary. Ambiguous boundary = phantom disambiguation at the dispatch level.

---

### 10. Agent Context Amnesia

Subagent spawned via Task tool without sufficient context in the prompt to operate autonomously.

```
PARENT CONTEXT: Full conversation history, file reads, user intent
SPAWN PROMPT:   "Analyze the agent architecture"
SUBAGENT SEES:  Just that one line

ERROR: Enthymeme gap at spawn boundary
       Subagent has conclusion request but not premises
```

**Diagnostic:**
- Task tool `prompt` parameter is short/vague
- Prompt references "the file" / "the error" / "above" without specifics
- Subagent would need context it doesn't have

**Detection in codebase:**
- Search for Task tool invocations in command .md files
- Check if prompts carry sufficient standalone context
- Look for deictic references ("this", "that", "the above") without antecedents

**Fix:**
```markdown
# BAD — amnesiac
prompt: "Review the agent for issues"

# GOOD — context-complete
prompt: >-
  Review the agent defined in /path/to/agent.md.
  Check that its description field contains specific trigger phrases,
  its tools list matches its stated capabilities, and its system prompt
  provides enough context for autonomous operation.
  Return: list of gaps found with severity and fix suggestions.
```

Principle: Every subagent spawn is a **transition**. The prompt IS the state passed across the boundary. If the prompt is an enthymeme, the subagent will hallucinate the missing premises.

---

### 11. Hook Event Mismatch

Hook registered for an event that doesn't carry the state the hook needs to inspect.

```
HOOK EVENT:     PreToolUse
HOOK CHECKS:    Whether output contains sensitive data
PROBLEM:        PreToolUse fires BEFORE execution — no output exists yet

ERROR: Schema Optionality Mismatch across event boundary
       Hook reads field that doesn't exist at this event stage
```

**Diagnostic:**
- Hook registered on PreToolUse but needs PostToolUse state
- Hook registered on Stop but needs per-tool granularity
- Hook checks `tool_output` in a `PreToolUse` context

**Detection in codebase:**
- Read hooks.json for event registrations
- For each hook, identify what state it inspects
- Verify that state is available at the registered event stage

**Fix:**
```json
{
  "hooks": {
    "PostToolUse": [{
      "description": "Check output for sensitive data",
      "command": "check-sensitive.sh"
    }]
  }
}
```

Principle: Events are **temporal boundaries**. A hook that reads future state is a paralogism — formally structured but materially impossible.

---

### 12. Reference Overflow

Skill loads disproportionate reference material relative to what's needed for the task.

```
SKILL.md:        5KB of framework
REFERENCES/:     280KB across 7 files
TASK:            Needs 1 reference for this invocation
LOADED:          All 280KB into context

ERROR: Token budget consumed by irrelevant premises
       Actual reasoning squeezed into remaining context
       Quality degrades from noise, not absence
```

**Diagnostic:**
- Total reference material > 50KB
- References don't have clear conditional loading instructions
- SKILL.md doesn't specify WHEN to load WHICH reference
- All references loaded on every invocation

**Detection in codebase:**
- Measure total size of skill directory
- Check SKILL.md for loading directives ("load X when Y")
- Check if references have clear, distinct scopes

**Fix:**
```markdown
## References — Load on Demand

Load ONLY the reference matching the current phase:

- SCAN phase → [references/pipeline.md] §1 only
- ANALYZE phase → [references/anti-patterns.md]
- OPTIMIZE phase → [references/templates.md]
- Terminology questions → [references/lexicon.md]

Do NOT load all references at once.
```

Principle: In token-bounded systems, **every premise has a cost**. Flooding context with unused premises is the inverse of starvation — death by abundance. The hypokeimenon (context window) has finite capacity. Respect it.

---

### 13. Command Enthymeme

Slash command prompt assumes context that won't be present when invoked.

```
COMMAND .md:     "Analyze the pipeline discussed above"
INVOCATION:      User types /chain-audit in fresh session
CONTEXT:         No pipeline has been discussed

ERROR: Command premise references non-existent prior context
       "Above" points to nothing
```

**Diagnostic:**
- Command references $ARGUMENTS but doesn't handle empty case
- Command assumes prior conversation context
- Command uses deictic references without fallback

**Detection in codebase:**
- Read command .md files
- Check if $ARGUMENTS is validated or has defaults
- Check for assumptions about conversation history

**Fix:**
```markdown
## Entry Point

Target: $ARGUMENTS

If $ARGUMENTS is empty:
  Ask user: "What would you like to analyze?"
  Options:
  1. A repository path (for code pipeline analysis)
  2. A plugin directory (for Claude Code plugin audit)
  3. Current project (scan working directory)

If $ARGUMENTS is provided:
  Validate path exists before proceeding.
```

Principle: A command is an **entry point**, not a continuation. It must be self-sufficient. Any premise it needs must come from its arguments or be gathered at runtime.

---

## Diagnostic Checklist

For any observed failure, check systematically:

### Classic Chain Patterns
```
□ Phantom Disambiguation — asking clarification when cardinality=1?
□ Capability Hallucination — promising action outside manifest?
□ Context Starvation — downstream missing upstream context?
□ Vacuum Reasoning — domain assertion without retrieval citation?
□ Eristic Critique — objecting without ground truth basis?
□ Provenance Loss — can we trace how error was produced?
□ Schema Mismatch — Optional treated as Required?
□ Circular Collapse — validating against own output?
```

### Claude Code / LLM-Native Patterns
```
□ Description Starvation — trigger boundary ambiguous?
□ Agent Context Amnesia — subagent prompt missing premises?
□ Hook Event Mismatch — hook reads state unavailable at event?
□ Reference Overflow — loading 100KB when 5KB needed?
□ Command Enthymeme — command assumes absent context?
```

Each checked box points to a specific remediation pattern above.
