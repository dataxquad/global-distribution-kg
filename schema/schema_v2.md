# Global Distribution Schema V2 (Rich Fact-First)

> **Core Principle**: Store Universal Facts. Roles are derived from Relationships.
> **Structure**: Facts (Commitments), Intent (Plans), Events (History), Provenance (Evidence).
> **LLM Rule**: Do not infer roles (e.g., `:Manufacturer`, `:Customer`) as node labels. Extract only what is explicitly stated or strictly implied by a verb (e.g., "manufactures" -> `[:MANUFACTURES]`).
> **Single Entity Rule**: There is only one thing: `Product`. "Part", "Accessory", and "Component" are contextual roles derived from relationships.
> **Commercial Link Rule**: Companies are connected by Commitments (`CommercialRelationship` node), not by intentions or roles. If there is no signed contract, there is no relationship.

---

## 1. Organization Entities

### `Company`
A legal or physical business entity.
*   **LLM Rule**: Extract any named organization, business, non-profit, or government entity. Do not create separate nodes for branches unless they are legally distinct subsidiaries.
*   **Properties**:
    *   `companyId` (UUID) - Unique system identifier.
    *   `legalName` (string) - Formal registered name (e.g., "Acme Corp Ltd.").
    *   `tradeName` (string) - Doing business as (DBA) name (e.g., "Acme Robotics").
    *   `taxId` (string) - Tax identification number if available.
    *   `website` (url) - Primary domain.
    *   `description` (string) - Brief summary of business activities.
    *   `status` (enum) - `Active`, `Inactive` (only if explicitly stated).
    *   `country`, `region`, `city` (string) - Headquarters location.
    *   `stockSymbol` (string) - If public.

### `Person`
A human being, typically an employee, contact, or executive.
*   **LLM Rule**: Extract named individuals. Do not create nodes for generic roles like "The Manager".
*   **Properties**:
    *   `personId` (UUID) - Unique identifier.
    *   `fullName` (string) - First and last name.
    *   `email` (string) - Business email.
    *   `phone` (string) - Contact number.
    *   `jobTitle` (string) - Official title (e.g., "VP of Sales").
    *   `roleCategory` (string) - Generalized role for grouping (e.g., "Executive", "Technical", "Sales").
    *   `timezone` (string) - IANA timezone string.
    *   `linkedinUrl` (url) - Social profile.

### `Site`
A physical location (warehouse, office, customer site, repair center).
*   **LLM Rule**: Extract distinct physical locations. Differentiate from `Company` (the entry) vs `Site` (the bricks-and-mortar place).
*   **Properties**:
    *   `siteId` (UUID) - Unique identifier.
    *   `name` (string) - Descriptive name (e.g., "Northwing Warehouse").
    *   `type` (enum) - `Warehouse`, `Office`, `Retail`, `DataCenter`, `CustomerSite`.
    *   `address` (string) - Full street address.
    *   `city`, `postalCode` (string) - Location details.
    *   `geo` (point) - Latitude/Longitude.
    *   `capacity_sqm` (number) - Usage size if specified.

### `Equipment`
A specific physical instance of a Product (e.g., a specific robot with a serial number).
*   **Principle**: **Products are what you sell. Equipment is what exists.**
*   **LLM Rule**: Only extract if a specific serial number or unique physical asset is mentioned. For general product models, use `Product`.
*   **Properties**:
    *   `equipmentId` (UUID) - Unique identifier.
    *   `serialNumber` (string) - The manufacturer's serial.
    *   `assetTag` (string) - Internal tracking tag.
    *   `firmwareVersion` (string) - Current software version.
    *   `commissionedAt` (timestamp) - Date put into service.
    *   `status` (enum) - `Active`, `InRepair`, `Retired`, `Inventory`.

---

## 2. Classification Entities

### `Territory`
Geographical or commercial Zone.
*   **LLM Rule**: Extract named sales regions or commercial zones.
*   **Properties**:
    *   `territoryId` (UUID) - Unique identifier.
    *   `name` (string) - Name (e.g., "EMEA", "North America", "France").
    *   `type` (enum) - `Country`, `Region`, `Continent`, `SalesZone`.
    *   `isoCode` (string) - ISO 3166 code if applicable (e.g., "FR", "US-CA").

