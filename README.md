# opensearch
opensearch and EKL 

=====================

Below is a **complete, clean, copy-friendly set of notes on Amazon OpenSearch**, followed by **real SDE-3 level interview questions with answers**.

---

# 📘 **Amazon OpenSearch – Complete Notes**

## ✅ **1. What is Amazon OpenSearch?**

Amazon OpenSearch is a **search, analytics, and observability service** based on the open-source OpenSearch (successor of Elasticsearch).
It is used for:

* Full-text search
* Log analytics
* Real-time application monitoring
* Distributed indexing & querying
* Security analytics
* Dashboarding (OpenSearch Dashboards)

Amazon OpenSearch Service = Fully managed version on AWS.

---

## ✅ **2. Core Components**

### **a. Cluster**

A collection of nodes that store data and process queries.

### **b. Nodes**

Different types:

* **Master nodes** – manage cluster state.
* **Data nodes** – store indexes, shards, process data.
* **Ingest nodes** – pipeline processing.
* **UltraWarm nodes** – inexpensive, warm storage.
* **Cold nodes** – lowest-cost storage for older data.
* **ML nodes** – run ML-based anomaly detection, KNN.

### **c. Index**

Logical namespace for documents (like a table in RDBMS).

### **d. Shards**

Data is split into:

* **Primary shards** – original partitions
* **Replica shards** – copies for fault tolerance & performance

---

## ✅ **3. How Data is Stored**

OpenSearch uses **inverted indexes**:

* Used for fast full-text search
* Maps terms → documents

Other structures:

* BKD trees for numeric & geo fields
* Doc values for sorting and aggregations
* Columnar storage for analytics

---

## ✅ **4. Querying: DSL (Domain-Specific Language)**

### **Query DSL Types**

1. **Match Queries**

   * Full-text search
2. **Term Queries**

   * Exact match
3. **Bool Queries**

   * must, must_not, should, filter
4. **Range Queries**
5. **Aggregations**

   * Metrics (sum, avg, min, max)
   * Buckets (terms, histogram)

Example:

```json
{
  "query": {
    "bool": {
      "must": { "match": { "title": "sports" }},
      "filter": { "range": { "price": { "gte": 100 } }}
    }
  }
}
```

---

## ✅ **5. OpenSearch Features**

### **a. Full-Text Search**

* Match
* Multi-match
* Phrase & prefix search
* Highlighting
* Relevance scoring (BM25)

### **b. Aggregations**

Equivalent to GROUP BY with analytics.

### **c. Observability**

* Log ingestion
* Trace analytics
* Dashboards

### **d. Security**

* Fine-grained access control
* IAM & SAML integration
* Encryption at rest & in transit
* Audit logs

### **e. Index Lifecycle Management (ILM)**

Automate index transitions:

* Hot → Warm → Cold → Delete

---

## ✅ **6. AWS-Specific Enhancements**

### **a. UltraWarm**

* Stores 90 days of logs at 70% cost reduction
* Uses S3 for storage, caching layers

### **b. Cold Storage**

* Even cheaper
* Queryable from S3 directly

### **c. Auto-Tune**

Automatically adjusts cluster resources & memory.

### **d. Serverless OpenSearch**

No cluster management
Pay-per-usage
Good for unpredictable traffic.

---

## ✅ **7. Performance Optimization Best Practices**

### **Indexing**

* Use bulk API for high throughput
* Avoid unnecessary nested fields
* Disable _source if not needed
* Tune refresh interval for ingest-heavy workloads

### **Querying**

* Use filters instead of match (filters are cached)
* Use keyword fields for exact match
* Use doc_values for sorting/aggregations
* Prefer multi-index over massive single index

### **Shards**

* Keep shard count small:

  ```
  ~20–40GB per shard is optimal
  ```
* Avoid large numbers of small shards
* Use shard splitting if needed

---

## ✅ **8. Typical Use Cases**

