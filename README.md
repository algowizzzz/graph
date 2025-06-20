# graph


# RiskGPT Data Ingestion & Metadata Specification

## 1. Objective

Establish a scalable, standards-driven pipeline to ingest, structure, embed, index, and surface both unstructured and structured data for the RiskGPT enterprise agent. This enables metadata-aware semantic and temporal retrieval, supporting graph-based knowledge exploration across BMO internal documents, external regulatory filings, APIs, and curated news.

Key features:

- Section‑level JSON docs with complete metadata
- Embedding generation and storage
- Metadata‑aware vector‑store queries
- Graph DB for relationships and dependencies

---

## 2. Epics & User Stories

### Epic 1: Data Ingestion & Preprocessing

### **Story 1.1 – RiskGPT Document Standardization**

Working with different file types, .doc, ppt, .txt. .md, .py, .html

**Test Cases:**
successful ingestion of atleast one doc for each file type 
****

### **Story 1.2 – RiskGPT Document Standardization**

I want to split each consolidated financial statement into section‑level JSON docs with metadata (domain, subdomain, data_classification, document_type, context, year, quarter, section, filename) so that downstream components receive uniformly structured inputs. DYNAMICALLY, not by page numbers, by identifying section titles and table of contents. 

**Test Cases:**

1. **Metadata completeness** – Every JSON doc includes all required fields.
2. **JSON validity** – Output passes schema validation.
3. **Visuals → Tables** – Embedded charts extracted into arrays, not blobs.

### **Story 1.3 – Embedding Generation**

I want to compute and store an embedding vector for each RiskGPT doc as one chunk so that semantic retrieval can operate reliably.

**Test Cases:**

1. **Vector shape** – Each embedding is a fixed-length float vector (e.g., 1×1536).
2. **Non-null values** – No element is NaN or null.
3. **Persistence** – Querying by doc ID returns the same vector.

---

## Epic 2: Metadata Indexing & Storage

### **Story 2.1– Vector Store Provisioning with Metadata Filters**

I want to spin up the vector store with metadata‑aware similarity search so I can query embeddings with structured filters. 

- With top K = 5, 3 and 1 to be used for different use cases

**Test Cases:**

1. Queries to be provided for testing 
2. **Filter capability** – Filtering by `{RISKGPT doc name, context domain, year}` returns correct docs.

---

## **2.2 Standardized MetaData Structure**

### Unified JSON Schema for RISKGPT Documents

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "RiskGPT Document",
  "type": "object",
  "required": [
    "domain",
    "subdomain",
    "data_classification",
    "document_type",
    "context",
    "section",
    "filename",
    "ingestion_date",
    "source_uri",
    "checksum",
    "version",
    "embedding_id",
    "word_count"
  ],
  "properties": {
    "domain": {
      "type": "string",
      "enum": ["external", "ERPM"]
    },
    "subdomain": {
      "type": "string",
      "minLength": 1
    },
    "data_classification": {
      "type": "string",
      "enum": ["public", "internal", "confidential"]
    },
    "document_type": {
      "type": "string",
      "minLength": 1
    },
    "context": {
      "type": "string",
      "minLength": 1
    },
    "year": {
      "type": "integer",
      "minimum": 1900,
      "maximum": 2100
    },
    "quarter": {
      "type": ["string", "null"],
      "pattern": "^Q[1-4]$"
    },
    "section": {
      "type": "string",
      "minLength": 1
    },
    "filename": {
      "type": "string",
      "pattern": "^(external|ERPM)_[A-Za-z0-9&]+_[A-Za-z0-9]+_[A-Za-z0-9&]+_\d{4}(?:_Q[1-4])?_[A-Za-z0-9]+$"
    },
    "ingestion_date": {
      "type": "string",
      "format": "date"
    },
    "source_uri": {
      "type": "string",
      "format": "uri"
    },
    "checksum": {
      "type": "string",
      "pattern": "^[A-Fa-f0-9]{64}$"
    },
    "version": {
      "type": "integer",
      "minimum": 1
    },
    "embedding_id": {
      "type": "string",
      "format": "uuid"
    },
    "parent_id": {
      "type": ["string", "null"],
      "format": "uuid"
    },
    "word_count": {
      "type": "integer",
      "minimum": 0
    }
  },
  "additionalProperties": false
}