### `Industry`
Business classification (ISIC).
*   **LLM Rule**: Classification codes for business sectors.
*   **Properties**:
    *   `industryId` (UUID) - Unique identifier.
    *   `isicCode` (string) - Semantic code.
    *   `title` (string) - Standardized industry name (e.g., "Manufacturing of Electronics").

---

## 3. Product Entities
### `ProductCategory`
Taxonomic classification for products.
*   **LLM Rule**: Extract clear product categories (e.g., "Cleaning Robot", "Mowing Robot"). Do NOT use for temporary features.
*   **Properties**:
    *   `categoryId` (UUID) - Unique identifier.
    *   `name` (string) - Category name (e.g., "Commercial Scrubbers").
    *   `description` (string) - Scope of the category.

### `Product`
A model, family, component, or item offered for sale or used in assembly.
*   **LLM Rule**: Extract any sellable item, model, component, or spare part. Use `type` as a hint, not a strict schema classification. **Everything is a Product.**Categories are handled via `BELONGS_TO_CATEGORY` relationships.
*   **Properties**:
    *   `productId` (UUID) - Unique identifier.
    *   `modelName` (string) - Commercial name (e.g., "T-800", "Battery-Pack-50").
    *   `brand` (string) - Brand name if different from Manufacturer.
    *   `type` (string) - Optional hint (e.g., "Robot", "Battery", "Sensor", "Consumable").
    *   `description` (string) - Marketing description.

### `SKU`
Specific stock keeping unit for a product (commercial variant).
*   **LLM Rule**: Use when specific sales configurations (colors, packs, region-specific plugs) are mentioned.
*   **Properties**:
    *   `sku` (string) - Unique string identifier.
    *   `name` (string) - Sales name (e.g., "T-800-Midnight-Black").
    *   `currency` (string) - Currency code.
    *   `listPrice` (number) - Standard price.

---

## 4. Business Commitments (Facts)
> **Rule**: These exist ONLY if a formal agreement or transaction has occurred.

### `CommercialRelationship`
A reified long-term commercial bond between companies (e.g., Franchise, Partnership).
*   **LLM Rule**: Create ONLY if there is an explicit mention of a signed agreement/contract defining the relationship. Do NOT create for prospective or informal links.
*   **Properties**:
    *   `relationshipId` (UUID) - Unique identifier.
    *   `type` (enum) - `Franchise`, `Partnership`, `ServiceAgreement`, `Distributorship`.
    *   `status` (enum) - `Active`, `Expired`, `Terminated`, `Suspended`.
    *   `startDate` (date) - Effective start.
    *   `endDate` (date) - Effective end.

### `Order`
Confirmed transaction.
*   **LLM Rule**: Extract ONLY confirmed orders. Quotes/Estimates are `Opportunities` or `Events`.
*   **Properties**:
    *   `orderId` (UUID) - Unique identifier.
    *   `orderNumber` (string) - Visible reference ID.
    *   `orderDate` (date) - Date of confirmation.
    *   `status` (enum) - `Confirmed`, `Shipped`, `Paid`, `Cancelled`.
    *   `currency` (string) - ISO currency code.
    *   `totalAmount` (number) - Final value.

### `OrderLine`
Item details within an order.
*   **Properties**:
    *   `orderLineId` (UUID) - Unique identifier.
    *   `quantity` (number) - Count.
    *   `unitPrice` (number) - Price per unit.
    *   `total` (number) - Line sum.

### `Contract`
Legal agreement document.
*   **LLM Rule**: Extract formal signed agreements. The `Contract` backs the `CommercialRelationship`.
*   **Properties**:
    *   `contractId` (UUID) - Unique identifier.
    *   `referenceNumber` (string) - External contract ID.
    *   `type` (enum) - `Distribution`, `Franchise`, `Service`, `NDA`, `Purchasing`.
    *   `startDate` (date) - Effective date.
    *   `endDate` (date) - Expiry date.
    *   `status` (enum) - `Active`, `Expired`, `Terminated`.

---

## 5. Business Intent (Plans)
> **Rule**: These represent declared plans, beliefs, or objectives. They are NOT facts about the external world.

### `Opportunity`
Declared intent or belief that a deal may happen.
*   **LLM Rule**: Extract sales leads, potential deals, or open negotiations.
*   **Properties**:
    *   `opportunityId` (UUID) - Unique identifier.
    *   `name` (string) - Deal name.
    *   `amount` (number) - Estimated value.
    *   `stage` (enum) - `Prospect`, `Qualified`, `Proposal`, `Negotiation`, `ClosedWon`, `ClosedLost`.
    *   `confidence` (float) - 0.0 to 1.0 probability.
    *   `expectedCloseDate` (date) - Forecast date.

