# Global Distribution Knowledge Graph (Fact-First Schema)

> **A Truth-Anchored Graph for the Future of B2B Distribution**

This repository contains the schema and operating principles for the **Global Distribution Knowledge Graph**.

Unlike traditional rigid schemas, this model is designed for **LLM Extraction and Agentic Reasoning**. It prioritizes **strict factual correctness** over convenient labeling, ensuring that AI agents can reason about history, intent, and commitments without hallucinating relationships that don't exist.

## Core Philosophy: Fact-First Design

We distinguish strictly between three layers of truth. This prevents "inference leakage" where beliefs are stored as facts.

| Layer | Definition | Example | Storage |
| :--- | :--- | :--- | :--- |
| **1. Facts** | Observable, historical reality. | "Company A ordered Widget X." | `Event`, `Order`, `Equipment` |
| **2. Intent** | Beliefs, plans, or targets. | "Company A is a prospect." | `Opportunity`, `Goal` |
| **3. Commitments** | Binding agreements. | "Signed Distribution Contract." | `CommercialRelationship` |

> **Rule:** Never store Intent as a Fact. A "Prospect" is a derived state, not a label.

## The Schema (V2)

The current standard is defined in **[schema/schema_v2.md](schema/schema_v2.md)**.

### Key Design Decisions

#### 1. Unified Product Entity
*   **Decision**: There is only `Product`.
*   **Why**: A battery is a product. A robot is a product. A spare part is a product. Differentiating them via labels (`:Part`, `:Accessory`) is fragile.
*   **Implementation**: `(Product)-[:BELONGS_TO_CATEGORY]->(ProductCategory)`.

#### 2. Physical vs. Abstract (The Instance Rule)
*   **Decision**: `Equipment` is a physical instance of a `Product`.
*   **Principle**: "**Products are what you sell. Equipment is what exists.**"
*   **Implementation**: `(Equipment)-[:INSTANCE_OF]->(Product)`.

#### 3. Reified Commercial Relationships
*   **Decision**: Companies are NOT connected by direct edges like `[:PARTNER_OF]`.
*   **Why**: Relationships have state (Active, Expired), dates, and contracts. Direct edges cannot model history or expiry cleanly.
*   **Implementation**: `(Company)-[:HAS_RELATIONSHIP]->(CommercialRelationship)<-[:HAS_RELATIONSHIP]-(Company)`.

#### 4. Event-Sourced History
*   **Decision**: No mutable "Status" nodes.
*   **Why**: Progress is a summary of history, not a thing that exists.
*   **Implementation**: Store `Events` (Meetings, Signatures). Derive status at query time.

## Documentation Structure

*   **[schema/schema_v2.md](schema/schema_v2.md)**: **THE SOURCE OF TRUTH.** Key entities, properties, and relationships.
*   **[guidelines_and_examples/organizing_principles.md](guidelines_and_examples/organizing_principles.md)**: **Design Challenges & Decisions.** Read this to understand the *why* behind the schema (Fact vs Insight, Handling Uncertainty).
*   **[guidelines_and_examples/example_queries.cypher](guidelines_and_examples/example_queries.cypher)**: Standard patterns for querying this specific graph structure.

## Getting Started

1.  **Read the Principles**: Start with `guidelines_and_examples/organizing_principles.md` to align your mindset.
2.  **Review the Schema**: `schema/schema_v2.md` contains the extraction rules.
3.  **Validate**: Ensure your LLM prompts extract *Verbs* (Facts), not *Roles* (Inferences).

## License

**Copyright © 2025 Distify.ai**

This project is licensed under the **Apache License, Version 2.0**. You are free to use, modify, and distribute this schema for commercial and proprietary use, provided you give appropriate credit.