```

---

## Example Documents

### Domains & Subdomains

- **external**
    - subdomain: `SEC` → Used for filings like 10-K, 6-K, 40-F, etc.
    - subdomain: `news` → NewsItem documents from sources like Yahoo, Reuters
- **ERPM**
    - subdomain: `MR` → Market Risk reports, SOPs, BMA docs
    - subdomain: `C&CCR` → Credit & Counterparty Risk policies, procedures, TMA

### Document Types

- `10K`, `6K`, `40F` – Regulatory filings (external → SEC)
- `Report` – Quarterly/periodic risk summaries (internal → ERPM)
- `Policy`, `SOP`, `TMA`, `BMA`, `internal_process` – Governance & operational docs (internal → ERPM)
- `NewsItem` – External public news (external → news)

### Sample Documents

### 4.1 External SEC Filing (Public)

```json
{
  "domain": "external",
  "subdomain": "SEC",
  "data_classification": "public",
  "document_type": "10K",
  "context": "RBC",
  "year": 2025,
  "quarter": "Q1",
  "section": "Financials",
  "filename": "external_SEC_10K_RBC_2025_Q1_Financials",
  "ingestion_date": "2025-06-19",
  "source_uri": "https://sec.gov/.../rbc2025q1",
  "checksum": "4bf0...e3a9",
  "version": 1,
  "embedding_id": "3d2f...aa4e",
  "parent_id": null,
  "word_count": 4200
}

```

### 4.2 BMO Quarterly Report (Internal)

```json
{
  "domain": "ERPM",
  "subdomain": "MR",
  "data_classification": "internal",
  "document_type": "Report",
  "context": "MRCROQuarterlyReports",
  "year": 2025,
  "quarter": "Q1",
  "section": "RWA",
  "filename": "ERPM_MR_Report_MRCROQuarterlyReports_2025_Q1_RWA",
  "ingestion_date": "2025-06-19",
  "source_uri": "https://sharepoint/.../Q1_2025_RWA",
  "checksum": "a7d3...c8b2",
  "version": 1,
  "embedding_id": "5aa1...fc99",
  "parent_id": "7e4c...ab12",
  "word_count": 3800
}

```

### 4.3 BMO Policy (Short, Internal)

```json
{
  "domain": "ERPM",
  "subdomain": "C&CCR",
  "data_classification": "internal",
  "document_type": "Policy",
  "context": "Utilities&PowerGeneration",
  "year": 2022,
  "quarter": "Q1",
  "section": "all",
  "filename": "ERPM_C&CCR_Policy_Utilities&PowerGeneration_2022_Q1_all",
  "ingestion_date": "2025-06-19",
  "source_uri": "https://sharepoint/.../UtilitiesPolicy2022",
  "checksum": "cc1f...dea8",
  "version": 1,
  "embedding_id": "bcd2...ed91",
  "parent_id": null,
  "word_count": 2400
}

```

### 4.4 BMO Policy (Subdivided, Internal)

```json
{
  "domain": "ERPM",
  "subdomain": "C&CCR",
  "data_classification": "internal",
  "document_type": "Policy",
  "context": "CCLM",
  "year": 2024,
  "quarter": "Q1",
  "section": "FinancialAnalysis",
  "filename": "ERPM_C&CCR_Policy_CCLM_2024_Q1_FinancialAnalysis",
  "ingestion_date": "2025-06-19",
  "source_uri": "https://sharepoint/.../CCLM_Policy_2024",
  "checksum": "e7af...cc29",
  "version": 1,
  "embedding_id": "9fa0...ac42",
  "parent_id": "2e9c...ba88",
  "word_count": 2100
}

