---
description: Analyze an agentic pipeline or Claude Code plugin for inference chain gaps
argument-hint: Path to repository or plugin directory (optional)
---

# Chain Audit

Polysyllogistic chain analysis — find where premises are missing between agents.

## Entry Point

Target: $ARGUMENTS

**If $ARGUMENTS is empty:**
Determine what to analyze. Ask the user:
1. A repository path containing an agentic pipeline (LangGraph, AutoGen, CrewAI, Agent SDK)
2. A Claude Code plugin directory to audit
3. Current working directory (scan for any agentic architecture)

**If $ARGUMENTS is provided:**
Validate the path exists. Determine whether it's a code pipeline or a Claude Code plugin by checking for `.claude-plugin/plugin.json`.

## Execution

Follow the polysyllogism skill's execution pipeline:

### Phase 1: SCAN
Launch the chain-analyzer agent via Task tool. The agent prompt MUST include:
- The **resolved absolute path** to scan (not "the target" — the actual path string)
- Whether we detected it as a code pipeline or Claude Code plugin (if known)
- Any specific concerns the user mentioned

Example spawn prompt:
> "Scan the agentic architecture at C:/Users/foo/project/. It appears to be a LangGraph pipeline.
> Produce a full topology map: every agent, state schema, transitions, and tools."

The agent will return a topology map. Present it to the user and confirm before proceeding.

### Phase 2: ANALYZE
Using the topology, decompose every transition into its logical chain:
- What does each downstream component REQUIRE?
- What does each upstream component PROVIDE?
- Where are the gaps?

Classify each gap against the 14 anti-patterns catalog. Present the gap report with severity ratings.

### Phase 3: OPTIMIZE
For each confirmed gap:
- Generate the specific fix using templates from the skill references

### Phase 4: DELIBERATE
Before presenting fixes to the user, run each through the phronesis gate:
- What does this fix remove or change?
- Why was the original thing there? (Charitable interpretation)
- Does the fix preserve intent while closing the gap?
- Is this removal or redistribution?

Flag any Procrustean Fixes (pattern #14) and redesign them as redistributions.

### Phase 5: APPLY
- Show the deliberated fixes to the user
- On approval, apply with Edit tool

### Phase 6: VERIFY
Re-run the analysis to confirm all gaps are closed. Report results.

## Output

The final output is not a document — it's a corrected codebase with a verification report showing:
- Gaps found and classified
- Fixes applied (with file:line references)
- Verification status (all closed / remaining gaps)