* Application search (e-commerce, apps)
* Log analytics (CloudWatch → OpenSearch)
* SIEM (Security Information Event Mgmt)
* Metrics & monitoring
* APM traces
* Real-time dashboarding
* Geo-spatial queries
* KNN vector similarity search (for ML embeddings)

---

# 🧠 **Amazon OpenSearch – Interview Questions & Answers (SDE-3)**

---

## **1. What is the difference between OpenSearch and Elasticsearch?**

### **Answer:**

* OpenSearch = AWS-backed open-source fork of Elasticsearch 7.10
* Elasticsearch moved to SSPL (non-open)
* OpenSearch is fully Apache 2.0 licensed
* OpenSearch Service integrates deeply with AWS:

  * UltraWarm, Cold storage
  * IAM integration
  * Serverless mode
  * AutoTune

---

## **2. What happens when you index a document?**

### **Answer:**

1. Document is analyzed (tokenized, normalized)
2. Stored in inverted index
3. Written to primary shard
4. Replicated to replica shard
5. Refresh happens (default 1s) to make document searchable
6. Commit happens periodically & flushed to disk

---

## **3. What is the difference between match and term queries?**

### **Answer:**

| Query     | Purpose                         |
| --------- | ------------------------------- |
| **match** | Full-text search, uses analyzer |
| **term**  | Exact search, no analysis       |

Example:

* `"match": {"title": "iphone 14"}`
* `"term": {"status": "DELIVERED"}`

---

## **4. How do you optimize indexing performance?**

### **Answer:**

* Use **Bulk API**
* Increase **refresh_interval**
* Disable **replicas** during bulk ingest
* Use **hot-warm architecture**
* Disable unnecessary features: `_all`, `_source`
* Use `routing` to reduce shard hops

---

## **5. How do you design a cluster for 500M log events/day?**

### **Answer Outline (SDE-3 level):**

* Partition data by time (daily indices)
* Use hot → warm architecture
* Hot nodes: SSD-backed, large heap
* Warm nodes: UltraWarm (cheaper)
* Use ILM to transition daily
* 6–12 shards per index (20–40GB each)
* Dedicated master nodes
* Bulk ingestion pipeline (Kinesis / Firehose / Logstash)
* Use filters & aggregations carefully

---

## **6. Explain shard allocation and rebalancing process.**

### **Answer:**

OpenSearch uses a master node that:

1. Tracks shard allocation in cluster state
2. Assigns primaries on index creation
3. Rebalances shards when:

   * New nodes join
   * Nodes fail
   * Replicas change
4. Uses awareness attributes (zone awareness)
5. Avoids hotspotting using shard balancing algorithms

---

## **7. How does OpenSearch achieve high availability?**

### **Answer:**

* Replica shards
* Zone aware cluster placement
* Dedicated master nodes
* Support for multiple AZ deployments
* Automatic failover & shard reassignments

---

## **8. What causes “shard explosion”? How to avoid it?**

### **Answer:**

**Shard Explosion = too many small shards → memory pressure**

Avoid by:

* Target 20–40GB per shard
* Avoid daily index creation with high replica count
* Use rollover API
* Use index templates
* Consolidate indices

---

## **9. What is KNN search in OpenSearch?**

### **Answer:**

* Vector similarity search for ML embeddings
* Uses HNSW, IVF algorithms
* Use cases:

  * Recommendation systems
  * Semantic search
  * Image/video similarity

---

## **10. How does OpenSearch Serverless work?**

### **Answer:**

* No node provisioning
* Auto-scaling compute
* Auto-tuning storage
* Pricing based on IndexingCU + QueryCU
* Great for spiky or unpredictable workloads

---

Below are **clean, copy-friendly, interview-ready notes** on **OpenSearch CRUD + Advanced Queries**, including examples and explanations.

---

# 📘 **OpenSearch CRUD + Advanced Queries – Complete Notes**

---

# ✅ **1. Basic CRUD in OpenSearch**

OpenSearch uses REST APIs for all operations.

---

## ### **1.1 CREATE (Index a Document)**

### **API**

```
PUT /index_name/_doc/1
```

### **Example**