```

### 4.5 News Item (External)

```json
{
  "domain": "external",
  "subdomain": "news",
  "data_classification": "public",
  "document_type": "NewsItem",
  "context": "YahooFinance",
  "year": 2025,
  "quarter": null,
  "section": "all",
  "filename": "external_news_NewsItem_YahooFinance_20250619_all",
  "ingestion_date": "2025-06-19",
  "source_uri": "https://finance.yahoo.com/...",
  "checksum": "f3d2...b1c0",
  "version": 1,
  "embedding_id": "c4d5...e7f8",
  "parent_id": null,
  "word_count": 350
}

```

---

## 5. Structured API Metadata Schema

```json
{
  "domain": "ERPM",
  "subdomain": "MR",
  "data_classification": "confidential",
  "context": "CCR",
  "section": "Adaptiv",
  "api_name": "LimitUtilisation",
  "date": "2025-06-19",
  "dependencies": {
    "depends_on": ["ERPM_MR_APIS_CCR_Adaptiv_RiskView_20250619"],
    "used_by": ["MR_Reporting"]
  },
  "filename": "ERPM_MR_APIS_CCR_Adaptiv_LimitUtilisation_20250619",
  "ingestion_date": "2025-06-19"
}

```

---

---

## Complete JSON Schema Definition

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "RiskGPT Document",
  "type": "object",
  "required": [
    "domain",
    "subdomain",
    "data_classification",
    "document_type",
    "context",
    "section",
    "filename",
    "ingestion_date",
    "source_uri",
    "checksum",
    "version",
    "embedding_id",
    "word_count"
  ],
  "properties": {
    "domain": {"type": "string", "enum": ["external", "ERPM"]},
    "subdomain": {"type": "string", "minLength": 1},
    "data_classification": {"type": "string", "enum": ["public", "internal", "confidential"]},
    "document_type": {"type": "string", "minLength": 1},
    "context": {"type": "string", "minLength": 1},
    "year": {"type": "integer", "minimum": 1900, "maximum": 2100},
    "quarter": {"type": ["string", "null"], "pattern": "^Q[1-4]$"},
    "section": {"type": "string", "minLength": 1},
    "filename": {"type": "string", "pattern": "^(external|ERPM)_[A-Za-z0-9&]+_[A-Za-z0-9]+_[A-Za-z0-9&]+_\\d{4}(?:_Q[1-4])?_[A-Za-z0-9]+$"},
    "ingestion_date": {"type": "string", "format": "date"},
    "source_uri": {"type": "string", "format": "uri"},
    "checksum": {"type": "string", "pattern": "^[A-Fa-f0-9]{64}$"},
    "version": {"type": "integer", "minimum": 1},
    "embedding_id": {"type": "string", "format": "uuid"},
    "parent_id": {"type": ["string", "null"], "format": "uuid"},
    "word_count": {"type": "integer", "minimum": 0}
  },
  "additionalProperties": false
}

```

---


## Epic 5: Map-Reduce Summarization & Orchestration

### Story 5.1 – Batch Summarization (“Map” Phase)

**As an** LLM-orchestrator **I want to** break a large set of Doc nodes into batches and generate concise micro-summaries for each batch **so that** I can stay within token limits.

**Test Cases:**

1. **Batch size limit -** When given 50 docs, the system only batches up to 10 at a time.
2. **Summary completeness -** Each micro-summary must mention at least two data points (e.g. quarter label + metric value).
3. **Token safety -** Generated prompts plus expected summary must not exceed 4,000 tokens.

### Story 5.2 – Final Aggregation (“Reduce” Phase)

**As an** LLM-orchestrator, **I want to** combine micro-summaries into a single narrative that addresses the user’s original question **so that** I deliver one coherent answer.

**Test Cases:**

1. **Cohesive narrative**
    - The final output mentions all banks/quarters covered and synthesizes trends.
2. **No data loss**
    - All KPIs from at least 80% of the micro-summaries must appear in the final aggregated answer.
3. **Follow-up readiness**
    - If the user asks “drill into BMO Q3,” the system can isolate that quarter’s micro-summary without re-running full batches.

---

## Epic 6: Verified Prompts 

