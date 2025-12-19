# LLM-Driven Graph Schema Principles (Fact-First Design)

## Purpose

This document defines schema and ontology principles for building a graph database for the **Global Distribution Business**, populated by LLM extraction from unstructured text.

The goal is to store **universal facts** that every business in the world can use as a foundation, while leaving room for specific use cases in future versions.

This approach ensures:
- Reduced LLM reasoning cost
- Avoidance of premature business interpretation
- Preservation of factual correctness
- Consistent, reusable inference later

## Core Principle: Facts First, Insights Later

### Definition

- **Facts**: Explicitly stated, observable relationships in source data
- **Insights / Inferences**: Conclusions derived by reasoning over facts

> ➡ **Only facts belong in the graph at this stage.**
> ➡ **Insights must NOT be stored as facts unless they become stable, reusable, and agreed upon.**

### What Is a Fact?

A fact is something that appears directly and unambiguously in the source text.

**Examples (Facts — STORE THESE)**
- Company A produces Product X
- Company A sells Product X to Company B
- Company B operates in Europe

These are:
- Verb-based
- Contextual
- Non-interpretive

### What Is NOT a Fact?

Anything that requires thinking, interpretation, or aggregation across facts.

**Examples (Inferences — DO NOT STORE YET)**
- Company A is a Manufacturer
- Company B is a Customer
- Company A is a Global Distributor
- Company A dominates Market Y

These are:
- Role assignments
- Business interpretations
- Potentially context- or time-dependent

## Ontology Design Rule: Rows vs Types

### Rule

If something is derived from multiple facts, it is a row-level insight, not a type.

- Facts = graph edges
- Insights = query results, rules, or views

**Do not convert insights into:**
- Node labels
- Classes
- Boolean properties
**...until they are proven necessary and stable.**

## Class Design Principles

### Use Few, General Core Classes

**Recommended examples:**
- `Company`
- `Product`
- `Person`
- `Organization`
- `Location`
- `Region`

**Avoid:**
- `Manufacturer`
- `Distributor`
- `Customer`
- `Supplier`

These are roles, not entity types.

## Relationship Design Principles

### Prefer Verbs Over States

**Use:**
- `producesProduct`
- `sellsProductTo`
- `suppliesProductTo`
- `operatesInRegion`

**Avoid:**
- `isManufacturer`
- `isCustomer`
- `isDistributor`

> **Verbs represent facts. States represent interpretations.**

## LLM Extraction Rules

**LLMs should:**
- Extract entities
- Extract verb-based relationships
- Extract literals (dates, quantities, locations)

**LLMs should NOT:**
- Assign roles
- Classify companies by business type
- Apply domain logic
- Perform aggregation or comparison

## Inference Belongs to a Later Layer

Insights such as `Manufacturer`, `Customer`, `Distributor`, `Partner` should be produced via:
- SPARQL queries
- OWL reasoning
- Rules engines
- Application logic

**Example:**
"If a company produces a product, it can be considered a manufacturer."
*This is a rule, not a fact.*

### When to Promote an Insight into the Graph

Only promote an inferred concept into the schema if it is:
- Consistently defined
- Widely reused
- Business-critical
- Stable over time

Until then:
➡ **Keep it computed, not stored.**

## Summary Principles

1. Store facts, not conclusions
2. LLMs extract what is said, not what is meant
3. Roles emerge from relationships
4. Inference is reversible; facts are not
5. Delay schema commitment
6. Optimize for correctness over convenience

## Guiding Question

Before adding anything to the graph, ask:

> **"Is this explicitly stated in the source, or is it something we concluded?"**

If it’s a conclusion — **do not store it yet.**

---

## Intent vs Fact Principle (Handling Uncertainty Correctly)

### Why This Principle Exists

In real business systems, not all important information is certain.

We frequently need to represent:
* Potential deals
* Planned implementations
* Sales targets
* Strategic focus areas

These are **important**, but they are **not facts about the external world**.

This principle defines **how to store uncertainty without corrupting factual truth**.

### Core Distinction: Three Kinds of Truth

The graph must clearly separate **three different kinds of truth**:

#### 1. Facts (What Actually Happened)

Objective, observable, irreversible.

Examples:
* A meeting occurred
* A product was discussed
* An order was placed
* A contract was signed

**Facts are stored as:**
* Entities
* Verb-based relationships
* Events

#### 2. Intent (What We Plan or Believe Might Happen)

Declared plans, targets, expectations, or hypotheses.

Examples:
* A potential opportunity with Company A
* A sales goal of 100k GMV in 2025
* A planned deployment at Site X

