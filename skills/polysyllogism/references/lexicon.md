# Lexicon — High-Gravity Terminology

Precision vocabulary for chain analysis. Coordinates, not metaphors.

## Contents

1. [Core Logical Terms](#core-logical-terms)
2. [Pipeline-Specific Terms](#pipeline-specific-terms)
3. [Compound Terms](#compound-terms)

---

## Core Logical Terms

### POLYSYLLOGISM
Chain of syllogisms where each conclusion becomes next premise. The fundamental structure of multi-agent pipelines.

### ENTHYMEME
Syllogism with suppressed premise. The hidden assumption that makes argument work—or fail silently.

```
STATED:    "FAQ mentions parking" → "Tell user about parking"
UNSTATED:  "FAQ is authoritative source"
UNSTATED:  "User query matches FAQ topic"
```
In pipelines: Context not propagated = enthymeme gap.

### SORITES
"Heap paradox" — chain where small valid steps accumulate into invalid conclusions. Each agent makes reasonable assumptions; accumulated assumptions produce nonsense.

### PARALOGISM
Formally valid, materially false. Syllogism looks correct but produces wrong conclusions because a premise is false. Kant's term for reason deceiving itself.

### ERISTIC
Argument for victory rather than truth. Critique agent that objects to validate process, not improve accuracy.

### APODEICTIC
Necessarily true. Cannot be otherwise. The hardest constraints.

```
APODEICTIC: "This agent cannot send screenshots" (architectural fact)
ASSERTORIC: "This agent should not send screenshots" (negotiable)
```
Capability manifests should be apodeictic.

### ELENCHUS
Socratic refutation. Testing claims against ground truth until contradiction emerges. The method critique agents should use.

### PROLEPSIS
Anticipation of objection. Addressing counterarguments before they arise.

```
WITHOUT: "Generate a helpful response"
WITH:    "Generate response. Do NOT ask clarification if cardinality=1."
```
Defensive prompting.

### DIHAIRESIS
Platonic division. Cutting concept at natural joints.

```
VALID:   "Cities" → "with parking" / "without parking"
INVALID: "Cities" → "large" / "small" (arbitrary)
```
Phantom disambiguation = dihairesis where no joint exists.

---

## Pipeline-Specific Terms

### HYPOKEIMENON
Aristotle's "underlying" — what persists through change. The substrate.

In pipelines: The state object. What survives every agent transition.

```python
class AgentState(TypedDict):  # This IS the hypokeimenon
    query: str                 # persists
    retrieval_docs: List[Doc]  # persists
    response: str              # transforms
```

### CONTEXT CARDINALITY
What MUST persist vs. what CAN drop.

```
CARDINAL:     query, retrieval_docs, language
NON-CARDINAL: intermediate_thoughts, timing_metadata
```

### VERIFICATION CLOSURE
Validator has all premises needed to complete proof. No enthymeme gaps.

```
OPEN:   Validator has {response} but needs {response, docs, query}
CLOSED: Validator has {response, docs, query}
```

### PROVENANCE THREADING
Each output carries its derivation chain. Full trace from input to output.

### GROUNDED INFERENCE
Syllogism where middle term is RETRIEVED, not GENERATED.

```
UNGROUNDED: "Cities typically have multiple locations" (training)
GROUNDED:   "FAQ shows {city} has 1 location" (retrieval)
```

### CAPABILITY MANIFEST
Apodeictic list of what agent CAN and CANNOT do. Architectural facts.

### STATE SCHEMA INVARIANT
Contract that survives all transitions. What must be true at every point.

---

## Compound Terms

### ENTHYMEME GAP
The specific missing premise in a chain.

```
GAP: Agent_Retriever → Agent_Validator
     Missing: retrieval_metadata
     Impact: Validator cannot check cardinality
```

### ERISTIC DRIFT
When critique agents shift from truth-seeking to process-validating.

### PHANTOM DISAMBIGUATION
Dihairesis where no joint exists. "Which X?" when cardinality(X) = 1.

### VACUUM REASONING
Domain assertion without retrieval grounding.

### CIRCULAR REFERENCE COLLAPSE
Validation against own output. Echo chamber in loops.

---

## Usage Protocol

1. **Name failure precisely** — use lexicon term
2. **Identify logical structure** — what syllogism failed?
3. **Locate in chain** — which transition introduced gap?
4. **Apply remediation** — from anti-patterns reference

The vocabulary is diagnostic. Each term points to a specific fix.