### Story 6.1 – if three users did thumbs up, update the prompt and answer as verified, if three users did thumbs down save as incorrect asnwer  
### Story 6.2 - Use verification tag for new user queries for improving answer accuracy: 
Step 1 Fetch from verified results: Use RAG to fetch results from verified prompts before inference if verified answer exsist.  
Step 2: use the verification resulst in answer to provide confidence to user. 

-----

### Story 0.1 – Query Intent Classification & Temporal Constraint Extraction

**As an** LLM-orchestrator

**I want to** analyze each incoming user query, determine whether it’s **semantic** or **temporal**, and—if temporal—extract the explicit time range (years/quarters)

**so that** I can build precise metadata filters and propose the right retrieval strategy.

**Flow & Examples:**

1. **User Query:** “Show me the RWA trend over the past five quarters”
    - **Classification:** temporal
    - **Extracted Range:** last 5 quarters → `[Q1 2024, Q2 2024, Q3 2024, Q4 2024, Q1 2025]`
2. **User Query:** “what are mortgage‐backed securities ”
    - **Classification:** semantic
    - **No time range** extracted

**Test Cases:**

1. **Binary classification accuracy**
    - 100 labeled queries → ≥ 90% correct semantic vs. temporal.
2. **Temporal extraction accuracy**
    - “Q1 2020–Q4 2022” → range = `[2020 Q1, …, 2022 Q4]` exactly.
3. **Low-confidence fallback**
    - “Show me the history of capital ratios” → confidence < 60% → ask “Do you mean a trend over a specific period?”

---

### Story 0.2 – User Confirmation of Plan (Always Ask)

**As an** LLM-orchestrator

**I want to** present back both:

- the **detected intent** (semantic vs. temporal),
- and the **time range** if temporal,
    
    **so that** the user can confirm or correct before we run any retrieval.
    

**Flow & Examples:**

1. **After classification above**, system says:
    - **Temporal example:**
        
        > “I detected a temporal query for RWA, covering Q1 2024 through Q1 2025. Shall I fetch and trend that data?”
        > 
    - **Semantic example:**
        
        > “I detected a semantic query comparing mortgage-backed securities. Shall I run a top-5 RAG retrieval across all banks?”
        > 
2. **User Responses:**
    - **“Yes”** → proceed.
    - **“No, use Q2 2023–Q2 2024”** → update extracted range, replay classification+confirmation.
    - **No clear reply** after two prompts → ask explicitly:
        
        > “Would you like (1) a temporal trend or (2) a semantic comparison?”
        > 

**Test Cases:**

1. **Confirmation prompt for both paths**
    - Ensures both semantic and temporal queries always get a “Shall I proceed?” prompt.
2. **Respects corrections**
    - If user corrects the time range or intent, the system re-runs Story 0.1 with that guidance.
3. **Fallback on no reply**
    - Two non-answers → explicit “Trend or summary?” question.

---

###

-----

# Self Awareness Tool

---

## **Epic 4 – Self-Awareness & Dynamic Guidance Tool**

**Outcome**

Provide a single tool that can (a) list its own capabilities and limits and (b) answer “How do I…?” questions by consulting a 5 000-word JSON description of all procedures, training notes, and tool usage guides.

---

### **Story 4.1 – Greeting Prompt explaining capability**

**I want** the agent to read a static JSON file (`capability_manifest.json`, ≈ 5 000 words) at startup **so that** the Self-Awareness tool always knows its full abilities. and a greeting prompt is created based on that. 

**Acceptance**

1. File path set once via `SELF_AWARENESS_MANIFEST=/opt/docs/capability_manifest.json`.
2. On load failure, agent logs a single error and continues without the tool (no crash).
3. Hot-reload flag (`-reload-manifest`) re-reads the file without restart.

---

---

### **Story 4.2 – Implement `self_awareness(query: str)` Tool**

**As an** LLM-orchestrator

**I want** a LangChain tool `self_awareness(query)`

**so that** users (human or chain) can ask about capabilities or procedures.

**Acceptance**