```json
PUT products/_doc/1
{
  "name": "iPhone 15",
  "brand": "Apple",
  "price": 799,
  "in_stock": true
}
```

### Notes:

* If ID is omitted → auto-generated
* `PUT` = create/update
* `POST` = create with auto-id

---

## ### **1.2 READ (Get a Document)**

### **API**

```
GET /index_name/_doc/1
```

### **Example**

```json
GET products/_doc/1
```

---

## ### **1.3 UPDATE (Partial Update)**

### **API**

```
POST /index_name/_update/1
```

### **Example**

```json
POST products/_update/1
{
  "doc": {
    "price": 749
  }
}
```

### Scripted Update

```json
POST products/_update/1
{
  "script": {
    "source": "ctx._source.price += params.inc",
    "params": { "inc": 10 }
  }
}
```

---

## ### **1.4 DELETE Document**

### **API**

```
DELETE /index_name/_doc/1
```

---

## ### **1.5 BULK Operations (Very Important for Interviews)**

### **API**

```
POST _bulk
```

### **Example**

```
{ "index": { "_index": "products", "_id": 2 } }
{ "name": "Pixel 8", "brand": "Google", "price": 699 }

{ "update": { "_index": "products", "_id": 1 } }
{ "doc": { "in_stock": false } }

{ "delete": { "_index": "products", "_id": 3 } }
```

### Notes:

* Bulk API is heavily used in ingestion pipelines (Kinesis -> Lambda -> OpenSearch).
* MUST end with newline character.

---

# 🚀 **2. Advanced OpenSearch Querying (Interview-Focused)**

OpenSearch uses **Query DSL** (JSON-based query language).
Below are the most important patterns.

---

# ### **2.1 Full-text Search Queries**

## **Match Query**

```json
GET products/_search
{
  "query": {
    "match": {
      "name": "iphone case"
    }
  }
}
```

## **Multi-Match Query**

```json
{
  "query": {
    "multi_match": {
      "query": "iphone",
      "fields": ["name", "description"]
    }
  }
}
```

---

# ### **2.2 Term-Level Queries (Exact Match)**

## **Term Query**

```json
{
  "query": {
    "term": { "brand.keyword": "Apple" }
  }
}
```

**When to use keyword?**
For exact matches → use `field.keyword`

---

## **Range Query**

```json
{
  "query": {
    "range": {
      "price": { "gte": 500, "lte": 1000 }
    }
  }
}
```

---

# ### **2.3 Boolean Queries (must, should, must_not, filter)**

### **Most important query structure for interviews**

```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "iphone" }}
      ],
      "filter": [
        { "range": { "price": { "gte": 500 }}}
      ],
      "must_not": [
        { "term": { "in_stock": false }}
      ],
      "should": [
        { "match": { "brand": "Apple" }}
      ]
    }
  }
}
```

### Notes:

* `filter` does **not** affect scoring → FAST & CACHED
* `must` affects scoring

---

# ### **2.4 Aggregations (Very Important)**

Equivalent to SQL GROUP BY.

## **Terms Aggregation**

```json
{
  "aggs": {
    "brands": {
      "terms": { "field": "brand.keyword" }
    }
  }
}
```

## **Metrics Aggregation**

```json
{
  "aggs": {
    "avg_price": { "avg": { "field": "price" }},
    "max_price": { "max": { "field": "price" }}
  }
}
```

## **Bucket + Metric Aggregation**

```json
{
  "aggs": {
    "brand_prices": {
      "terms": { "field": "brand.keyword" },
      "aggs": {
        "avg_price": { "avg": { "field": "price" }}
      }
    }
  }
}
```

---

# ### **2.5 Sorting + Pagination**

## **Sort**

```json
{
  "sort": [
    { "price": "asc" }
  ]
}
```

## **Pagination: from + size**

```json
{
  "from": 20,
  "size": 10
}
```

### 🛑 NOTE

Using `from` with very large values → slow
Use **search_after** instead.

---

# ### **2.6 search_after (Deep Pagination)**

