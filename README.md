# mdb-osi

# OSI -- Open Semantic Interchange & MDB-Engine

> **Status: IMPLEMENTED.** The OSI integration described in this document is fully built. See `mdb_engine/osi/` for the implementation, `OSI_PROPOSED_INTEGRATION.md` for the engineering blueprint, and `OSI_MANIFEST.md` for manifest configuration. The first working example is `sso-app-3` with a comprehensive family management semantic model (10 datasets, 12 relationships, 6 metrics, 515 synonyms).

An analysis of the [Open Semantic Interchange (OSI)](https://opensemanticinterchange.org/) standard, how it relates to MDB-Engine's knowledge graph and memory systems, and where the opportunities lie.

---

## Table of Contents

1. [What is OSI?](#1-what-is-osi)
2. [Where MDB-Engine Overlaps with OSI](#2-where-mdb-engine-overlaps-with-osi)
3. [Concrete Opportunities](#3-concrete-opportunities)
4. [Architectural Considerations](#4-architectural-considerations)
5. [Strategic Position](#5-strategic-position)

---

## 1. What is OSI?

The **Open Semantic Interchange (OSI)** is an open-source, vendor-neutral standard for defining and sharing semantic data across AI, BI, and data platforms. It was launched in September 2025 by Snowflake with founding partners including dbt Labs, Salesforce, RelationalAI, ThoughtSpot, Atlan, Alation, and Sigma. As of early 2026, the working group has expanded to 40+ organizations and the [specification is finalized](https://www.snowflake.com/en/blog/open-semantic-interchanges-specs-finalized/) under an Apache 2.0 license.

### The Core Problem OSI Solves

Every organization defines "revenue" differently in every dashboard, notebook, and AI prompt. Metrics drift across tools and trust erodes. OSI eliminates this by providing a **single, declarative standard** for business semantics:

> "Define your business metrics once. Use them everywhere."

### How It Works

OSI uses a **declarative YAML specification** (powered by [MetricFlow](https://github.com/dbt-labs/metricflow), dbt's open-source semantic engine) to define:

| OSI Concept | What It Represents |
|---|---|
| **Semantic Models** | Data entry points (tables/views with metadata) |
| **Entities** | Join keys and typed identifiers (primary, foreign, unique) |
| **Measures** | Aggregatable quantities (`SUM(amount_usd)`, `COUNT(orders)`) |
| **Dimensions** | Ways to slice data (time, category, geography) |
| **Metrics** | Governed calculations combining measures and dimensions |
| **Relationships** | How semantic models connect via entities |

Example from the [OSI site](https://opensemanticinterchange.org/):

```yaml
semantic_model:
  name: orders
  description: Core order data from the ERP system
  model: raw_orders

  entities:
    - name: order_id
      type: primary
    - name: customer_id
      type: foreign

  measures:
    - name: order_total
      expr: amount_usd
      agg: sum

  dimensions:
    - name: order_date
      type: time
      type_params:
        time_granularity: day
```

MetricFlow compiles these definitions into **deterministic, provably-correct SQL** -- unlike text-to-SQL approaches that are probabilistic and unreliable. This means when an AI agent asks "what was revenue last quarter?", the answer is governed and auditable, not a best-guess.

### Key Players and Their Angles

| Organization | Role in OSI |
|---|---|
| **Snowflake** | Initiative leader, platform integration via Cortex Agents |
| **dbt Labs** | Open-sourced MetricFlow (Apache 2.0) as the OSI engine |
| **Salesforce / Tableau** | BI tool interoperability |
| **RelationalAI** | Bringing **relational knowledge graph semantics** into OSI for decision intelligence and AI reasoners |
| **ThoughtSpot** | "Agentic Semantic Layer" -- AI agents that query governed metrics |
| **Atlan / Alation** | Data catalog integration, metadata governance |

### The Knowledge Graph Connection (RelationalAI)

RelationalAI's involvement is particularly relevant to us. Their CEO Molham Aref stated:

> "RelationalAI has the opportunity to bring relational knowledge graph semantics into OSI to enable decision intelligence more broadly, and give customers and partners a durable way to define semantics once and use them everywhere -- from Snowflake Cortex Agent to other agentic applications."

This means **knowledge graphs are already part of the OSI conversation**. RelationalAI combines knowledge graphs with LLMs for text-to-reasoner capabilities that go beyond traditional RAG and text-to-SQL -- which is exactly the territory MDB-Engine operates in.

### References

- [OSI Official Site](https://opensemanticinterchange.org/)
- [Snowflake Announcement](https://www.snowflake.com/en/blog/open-semantic-interchange-ai-standard/)
- [Specification Finalized (Jan 2026)](https://www.snowflake.com/en/blog/open-semantic-interchanges-specs-finalized/)
- [dbt Open Sources MetricFlow](https://www.getdbt.com/blog/dbt-labs-affirms-commitment-to-open-semantic-interchange-by-open-sourcing-metricflow)
- [ThoughtSpot: The Agentic Semantic Layer and OSI](https://www.thoughtspot.com/blog/the-agentic-semantic-layer-and-OSI)
- [RelationalAI Joins OSI](https://www.relational.ai/post/rai-joins-snowflake-and-industry-leaders-to-establish-the-open-semantic-interchange)
- [OSI GitHub Repository](https://github.com/open-semantic-interchange/OSI)

---

## 2. Where MDB-Engine Overlaps with OSI

MDB-Engine's knowledge graph and memory system share deep structural parallels with OSI -- but approach the problem from the **opposite direction**. Understanding both sides reveals the opportunity.

### The Fundamental Inversion

```
OSI:         Analysts define structure up front  -->  Tools consume it
MDB-Engine:  Users converse naturally  -->  System discovers structure from text
```

OSI is **top-down**: domain experts declare entities, metrics, dimensions, and relationships in YAML. These definitions are static, governed, and auditable.

MDB-Engine is **bottom-up**: the LLM extraction engine (`mdb_engine/graph/extraction.py`) discovers entities and relationships from unstructured conversation, building a living knowledge graph that evolves with every interaction.

**These are not competing approaches. They are complementary halves of the same problem.**

### Concept Mapping

| MDB-Engine Construct | Code Location | OSI Equivalent | Notes |
|---|---|---|---|
| **Nodes** (`type:name`) | `graph/nodes.py` | **Entities** | Both are typed identifiers. MDB-Engine uses `person:alex_smith`; OSI uses `{name: customer_id, type: primary}` |
| **Node Types** | `graph/prompts.py` line 96 | **Entity Types** | MDB-Engine: `person, organization, project, event, location, concept, document, interest, product`. OSI: `primary, foreign, unique, natural` |
| **Edges** (relation, target, weight) | `graph/edges.py` | **Relationships / Joins** | Both define typed connections between entities. MDB-Engine adds `weight` and `active` for temporal modeling |
| **Relationship Types** | `graph/prompts.py` lines 100-103 | **Join Types** | MDB-Engine: `knows, likes, works_at, parent_of, ...` (behavioral). OSI: structural joins between semantic models |
| **Node Properties** | `graph/nodes.py` `properties` dict | **Dimensions** | Both attach typed attributes to entities. MDB-Engine properties are freeform; OSI dimensions are typed (`time`, `categorical`) |
| **Community Summaries** | `graph/community.py` | **Semantic Model Descriptions** | Both provide natural-language descriptions of entity clusters. MDB-Engine generates them with LLM; OSI authors them manually |
| **Graph Extraction** | `graph/extraction.py` | *(No equivalent)* | OSI has no extraction -- it requires manual definition. This is MDB-Engine's unique contribution |
| **Vector Embeddings** | `graph/nodes.py` `embedding` field | *(No equivalent)* | OSI doesn't embed entities. MDB-Engine embeds every node for hybrid vector+graph search |
| *(No equivalent)* | -- | **Measures / Metrics** | MDB-Engine doesn't define aggregatable calculations. This is OSI's core value-add |

### Where the Schemas Align

**MDB-Engine Node Document:**

```json
{
  "_id": "organization:mongodb",
  "type": "organization",
  "name": "MongoDB",
  "properties": {"industry": "database technology"},
  "edges": [
    {"relation": "employs", "target": "person:users_cousin", "weight": 1.0, "active": true}
  ],
  "embedding": [0.012, -0.045, ...],
  "user_id": "user123",
  "app_slug": "my_app",
  "source_memory_ids": ["mem_abc"]
}
```

**OSI Semantic Model (YAML):**

```yaml
semantic_model:
  name: organizations
  description: Known organizations and their relationships
  entities:
    - name: org_id
      type: primary
    - name: employee_id
      type: foreign
  dimensions:
    - name: industry
      type: categorical
  measures:
    - name: employee_count
      agg: count
```

The structural overlap is clear: both represent typed entities with properties/dimensions and typed relationships. The difference is in **origin** (discovered vs. declared) and **purpose** (conversational memory vs. governed analytics).

### Where the Schemas Diverge

| Aspect | MDB-Engine | OSI |
|---|---|---|
| **Origin** | LLM-extracted from conversation | Human-authored by analysts |
| **Mutability** | Living, constantly evolving | Static, versioned, governed |
| **Scope** | Per-user, per-app (multi-tenant) | Org-wide, shared truth |
| **Embeddings** | Every node has a vector | No vector semantics |
| **Weights** | Edges have 0.0-1.0 weight | Joins are binary (exists or not) |
| **Temporal** | `active` flag for soft-delete | No temporal modeling |
| **Aggregation** | No measures/metrics | Core purpose |
| **Query** | `$vectorSearch` + `$graphLookup` | Compiled to deterministic SQL |

---

## 3. Concrete Opportunities

### A. Import: Consume OSI Semantic Models to Bootstrap and Enrich Graphs

**Idea**: Parse OSI YAML definitions and use them to pre-seed or constrain MDB-Engine's knowledge graph.

#### A1. Dynamic Entity Types from OSI

Currently, MDB-Engine's entity types are hardcoded in `graph/prompts.py`:

```python
# Types: person, organization, project, event, location, concept, document, interest, product.
```

With OSI integration, these could be **dynamically loaded** from an organization's OSI semantic models. If a company's OSI spec defines semantic models for `orders`, `customers`, `products`, and `revenue`, those become valid node types for extraction:

```python
# Instead of hardcoded types, load from OSI:
osi_entity_types = parse_osi_yaml("semantic_models/")
# Result: ["order", "customer", "product", "revenue", "campaign", ...]
```

This means the extraction prompt would understand the organization's specific domain entities, not just generic types.

#### A2. Ground Truth for Entity Resolution

MDB-Engine's biggest entity resolution weakness is disambiguation -- "Alex" vs. "Alex Smith" vs. "Alexander Smith" may create separate nodes. If the organization has an OSI semantic model with a `customers` entity, that becomes a **ground truth registry**:

```
User says: "I spoke with Alex from the Portland team"
Graph extraction finds: person:alex
OSI lookup finds: customer_id: alex_smith (Portland office)
Resolution: Merge to person:alex_smith with high confidence
```

#### A3. Pre-seeded Relationship Types

OSI's entity relationships (joins) could augment MDB-Engine's relationship taxonomy. If an OSI model defines that `customers` connects to `orders` via `customer_id`, the extraction engine knows that `customer --placed--> order` is a valid relationship pattern.

---

### B. Export: Publish Discovered Knowledge as OSI-Compatible Artifacts

**Idea**: Generate OSI-compatible YAML from MDB-Engine's discovered knowledge graph.

This is where MDB-Engine becomes genuinely unique in the OSI ecosystem. No other tool discovers semantic structure from unstructured conversation.

#### B1. Conversation-Driven Semantic Discovery

Consider this scenario: A sales team uses an MDB-Engine-powered assistant for months. Over time, the knowledge graph accumulates:

- **Nodes**: `organization:acme`, `person:sarah_chen`, `product:enterprise_plan`, `concept:renewal_risk`
- **Edges**: `sarah_chen --manages--> acme`, `acme --subscribed_to--> enterprise_plan`, `acme --flagged_for--> renewal_risk`
- **Community**: "Enterprise account cluster around Acme Corp with renewal concerns"

This could be **exported** as an OSI semantic model:

```yaml
# Auto-generated from MDB-Engine knowledge graph
semantic_model:
  name: discovered_accounts
  description: >
    Account relationships discovered from sales team conversations.
    Community: Enterprise account cluster around Acme Corp with renewal concerns.
  entities:
    - name: account_id
      type: primary
      description: "Organizations identified in conversations"
    - name: contact_id
      type: foreign
      description: "People associated with accounts"
  dimensions:
    - name: risk_status
      type: categorical
      description: "Discovered risk indicators (e.g., renewal_risk)"
```

This makes MDB-Engine a **semantic discovery engine** -- it finds structure that analysts haven't yet formalized.

#### B2. Community Summaries as Semantic Descriptions

MDB-Engine's hierarchical community summaries (`community.py`) are LLM-generated natural-language descriptions of entity clusters. These map directly to OSI's `description` fields:

```
MDB-Engine Community Summary:
  "This community revolves around Acme Corp's enterprise subscription,
   managed by Sarah Chen, with an active renewal risk flag."

OSI Semantic Model Description:
  description: "Acme Corp's enterprise subscription, managed by Sarah Chen,
               with an active renewal risk flag."
```

The community detection + LLM summarization pipeline could directly produce OSI-compatible metadata.

---

### C. Bridge: OSI as the Interoperability Layer

**Idea**: Use OSI as a bridge between MDB-Engine's conversational knowledge and enterprise governed data.

This is the highest-impact opportunity.

#### C1. Governed Answers for Business Questions

When a user asks their MDB-Engine assistant: *"What was our revenue last quarter?"*

Today, the system would search memories and graph context for anything the user has *said* about revenue. But this is unreliable -- it's conversational recall, not data.

With an OSI bridge:

```
User: "What was our revenue last quarter?"

1. Graph classifies query as "global" (thematic business question)
2. System detects "revenue" matches an OSI-defined metric
3. Routes to MetricFlow for deterministic SQL compilation
4. Returns: "$4.2M (governed metric, source: finance.orders)"
5. Enriches with graph context: "Your team discussed revenue targets
   with Sarah Chen on Jan 15. She mentioned Acme's renewal at risk."
```

The governed metric provides the **trusted number**. The knowledge graph provides the **conversational context** around it. Together they give the user both the fact and the narrative.

#### C2. Grounding Conversational Entities in Enterprise Data

MDB-Engine extracts nodes like `organization:acme` from conversation. OSI semantic models define `customers` with structured dimensions (industry, region, ARR). Linking the two:

```
Graph node: organization:acme
  |
  ├── OSI link: customers.customer_id = "acme_corp"
  │   ├── industry: "Manufacturing"
  │   ├── region: "Pacific Northwest"
  │   └── arr: $2.4M
  |
  └── Graph edges (from conversation):
      ├── manages: person:sarah_chen
      ├── subscribed_to: product:enterprise_plan
      └── flagged_for: concept:renewal_risk
```

Now when the user asks "Tell me about Acme", the system provides both the governed data dimensions AND the conversational knowledge graph -- a complete picture that neither source could provide alone.

#### C3. Semantic-Aware Extraction

The most immediate technical win: pass OSI entity definitions as additional context to the graph extraction prompt. Instead of:

```
"Extract entities and relationships from this text..."
```

The prompt could include:

```
"Extract entities and relationships from this text.
 The organization uses these semantic models:
 - customers (entities: customer_id, account_manager_id)
 - orders (entities: order_id, customer_id; measures: order_total)
 - products (entities: product_id; dimensions: category, tier)

 When you identify entities, prefer mapping to these known models."
```

This would dramatically improve entity resolution, reduce hallucinated node types, and align the discovered graph with the organization's existing data model.

---

## 4. Architectural Considerations

### 4.1 Schema Alignment

Mapping between MDB-Engine's graph schema and OSI's YAML spec requires bridging two fundamentally different data models:

| Challenge | Detail |
|---|---|
| **ID formats** | MDB-Engine: `type:snake_case_name` (e.g., `person:alex_smith`). OSI: named entities within semantic models (e.g., `customer_id` of type `primary`). Need a mapping layer. |
| **Type systems** | MDB-Engine types are domain categories (`person`, `organization`). OSI types are structural roles (`primary`, `foreign`). These are orthogonal -- a node could be both `type: organization` (MDB) and `type: primary` (OSI role). |
| **Properties vs. Dimensions** | MDB-Engine properties are untyped dicts. OSI dimensions are typed (`time`, `categorical`). Export would need type inference. |
| **Edges vs. Joins** | MDB-Engine edges have weight, active flag, and free-form relation names. OSI joins are structural and binary. Only a subset of edges (structural relationships like `works_at`, `member_of`) would map cleanly. |

A practical mapping layer might look like:

```python
# Conceptual: OSI <-> MDB-Engine bridge
class OSIBridge:
    def import_semantic_model(self, osi_yaml: dict) -> list[GraphNode]:
        """Convert OSI entities/dimensions to MDB-Engine nodes."""
        ...

    def export_to_osi(self, nodes: list[GraphNode]) -> dict:
        """Convert MDB-Engine nodes/edges to OSI YAML."""
        ...

    def resolve_entity(self, node_id: str) -> OSIEntity | None:
        """Check if a graph node maps to a known OSI entity."""
        ...
```

### 4.2 Bidirectional Sync

The living nature of MDB-Engine's graph creates a sync challenge:

```
OSI Model (stable, versioned)          MDB-Engine Graph (living, evolving)
┌──────────────────────────┐           ┌──────────────────────────┐
│ customers:               │           │ organization:acme        │
│   - customer_id (PK)     │◄── link ──│   edges:                 │
│   - industry (dim)       │           │     manages -> sarah     │
│   - region (dim)         │           │     risk -> renewal      │
│                          │           │   properties:            │
│ v2.3 (last updated       │           │     industry: "Mfg"      │
│  Jan 2026)               │           │   (updated 5 min ago)    │
└──────────────────────────┘           └──────────────────────────┘
```

**Challenges:**
- OSI models are versioned and change infrequently. The graph changes with every conversation.
- Which source of truth wins for `industry`? The governed OSI dimension or the conversationally-discovered property?
- When a new node type appears in conversation that doesn't match any OSI model, does it get flagged for analyst review?

**Proposed approach:** OSI is always the **authority** for governed data (metrics, dimensions). MDB-Engine is the **authority** for conversational knowledge (relationships, context, temporal signals). Conflicts are resolved by source type, not recency.

### 4.3 LLM Extraction with OSI Context

The graph extraction prompt in `mdb_engine/graph/prompts.py` currently uses hardcoded entity types and relationship types. With OSI integration, the extraction system prompt could be dynamically augmented:

```python
# Current (static):
GRAPH_EXTRACTION_SYSTEM_PROMPT = """...
Types: person, organization, project, event, location, concept, document, interest, product.
..."""

# With OSI (dynamic):
def build_extraction_prompt(osi_models: list[dict]) -> str:
    osi_context = format_osi_models_for_prompt(osi_models)
    return f"""...
Types: person, organization, project, event, location, concept, document, interest, product.

ORGANIZATION SEMANTIC MODELS (prefer these when matching entities):
{osi_context}
..."""
```

This is the lowest-effort, highest-impact integration point. It doesn't require schema changes or new infrastructure -- just prompt engineering with OSI context.

### 4.4 Multi-Tenant Scope Differences

OSI semantic models are **organization-wide** -- they define what "revenue" means for the whole company. MDB-Engine graphs are scoped by **`app_slug`** and **`user_id`** -- each user has their own knowledge graph.

The bridge must handle this scope asymmetry:

```
OSI: org-wide semantic models
  └── Shared by all apps and users

MDB-Engine:
  └── app_slug: "sales_assistant"
        └── user_id: "sarah@acme.com"
              └── Personal knowledge graph
        └── user_id: "mike@acme.com"
              └── Personal knowledge graph
  └── app_slug: "support_bot"
        └── user_id: "customer_123"
              └── Personal knowledge graph
```

**Proposed approach:**
- OSI models are loaded at the **app level** (shared across all users of an app)
- Personal graph nodes link to OSI entities but don't modify them
- OSI-grounded queries (governed metrics) are user-agnostic
- Graph-grounded queries (conversational context) remain user-scoped

---

## 5. Strategic Position

### The Spectrum of Data Semantics

```
UNSTRUCTURED                                                    STRUCTURED
conversational ◄──────────────────────────────────────────────► governed

  "My cousin works      Knowledge Graph        OSI Semantic       Deterministic
   at MongoDB"          (MDB-Engine)            Models (YAML)      SQL (MetricFlow)
                        ┌────────────┐          ┌────────────┐     ┌────────────┐
   Raw text ──LLM──►    │ Nodes      │──bridge──│ Entities   │──►  │ SELECT     │
   extraction            │ Edges      │          │ Dimensions │     │   SUM(...) │
                        │ Communities│          │ Measures   │     │ FROM ...   │
                        └────────────┘          └────────────┘     └────────────┘

   MDB-Engine                                   OSI / dbt / MetricFlow
   territory                                    territory
```

MDB-Engine sits at the **unstructured/conversational** end. OSI sits at the **structured/governed** end. Today, these are separate worlds. The opportunity is to bridge them.

### The Value Proposition

> **Your BI tools define what "revenue" means (via OSI). Your AI assistant remembers what you said about it last Tuesday (via MDB-Engine). Both speak the same semantic language.**

No other system in the OSI ecosystem does what MDB-Engine does: **discover semantic structure from unstructured conversation and build a living knowledge graph**. Every other OSI participant consumes or transforms semantics that humans have already defined. MDB-Engine could be the only tool that **produces new semantic knowledge** for the ecosystem.

### MongoDB's Position

MongoDB Atlas is already the operational data platform for many organizations. MDB-Engine makes it the substrate for both:

1. **Conversational AI memory** -- per-user knowledge graphs built from interaction
2. **Structured semantic graphs** -- bridged to OSI for enterprise-grade governed analytics

The `$graphLookup` + `$vectorSearch` combination is unique to MongoDB and perfectly positioned for this hybrid role. No other database offers native graph traversal, vector search, AND document flexibility in a single platform.

### Competitive Landscape

| Player | What They Do | What We Add |
|---|---|---|
| **RelationalAI** | Knowledge graphs + reasoning on Snowflake | We do it on MongoDB, from conversation, with memory |
| **ThoughtSpot** | Agentic semantic layer for BI | We build the knowledge graph that feeds semantic layers |
| **dbt / MetricFlow** | Governed metric compilation | We discover the entities that metrics are defined over |
| **Atlan / Alation** | Data catalogs with OSI integration | We generate catalog-worthy metadata from conversation |

### Opportunity Summary

| Opportunity | Effort | Impact | Uniqueness |
|---|---|---|---|
| **OSI-aware extraction prompts** | Low (prompt engineering) | High (better entity resolution) | Medium |
| **Import OSI models as node type registry** | Medium (YAML parser + config) | Medium (domain-specific graphs) | Medium |
| **Export graph as OSI YAML** | Medium (serialization layer) | High (semantic discovery) | Very High -- no one else does this |
| **Bridge to governed metrics** | High (MetricFlow integration) | Very High (trusted answers + context) | Very High |
| **Community summaries as OSI descriptions** | Low (formatting) | Medium (metadata generation) | High |

### The Big Idea

MDB-Engine could become the **conversational front-end** for the OSI ecosystem:

1. Users talk to their AI assistant naturally
2. MDB-Engine discovers entities, relationships, and patterns from those conversations
3. Discovered semantics are exported as OSI-compatible artifacts
4. Analysts review and promote them into governed OSI models
5. Governed models flow back into MDB-Engine to improve future extraction

This creates a **flywheel** where conversation drives semantic discovery, which feeds governance, which improves conversation -- the virtuous cycle between unstructured intelligence and structured truth.

---