**Intent is real, but it is not reality.**

Intent represents:
* Organizational belief
* Strategic focus
* Human planning

Intent **may be wrong**, and that is acceptable.

#### 3. Commitment (What Has Been Agreed)

Mutual agreement or transaction.

Examples:
* Signed contracts
* Confirmed orders
* Formal partnerships

Commitments are facts, but **stronger than events**.

### Principle: Do Not Encode Uncertainty as Fact

> **Uncertainty must never be stored as a relationship between external entities.**

This means:

❌ Do NOT:
* Create Company–Company relationships for prospects
* Label companies as customers or leads
* Treat plans as outcomes

✅ Instead:
* Store uncertainty as **Intent objects**
* Support intent with **Events and Evidence**
* Promote intent to commitment only when facts justify it

### How Intent Is Modeled

#### Intent Objects (Allowed)

The graph may include **explicit intent entities**, such as:
* `Opportunity`
* `Goal`

These entities represent:
* Planned outcomes
* Expected value
* Strategic objectives

They are **not universal facts**, but **declared intent**.

#### How Intent Connects to Facts

Intent must always be grounded in facts.

```text
(Event) ── supports ──▶ (Opportunity)
(Opportunity) ── contributesTo ──▶ (Goal)
```

This ensures:
* Traceability
* Explainability
* Auditable reasoning

### Rule: No Direct World Claims from Intent

Intent entities:
* Must NOT assert roles (customer, distributor)
* Must NOT create Company–Company commitments
* Must NOT replace factual relationships

They only express:
> *“We believe this may happen, based on these facts.”*

### Lifecycle Rule: Promotion, Not Mutation

Intent does not change into fact by updating state.

Instead:
* New facts are added (events, orders, contracts)
* A new commitment relationship is created
* Intent may become irrelevant or historical

**Reality progresses by adding facts, not rewriting intent.**

### Retrieval Principle

When querying the graph:
* Questions about **reality** → use Facts and Relationships
* Questions about **plans or pipeline** → use Intent objects
* Questions about **why we believe something** → traverse Events and Evidence

This allows agents to:
* Reason transparently
* Assess confidence
* Explain decisions

### One-Line Rule

> **Facts describe the world.
> Intent describes our plans.
> Commitments describe agreements.
> Never collapse these layers.**

### Anti-Pattern to Avoid

If removing future data would make a statement false,
**that statement should not be stored as a fact.**

---

## Event Principle (No Separate Progress Concept)

### Why This Principle Exists

In many business systems, teams attempt to model **progress**, **status**, or **stage** as first-class concepts.

This leads to:
* Mutable state
* Conflicting interpretations
* Loss of history
* Blurred boundaries between facts and workflow

This principle defines a **single, unambiguous rule** for handling progress.

### Core Rule: One Event Type, No Progress Nodes

> **All things that happen over time are modeled as `Event`.
> “Progress” is never stored — it is derived.**

The graph must:
* Use **one unified `Event` concept**
* Avoid introducing a separate `Progress` label or node

### What an Event Represents

An `Event` represents **something that happened at a specific point in time**.

Examples:
* A meeting occurred
* A statement was made
* A budget decision was communicated
* A contract was signed
* An installation started

Events may represent:
* Interactions
* External business changes
* Decisions
* Milestones
* State changes (as facts, not as current status)

### What Progress Is (and Is Not)

**Progress is NOT a thing that happens.**

Progress is:
* A **human interpretation**
* A **summary over multiple events**
* A **query result**

Examples of progress (DO NOT STORE):
* “The opportunity is advancing”
* “This deal is stalled”
* “We are close to closing”

These statements depend on:
* Time
* Context
* Business logic

They must remain **derived**, not stored.

### How Progress Is Computed

Progress is determined by:
* Ordering events by time
* Interpreting the latest relevant events
* Applying business rules in the agent or application layer

> **If something summarizes history, it must be computed, not persisted.**

### Why There Is No `Progress` Node

Introducing a `Progress` node:
* Encodes inference as fact
* Duplicates event information
* Requires mutation or overwrite
* Breaks auditability

These outcomes directly violate the **Facts First** principle.

### Append-Only History Rule

> **Reality advances by adding new events, never by mutating old ones.**

If something appears to “change state”:
* Record a new Event
* Do not update or replace prior meaning

### One-Line Rule

> **If it happened at a time, it is an Event.
> If it summarizes what happened, it is a query.**

### Validation Question

Before adding a new label or node, ask:

> **“Did this thing happen at a specific time?”**

If **yes** → Event
If **no** → Derived concept (do not store)