1. Empty `query` → concise capability summary (≤ 400 words).
2. Non-empty `query` → RAG: similarity search (Story 4.2) → synthesize answer with citations.
3. Response always ≤ 4 096 tokens and returned in < 300 ms at P95 for queries < 512 chars.

-----

# SEC Filings – Graph Implementation Plan (v2)

> Scope: Stand-alone instructions to model, load, and query U.S./Canadian SEC filings (10-K, 8-K, 6-K, 40-F, etc.) in Neo4j using the RiskGPT metadata schema.
> 
> 
> The only changes from v1 are the **tightened uniqueness scopes, idempotent `MERGE` statements, and composite period indexes** flagged during the review.
> 

---

## 1 · Graph Hierarchy

```
(:Domain {name:'external'})
  └─[:HAS_SUBDOMAIN]─> (:Subdomain {name:'SEC', data_classification:'public'})
      └─[:HAS_COMPANY]─> (:Company {name:'RBC'})
          └─[:HAS_YEAR]─> (:Year {value:2025})
              └─[:HAS_QUARTER]─> (:Quarter {label:'Q1', year:2025, company:'RBC'})
                  └─[:HAS_DOC]─> (:Document {filename:'external_SEC_10K_RBC_2025_Q1_Financials'})
                      └─[:HAS_SECTION]─> (:Section {filename:'…', section:'Financials'})

```

### Node Labels & Mandatory Properties

| Label | Properties (mandatory bold) | Example |
| --- | --- | --- |
| **Domain** | **name** | `external` |
| **Subdomain** | **name**, **data_classification** | `SEC`, `public` |
| **Company** | **name** | `RBC`, `TD`, `BMO` |
| **Year** | **value** (int) | `2025` |
| **Quarter** | **label**, **year**, **company** | `Q1`, `2025`, `RBC` |
| **Document** | **filename**, document_type, context, year, quarter | `external_SEC_10K_RBC_2025_Q1_Financials` |
| **Section** | **filename**, **section** | `…`, `Financials` |

### Relationship Types

| Type | Start → End | Purpose |
| --- | --- | --- |
| `HAS_SUBDOMAIN` | Domain → Subdomain | External hierarchy |
| `HAS_COMPANY` | Subdomain → Company | Issuer grouping |
| `HAS_YEAR` | Company → Year | Calendar year |
| `HAS_QUARTER` | Year → Quarter | Quarter |
| `HAS_DOC` | Quarter → Document | Attaches filing to period |
| `HAS_SECTION` | Document → Section | Section decomposition |
| `RELATED_FILING` | Document ↔ Document | Links 10-K ↔ all 8-Ks for same company-period |

---

## 2 · One-Time DDL (constraints & indexes)

```
// ---------- Uniqueness constraints ----------

// simple keys
CREATE CONSTRAINT domain_name   IF NOT EXISTS ON (d:Domain)      ASSERT d.name IS UNIQUE;
CREATE CONSTRAINT sub_name      IF NOT EXISTS ON (s:Subdomain)   ASSERT s.name IS UNIQUE;
CREATE CONSTRAINT comp_name     IF NOT EXISTS ON (c:Company)     ASSERT c.name IS UNIQUE;
CREATE CONSTRAINT year_value    IF NOT EXISTS ON (y:Year)        ASSERT y.value IS UNIQUE;
CREATE CONSTRAINT doc_file      IF NOT EXISTS ON (d:Document)    ASSERT d.filename IS UNIQUE;

// composite key: one Q1 per company-year
CREATE CONSTRAINT quarter_scope IF NOT EXISTS
ON (q:Quarter) ASSERT (q.label, q.year, q.company) IS UNIQUE;

// composite key: unique section file+name per document
CREATE CONSTRAINT section_scope IF NOT EXISTS
ON (s:Section) ASSERT (s.filename, s.section) IS UNIQUE;

// ---------- Performance indexes ----------
CREATE INDEX doc_period_lookup IF NOT EXISTS
FOR (d:Document) ON (d.context, d.year, d.quarter, d.document_type);

```

---