### `Goal`
Declared strategic objective or target.
*   **LLM Rule**: Extract stated business targets (e.g., "We aim to sell 500 units").
*   **Properties**:
    *   `goalId` (UUID) - Unique identifier.
    *   `name` (string) - Summary (e.g., "100k GMV 2025").
    *   `targetValue` (number) - Numeric target.
    *   `metric` (string) - Unit of measure (e.g., "USD", "Units").
    *   `timeScope` (string) - Period (e.g., "Q1 2025").

---

## 6. Events (Interactions)
> **Rule**: All time-bound occurrences. Relationships (like "Prospect") are derived from these histories.

### `Event`
Unified node for Interactions, Activities, and State Changes.
*   **LLM Rule**: Extract any discrete event: meeting, email, status change, login, error log.
*   **Properties**:
    *   `eventId` (UUID) - Unique identifier.
    *   `eventCategory` (enum) - `Interaction`, `Activity`, `StateChange`, `SystemLog`.
    *   `eventType` (string) - Specific type (e.g., `Call`, `Email`, `Meeting`, `OrderPlaced`, `TicketClosed`, `RelationshipExpired`).
    *   `occurredAt` (timestamp) - Exact time.
    *   `summary` (string) - Short text description.
    *   `source` (string) - Origin system or reporter.
    *   `durationMinutes` (integer) - Length of event if applicable.

---

## 7. Provenance (Evidence)

### `Source`
Provenance Container (Infrastructure, not Knowledge).
*   **LLM Rule**: The file, URL, or document where facts were extracted from.
*   **Properties**:
    *   `sourceId` (UUID) - Unique identifier.
    *   `sourceType` (enum) - `PDF`, `Email`, `Chat`, `NewsArticle`, `WebPage`.
    *   `uri` (string) - Location or Path.
    *   `ingestedAt` (timestamp) - Processing time.
    *   `title` (string) - Document title.

### `Chunk`
Evidence fragment supporting facts.
*   **LLM Rule**: The specific paragraph or sentence used for extraction.
*   **Properties**:
    *   `chunkId` (UUID) - Unique identifier.
    *   `text` (string) - The content.
    *   `confidence` (float) - Extraction confidence.

---

## 8. Universal Fact Relationships (The Verbs)

### Organization & Structure
| Relationship | Source | Target | Description / LLM Use Case |
| :--- | :--- | :--- | :--- |
| `SUBSIDIARY_OF` | `Company` | `Company` | Explicit ownership structure (Child -> Parent). |
| `PARENT_OF` | `Company` | `Company` | Inverse of Subsidiary (Parent -> Child). |
| `HAS_SITE` | `Company` | `Site` | Company owns/operates a specific facility. |
| `LOCATED_IN` | `Site` | `Territory` | Geographic placement of a facility. |
| `WORKS_FOR` | `Person` | `Company` | Employment relationship. |
| `OPERATES_IN` | `Company` | `Territory` | General presence in a region (use `DISTRIBUTES` for specific product rights). |

### Commercial Links (The Bridge)
| Relationship | Source | Target | Description / LLM Use Case |
| :--- | :--- | :--- | :--- |
| `HAS_RELATIONSHIP` | `Company` | `CommercialRelationship` | Connects parties to the relationship node. |
| `DEFINED_BY` | `CommercialRelationship` | `Contract` | Links the relationship to the binding document. |
| `RELATIONSHIP_CHANGED`| `CommercialRelationship` | `Event` | Tracks state updates (Renewal, Expiry). |

### Commercial Transactions (Commitments)
| Relationship | Source | Target | Description / LLM Use Case |
| :--- | :--- | :--- | :--- |
| `PLACED_ORDER` | `Company` | `Order` | Purchaser initiating a transaction. |
| `SOLD_BY` | `Order` | `Company` | Seller in a transaction. |
| `CONTAINS_LINE` | `Order` | `OrderLine` | Structure of an order. |
| `REFERENCES_SKU` | `OrderLine` | `SKU` | Specific item ordered. |
| `BINDS_COMPANY` | `Contract` | `Company` | Party to a contract. |
| `OWNS_ASSET` | `Company` | `Equipment` | Legal ownership of specific equipment. |

