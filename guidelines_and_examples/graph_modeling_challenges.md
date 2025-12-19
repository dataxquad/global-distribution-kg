# Graph Modeling Challenges & Current Decisions

**Context: LLM-Driven Global Distribution Graph**
**Purpose: Preparation for Neo4j Discussion (Jan 8)**

---

## 1. Challenge: Storing Complex, Unstructured Business Information

### Problem

* Business knowledge comes from emails, meetings, news, decks, chats
* Information is:
  * Messy
  * Partially true
  * Time-dependent
  * Often interpretive
* Risk: polluting the graph with assumptions and premature conclusions

### Key Decision

* **Store facts, not insights**
* Treat the graph as a **source of truth**, not a reasoning engine

### Current Approach

* Graph stores:
  * Explicit entities
  * Verb-based relationships
  * Time-bound events
* Insights (roles, importance, opportunity quality):
  * Derived at query time
  * Promoted into schema only if they are:
    * Repeated
    * Stable
    * Business-critical

### Open Question for Neo4j

* How do mature graph systems prevent “insight leakage” into core schema at scale?

---

## 2. Challenge: Representing Uncertainty (Opportunities, Plans, Pipeline)

### Problem

* Sales and business development are inherently uncertain
* We still need to:
  * Track opportunities
  * Explain why we believe they exist
  * Query pipeline status

### Key Decision

* **Separate uncertainty from facts**
* Never encode uncertainty as Company–Company relationships

### Current Approach

* Introduce **Intent entities** (e.g., `Opportunity`, `Goal`)
* Ground intent using **Events**
* Example:
  * Opportunity exists → because meetings happened
  * Progress is shown → via new events, not property mutation

### Principle

* Intent is *real*, but not *reality*
* Intent does not assert outcomes, roles, or commitments

### Open Question for Neo4j

* Is modeling intent as first-class nodes (instead of states) a common enterprise pattern?

---

## 3. Challenge: Tracking Progress Without Corrupting Truth

### Problem

* Business states evolve:
  * Negotiating → Delayed → Approved → Cancelled → Revived
* Risk:
  * Overwriting properties loses history
  * State labels encode interpretation

### Key Decision

* **All progress is expressed as Events**
* Nodes represent existence, not lifecycle

### Current Approach

* Use a single, powerful `Event` node:
  * Meetings
  * Decisions
  * Delays
  * Approvals
* No separate `Progress` or `Status` nodes
* Status is reconstructed by traversing event history

### Principle

* Reality advances by **adding facts**, not mutating state

### Open Question for Neo4j

* Performance and query patterns for event-sourced graphs at scale?

---

## 4. Challenge: Two Dimensions of “Uncertainty”

### Dimension 1: Uncertain Relationship

**Example**
* “Company A plans to manufacture Product X”

**Decision**
* ❌ Do NOT create the relationship
* Only create `(Company)-[:MANUFACTURES]->(Product)` when it actually happens

---

### Dimension 2: Certain Concept, Uncertain Outcome

**Example**
* “There is an opportunity with Company A, but outcome is unknown”

**Decision**
* ✅ Create the `Opportunity` node
* ❌ Do NOT create customer/vendor relationships
* Use Events to track:
  * Meetings
  * Budget approval
  * Delays
  * Rejection or success

### Rule

* **Entities may exist before outcomes**
* **Relationships require facts**

### Open Question for Neo4j

* How do large graph systems balance early entity creation vs late relationship creation?

---

## 5. Challenge: LLM Misinterpretation Risk

### Problem

* LLMs tend to:
  * Assign roles
  * Infer intent
  * Collapse time

### Key Decision

* Enforce **schema-level guardrails**, not prompt-level discipline

### Current Approach

* Explicit extraction rules:
  * LLMs extract what is said, not what is implied
  * No role assignment
  * No probability or judgment
* Schema descriptions carry semantic constraints

### Open Question for Neo4j

* Best practices for schema-as-contract when LLMs are upstream producers?

---

## 6. What We Want Feedback On

* Is our **Fact / Intent / Event / Commitment** separation aligned with Neo4j best practices?
* Are there anti-patterns we should watch for early?
* How do other teams:
  * Audit graph correctness?
  * Prevent schema drift?
  * Scale event-heavy graphs?

---

## Summary

* We prioritize **correctness over convenience**
* We accept ambiguity, but **do not encode it as fact**
* We treat the graph as:
  * Auditable
  * Explainable
  * Inference-ready, not inference-polluted

**Goal of the discussion:**
Validate whether this discipline scales — and where Neo4j’s experience suggests refinement.