## 3 · Parameterised Cypher Loader (per filing)

```
:param domain        => 'external';
:param subdomain     => 'SEC';
:param class         => 'public';
:param company       => $company;        // 'RBC'
:param year          => $year;           // 2025
:param quarter       => $quarter;        // 'Q1'
:param filename      => $filename;       // full file name
:param doctype       => $doctype;        // '10K', '8K', etc.
:param sections      => $sections;       // list of {filename, section}
:param source_uri    => $source_uri;
:param ingestion_dt  => $ingestion_date; // '2025-06-19'
:param checksum      => $checksum;

MERGE (dom:Domain {name:domain})
MERGE (sub:Subdomain {name:subdomain, data_classification:class})
MERGE (dom)-[:HAS_SUBDOMAIN]->(sub)

MERGE (co:Company {name:company})
MERGE (sub)-[:HAS_COMPANY]->(co)

MERGE (yr:Year {value:year})
MERGE (co)-[:HAS_YEAR]->(yr)

// quarter scoped by company+year
MERGE (qt:Quarter {label:quarter, year:year, company:company})
MERGE (yr)-[:HAS_QUARTER]->(qt)

// idempotent document load
MERGE (d:Document {filename:filename})
  ON CREATE SET
    d.document_type   = doctype,
    d.context         = company,
    d.year            = year,
    d.quarter         = quarter,
    d.domain          = domain,
    d.subdomain       = subdomain,
    d.data_classification = class,
    d.source_uri      = source_uri,
    d.ingestion_date  = date(ingestion_dt),
    d.checksum        = checksum;

MERGE (qt)-[:HAS_DOC]->(d)

// sections
UNWIND sections AS sec
  MERGE (s:Section {filename:sec.filename, section:sec.section})
  MERGE (d)-[:HAS_SECTION]->(s);

```

### Linking 10-K to all 8-Ks (same issuer & period)

```
MATCH (ten:Document {document_type:'10K', context:$company, year:$year, quarter:$quarter})
MATCH (eight:Document {document_type:'8K', context:$company, year:$year, quarter:$quarter})
MERGE (ten)-[:RELATED_FILING]->(eight);

```

---

## 4 · Batch Loader (Python outline)

```python
import glob, json
from neo4j import GraphDatabase

neo = GraphDatabase.driver(URI, auth=(USER, PASS))
LOAD_CYPHER = open('cypher/sec_loader.cypher').read()
LINK_CYPHER = open('cypher/sec_link.cypher').read()

def load(tx, filing):
    tx.run(LOAD_CYPHER, **filing)

def link(tx, co, yr, qt):
    tx.run(LINK_CYPHER, company=co, year=yr, quarter=qt)

with neo.session() as sess:
    for path in glob.glob('sec_json/*.json'):
        filing = json.load(open(path))
        sess.execute_write(load, filing)     # idempotent MERGE

    # link after all loads
    for co in ['RBC','TD','BMO']:
        for yr, qt in [(2025,'Q1'), (2024,'Q1')]:
            sess.execute_write(link, co, yr, qt)

```

---

## 5 · Execution Checklist

1. **Run DDL** once on a clean Neo4j instance.
2. **Convert each filing** to RiskGPT-JSON (`sections` included).
3. **Run batch loader** (update paths & creds).
4. **Spot-check counts**
    
    ```
    MATCH (d:Document) RETURN count(d);
    MATCH (:Document)-[:RELATED_FILING]->(:Document) RETURN count(*) AS links;
    
    ```
    
5. **Create vector index** (optional Neo4j v5.12+)
    
    ```
    CREATE VECTOR INDEX filingEmbeddings IF NOT EXISTS
    FOR (s:Section) ON (s.embedding) OPTIONS {dimensions:1536, similarityFunction:'cosine'};
    
    ```
    

---

---

### Done ✅

All mandatory fixes are now embedded. The graph is idempotent, quarter scope is safe for multi-issuer loads, and period queries run in index time. Let me know if you want code for **APOC batch imports** or **GDS similarity graphs** next.

---