```json
{
  "size": 10,
  "query": { "match_all": {}},
  "search_after": [699, "Pixel 8"],
  "sort": [
    { "price": "asc" },
    { "name.keyword": "asc" }
  ]
}
```

---

# ### **2.7 Highlighting**

```json
{
  "query": {
    "match": { "description": "camera" }
  },
  "highlight": {
    "fields": { "description": {} }
  }
}
```

---

# ### **2.8 Nested Query**

For nested objects:

```json
{
  "query": {
    "nested": {
      "path": "specs",
      "query": {
        "bool": {
          "must": [
            { "match": { "specs.key": "camera" }},
            { "range": { "specs.value": { "gte": 40 }}}
          ]
        }
      }
    }
  }
}
```

---

# ### **2.9 Script Queries**

```json
{
  "query": {
    "script": {
      "script": {
        "source": "doc['price'].value * 0.9 < params.threshold",
        "params": { "threshold": 500 }
      }
    }
  }
}
```

---

# ### **2.10 KNN Vector Search (Advanced)**

For ML embeddings:

```json
{
  "query": {
    "knn": {
      "embedding_vector": {
        "vector": [0.1, 0.2, 0.3],
        "k": 5
      }
    }
  }
}
```

---

# 🎯 **3. Advanced Interview Questions + Answers**

---

## **Q1. Difference between match vs term query?**

**Term Query**

* Does NOT analyze input
* Exact match
* Use with `keyword` fields

**Match Query**

* Analyzed
* Tokenized and normalized
* Used for full-text search

---

## **Q2. filter vs must in bool query?**

| must                | filter                           |
| ------------------- | -------------------------------- |
| Affects scoring     | No scoring                       |
| Expensive           | Fast & cached                    |
| Use for text search | Use for exact/range/date queries |

---

## **Q3. How to optimize pagination?**

* Avoid large `from`
* Use **search_after**
* Use sort with unique tiebreak field
* Use index sorting if static data

---

## **Q4. How to optimize aggregations?**

* Use keyword fields
* Increase heap size
* Use smaller shard count
* Pre-aggregate using ingestion pipelines

---

## **Q5. What is the best way to ingest millions of records?**

* Bulk API
* 10–20MB batches
* 5–15 parallel threads
* Increase `refresh_interval`
* Set replicas = 0 during ingest
* Use dedicated ingest nodes

---

## **Q6. How to avoid shard explosion?**

* 20–40 GB per shard
* Avoid too many daily indices
* Use rollover API
* Use ILM

---

# Concepts of searching methods + how to implement them in OpenSearch (with Node.js examples)

Nice—I’ll cover the **concepts** you’ll be asked about in interviews (what each search does, tradeoffs), then show **how to implement** each in OpenSearch (mappings + query DSL) and give **Node.js** examples using the official OpenSearch JS client (`@opensearch-project/opensearch`). I’ll also show a small **vector (k-NN)** example (creation, indexing, search). Where useful I cite official docs.

---

# 1) Quick mapping from concept → when to use

* **Exact/term search** — look up exact values (IDs, status, tags). Fast, not analyzed.
* **Full-text (match)** — natural language, tokenized, scored (BM25). Use for user text.
* **Match_phrase** — phrase matching (order matters).
* **Prefix / Wildcard / Regex** — partial matches; wildcard can be slow.
* **Fuzzy / Damerau-Levenshtein** — tolerate typos/misspellings.
* **Boolean (bool)** — combine filters + scoring clauses. Filters are cached and fast.
* **Aggregations** — analytics / group-bys.
* **Nested** — query nested objects (arrays of objects) correctly.
* **Geo** — distance / bounding-box / geo-shape queries.
* **Vector / k-NN / Semantic** — similarity on embeddings (semantic search / recommendations). Use specialized vector field and k-NN search. ([OpenSearch Documentation][1])

---

# 2) Setup: Node client + helper functions

Install client:

```bash
npm install @opensearch-project/opensearch
```

Minimal client init:

```js
// opensearch-client.js
const { Client } = require('@opensearch-project/opensearch');

const client = new Client({
  node: process.env.OPENSEARCH_URL || 'http://localhost:9200',
  auth: {
    username: process.env.OS_USER || 'admin',
    password: process.env.OS_PASS || 'admin'
  }
});

module.exports = client;
```

(Official OpenSearch JS client docs and examples: docs & GitHub). ([OpenSearch Documentation][2])

---

# 3) Example index mapping (text, keyword, nested, geo, dense_vector)

Below mapping includes a `dense_vector` for vector / semantic search (k-NN). Exact `method` params may change by OpenSearch version; OpenSearch supports HNSW / k-NN plugin and index-level `index.knn` setting for approximate search. ([OpenSearch Documentation][3])

```json
PUT products
{
  "settings": {
    "index": {
      "number_of_shards": 1,
      "number_of_replicas": 0,
      "knn": true                   // required on some OpenSearch distributions for knn indexing
    }
  },
  "mappings": {
    "properties": {
      "name": { "type": "text", "fields": { "keyword": { "type": "keyword" } } },
      "brand": { "type": "keyword" },
      "price": { "type": "float" },
      "in_stock": { "type": "boolean" },
      "tags": { "type": "keyword" },
      "location": { "type": "geo_point" },
      "specs": {
        "type": "nested",
        "properties": {
          "k": { "type": "keyword" },
          "v": { "type": "text" }
        }
      },
      "embedding": {
        "type": "dense_vector",
        "dims": 3,
        "index": true,
        "method": {
          "name": "hnsw",
          "space_type": "cosinesimil",
          "engine": "nmslib",
          "parameters": { "ef_construction": 128, "m": 16 }
        }
      }
    }
  }
}
```

> Note: If using AWS OpenSearch Service / Serverless vectors, there are additional APIs (collections) and slightly different configuration. For cluster OpenSearch, k-NN mapping like above is commonly used. ([AWS Documentation][4])

---

# 4) CRUD + Bulk (Node.js)

```js
const client = require('./opensearch-client');

async function indexDoc(id, doc) {
  return client.index({
    index: 'products',
    id,
    body: doc,
    refresh: 'wait_for'   // for demo; in production use async refresh
  });
}

async function bulkIndex(docs) {
  const body = docs.flatMap(d => [{ index: { _index: 'products', _id: d.id } }, d.body]);
  return client.bulk({ refresh: true, body });
}
```

---

# 5) Query examples (DSL + Node.js calls)

I’ll show the DSL and then the Node.js call for each methodology.

---

## 5.1 Exact / Term query

**Use case:** lookup status, brand, IDs.
DSL:

```json
{ "query": { "term": { "brand": "Apple" } } }
```

Node:

```js
await client.search({ index: 'products', body: { query: { term: { brand: 'Apple' } } } });
```

---

## 5.2 Full-text / match

**Use case:** user search box over text.
DSL:

```json
{ "query": { "match": { "name": "wireless headphones" } } }
```

Node:

```js
await client.search({ index: 'products', body: { query: { match: { name: 'wireless headphones' } } }});
```

---

## 5.3 Phrase match

**Use case:** exact phrase, ordering matters.
DSL:

```json
{ "query": { "match_phrase": { "name": "wireless noise cancelling" } } }
```

---

## 5.4 Prefix / Wildcard / Regexp

**Prefix** (fast if keyword or analyzed `edge_ngram` used):

```json
{ "query": { "prefix": { "name.keyword": "iph" } } }
```

**Wildcard** (slower; avoid leading `*`):

```json
{ "query": { "wildcard": { "name.keyword": "iPhone*" } } }
```

**Regex**:

```json
{ "query": { "regexp": { "sku": "prod[0-9]+" } } }
```

---

## 5.5 Fuzzy search (typo-tolerance)

**Concept:** uses Damerau–Levenshtein edit distance; good for user typos. Be careful: high `fuzziness` + long text can be expensive. ([OpenSearch Documentation][1])

DSL — `match` with `fuzziness`:

```json
{
  "query": {
    "match": {
      "name": {
        "query": "iphon 15",
        "fuzziness": "AUTO"   // or 1/2
      }
    }
  }
}
```

Or `fuzzy` query:

```json
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "iphon",
        "fuzziness": 2,
        "max_expansions": 50
      }
    }
  }
}
```

Node:

```js
await client.search({
  index: 'products',
  body: {
    query: {
      match: { name: { query: 'iphon 15', fuzziness: 'AUTO' } }
    }
  }
});
```

**Interview note:** explain tradeoff: fuzzy helps UX but can produce more matches and CPU work — prefer when necessary and limit `max_expansions` or use phonetic analyzers if appropriate.

---

## 5.6 Boolean queries / filters (practical & performant)

**Pattern:** `filter` for non-scoring constraints (fast & cached), `must` for scoring.
DSL:

```json
{
  "query": {
    "bool": {
      "must": [{ "match": { "name": "headphones" } }],
      "filter": [
        { "term": { "brand": "Sony" }},
        { "range": { "price": { "lte": 300 } } }
      ],
      "should": [{ "match": { "tags": "wireless" }}],
      "minimum_should_match": 0
    }
  }
}
```

Node:

```js
await client.search({ index: 'products', body: {/* above body */} });
```

---

## 5.7 Aggregations (facets)

**Example:** brands + average price.
DSL:

```json
{
  "size": 0,
  "aggs": {
    "brands": {
      "terms": { "field": "brand", "size": 10 },
      "aggs": {
        "avg_price": { "avg": { "field": "price" } }
      }
    }
  }
}
```

---

## 5.8 Nested queries

**Use case:** arrays of objects where each object must be matched as a unit.
DSL:

```json
{
  "query": {
    "nested": {
      "path": "specs",
      "query": {
        "bool": {
          "must": [
            { "term": { "specs.k": "ram" } },
            { "match": { "specs.v": "8GB" } }
          ]
        }
      }
    }
  }
}
```

---

## 5.9 Geo queries

**Distance query**:

```json
{
  "query": {
    "bool": {
      "filter": {
        "geo_distance": {
          "distance": "10km",
          "location": { "lat": 12.9716, "lon": 77.5946 }
        }
      }
    }
  }
}
```

---

## 5.10 Vector (k-NN) search — semantic / embedding-based

**Concept:** convert texts/images to numeric vectors (embeddings) and find nearest vectors in vector space (cosine / l2). Use when you want semantic similarity rather than lexical match. k-NN in OpenSearch is backed by HNSW or Faiss engines (approximate search). ([OpenSearch Documentation][5])

### Index mapping recap

(see mapping above — `embedding` as `dense_vector` with `method` and `knn=true` settings).

### Indexing documents (embedding precomputed)

```js
// example doc w/ small 3-dim embedding for demo
await client.index({
  index: 'products',
  id: 'p1',
  body: {
    name: 'Noise cancelling headphones',
    brand: 'Acme',
    price: 199,
    embedding: [0.12, 0.54, -0.33]
  },
  refresh: 'wait_for'
});
```

### k-NN query DSL (approximate nearest neighbors)

```json
{
  "size": 5,
  "query": {
    "knn": {
      "embedding": {
        "vector": [0.13, 0.52, -0.30],
        "k": 5
      }
    }
  }
}
```

Node:

```js
const resp = await client.search({
  index: 'products',
  body: {
    size: 5,
    query: {
      knn: {
        embedding: {
          vector: [0.13, 0.52, -0.30],
          k: 5
        }
      }
    }
  }
});
console.log(resp.body.hits.hits);
```

> Notes:
>
> * For real models use 128–1536 dimension vectors (BERT, OpenAI, etc.). Use appropriate index settings and memory.
> * k-NN can be run with filters (combine lexical + vector: hybrid search). Example: filter by `brand` THEN run k-NN to restrict candidates. ([OpenSearch Documentation][6])

---

# 6) Hybrid search (vector + lexical)

Combine vector similarity score with lexical score using `bool` + `should` or `script_score`:

### simple hybrid using `bool` (vector in `should` + textual match)

```json
{
  "query": {
    "bool": {
      "should": [
        { "match": { "name": "noise cancelling headphones" } },
        {
          "knn": { "embedding": { "vector": [0.13,0.52,-0.30], "k": 10 } }
        }
      ]
    }
  }
}
```

Or use `script_score` if you want to combine scores mathematically (careful with performance).

---

# 7) Performance / interview tips (talking points)

* **Filters vs must**: use `filter` for non-scoring conditions (cached & faster).
* **Bulk indexing**: use bulk API and tune `refresh_interval`. Disable replicas during bulk ingest for speed.
* **Shard sizing**: avoid too many small shards (aim ~20–40GB per shard typical guidance).
* **Fuzzy caution**: fuzziness creates term expansions → CPU heavy; consider suggesters or phonetic analyzers. ([Opster][7])
* **Vector tradeoffs**: vectors require memory for graph structures (HNSW), tune `m` and `ef_construction`. Consider approximate vs exact search tradeoffs. ([OpenSearch Documentation][8])
* **Use index templates and ILM** for time-series logs.

---

# 8) Short interview-style Q&A (you can recite these)

Q: *When would you use fuzzy vs prefix vs n-gram?*
A: Use **fuzzy** for typo tolerance (Levenshtein edits); **prefix** for autocomplete from the start of the token; **n-gram/edge_ngram** during indexing for efficient partial-match/autocomplete across token boundaries.

Q: *How to combine vector & keyword constraints?*
A: Use `bool` with a `filter` for keywords (fast) and a `knn` clause (or `script_score`) for vector similarity; combine elegantly with `should`/score blending or custom `script_score`.

Q: *What is the `dense_vector` type used for?*
A: Stores fixed-dimension numeric vectors used by k-NN queries for similarity search; often combined with HNSW/FAISS indexer for approximate nearest neighbors. ([OpenSearch Documentation][3])

---

# 9) Full working Node.js example (create index, index docs, run fuzzy & knn)

```js
// example-run.js
const client = require('./opensearch-client');

async function run() {
  // 1) create index (if not exists) - truncated; assume mapping exists as earlier
  // 2) bulk index
  await client.bulk({
    refresh: 'wait_for',
    body: [
      { index: { _index: 'products', _id: '1' } },
      { name: 'iPhone 15', brand: 'Apple', price: 799, embedding: [0.1,0.2,0.3] },
      { index: { _index: 'products', _id: '2' } },
      { name: 'iPhon 15 case', brand: 'Acme', price: 19, embedding: [0.12,0.22,0.29] },
      { index: { _index: 'products', _id: '3' } },
      { name: 'Noise cancelling headphones', brand: 'Acme', price: 199, embedding: [0.2,0.1,-0.1] }
    ]
  });

  // 3) fuzzy search
  const fuzzyResp = await client.search({
    index: 'products',
    body: {
      query: {
        match: { name: { query: 'iphon', fuzziness: 'AUTO' } }
      }
    }
  });
  console.log('Fuzzy hits:', fuzzyResp.body.hits.hits.map(h => h._source.name));

  // 4) k-NN search
  const knnResp = await client.search({
    index: 'products',
    body: {
      size: 3,
      query: {
        knn: {
          embedding: { vector: [0.11, 0.21, 0.29], k: 3 }
        }
      }
    }
  });
  console.log('kNN hits:', knnResp.body.hits.hits.map(h => ({ id: h._id, score: h._score, name: h._source.name })));
}

run().catch(console.error);
```

---

# 10) References & further reading

* OpenSearch k-NN / dense_vector docs (k-NN query, mapping, HNSW). ([OpenSearch Documentation][5])
* OpenSearch JS client docs & examples. ([OpenSearch Documentation][2])
* Fuzzy query details (Damerau-Levenshtein). ([OpenSearch Documentation][1])
* AWS OpenSearch k-NN guide & serverless vector collections (if you use AWS-managed service). ([AWS Documentation][4])

---