### Supply Chain & Manufacturing
| Relationship | Source | Target | Description / LLM Use Case |
| :--- | :--- | :--- | :--- |
| `MANUFACTURES` | `Company` | `Product` | Company creates/builds the product. |
| `DISTRIBUTES` | `Company` | `Product` | Company distributes a product (can be inferred from CommercialRelationship). |
| `STORES` | `Site` | `Product` | Inventory held at a location. |
| `SHIPPED_FROM` | `Order` | `Site` | Logistics origin. |

### Product Relationships
| Relationship | Source | Target | Description / LLM Use Case |
| :--- | :--- | :--- | :--- |
| `BELONGS_TO_CATEGORY`| `Product` | `ProductCategory` | Explicit taxonomy classification. |
| `HAS_VARIANT` | `Product` | `SKU` | Product available as a specific commercial SKU (e.g., color, pack size). |
| `USES_COMPONENT` | `Product` | `Product` | Product requires another Product as a part (Assembly). |
| `COMPATIBLE_WITH` | `Product` | `Product` | Product works with another (e.g., Battery works with Robot). |
| `REPLACES` | `Product` | `Product` | Product is a newer version of another. |

### Intent & Planning (Uncertainty)
| Relationship | Source | Target | Description / LLM Use Case |
| :--- | :--- | :--- | :--- |
| `OWNS_OPPORTUNITY` | `Person` | `Opportunity` | Sales rep managing a deal. |
| `RELATED_TO_COMPANY` | `Opportunity` | `Company` | The prospect/customer involved in the potential deal. |
| `INVOLVES_PRODUCT` | `Opportunity` | `Product` | Product being discussed. |
| `CONTRIBUTES_TO` | `Opportunity` | `Goal` | Link deal value to a strategic target. |
| `OWNS_GOAL` | `Person` | `Goal` | Person accountable for the metric. |

### Equipment, Usage & Service
| Relationship | Source | Target | Description / LLM Use Case |
| :--- | :--- | :--- | :--- |
| `INSTANCE_OF` | `Equipment` | `Product` | **Canonical link.** The equipment is a physical instance of `Product` model. |
| `INSTALLED_AT` | `Equipment` | `Site` | Physical location of a machine. |
| `MAINTAINS` | `Person` | `Equipment` | Technician servicing a machine. |
| `SUPPORTED_BY` | `Equipment` | `Company` | Company responsible for SLA/Support. |

### Events, Usage & Provenance
| Relationship | Source | Target | Description / LLM Use Case |
| :--- | :--- | :--- | :--- |
| `PARTICIPATED_IN` | `Person` | `Event` | Attendee of a meeting/call. |
| `ORGANIZED` | `Company` | `Event` | Host of an event. |
| `SUBJECT_OF` | `Company` | `Event` | Topic of the event. |
| `DISCUSSED` | `Event` | `Product` | Product mentioned in interaction. |
| `GENERATED` | `Event` | `Opportunity` | Event that created the lead. |
| `CHANGED_STATE_OF` | `Event` | `Opportunity` | Event that moved stage. |
| `HAS_CHUNK` | `Source` | `Chunk` | Text segmentation. |
| `MENTIONS` | `Chunk` | `AnyEntity` | Extraction link (Evidence). |
| `SUPPORTS` | `Chunk` | `Event` | Evidence for an event happening. |

---

## 9. Derived Roles (Query Examples)

**Query: Who are the Active Franchises?**
```cypher
MATCH (hq:Company)-[:HAS_RELATIONSHIP]->(r:CommercialRelationship {type: 'Franchise', status: 'Active'})<-[:HAS_RELATIONSHIP]-(n:Company)
RETURN n, hq
```

**Query: Find all Cleaning Robots (via Category)**
```cypher
MATCH (p:Product)-[:BELONGS_TO_CATEGORY]->(c:ProductCategory {name: 'CleaningRobot'})
RETURN p
```

**Query: Which companies own "Skywalker-50" units?**
```cypher
MATCH (c:Company)-[:OWNS_ASSET]->(e:Equipment)-[:INSTANCE_OF]->(p:Product {modelName: "Skywalker-50"})
RETURN DISTINCT c
```

**Query: What are the spare parts for "Skywalker-50"?**
```cypher
MATCH (p:Product {modelName: "Skywalker-50"})-[:USES_COMPONENT]->(component:Product)
RETURN component
```
