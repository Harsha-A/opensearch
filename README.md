# opensearch
opensearch and EKL 

=====================

## Amazon OpenSearch – Quick‑Study Notes

| Section | Key Points | Practical Tips |
|--------|------------|----------------|
| **What is OpenSearch?** | • Fork of Elasticsearch 7.10 (Apache 2.0) <br>• Fully managed “OpenSearch Service” on AWS <br>• Search, analytics, observability in one stack | • Use **OpenSearch Dashboards** for visualising logs & metrics <br>• Take advantage of AWS‑only features (UltraWarm, Cold, Serverless, Auto‑Tune) |
| **Core Architecture** | • **Cluster** – one logical entity of nodes <br>• **Nodes**: <br> - **Master** – cluster state <br> - **Data** – index shards <br> - **Ingest** – pipeline <br> - **UltraWarm/Cold** – cost‑effective, long‑term storage <br> - **ML** – KNN, anomaly <br>• **Index** – logical namespace (like a table) <br>• **Shards** – primary & replica | • Keep shard size ~20–40 GB for best memory/performance <br>• Use **dedicated master nodes** (≥3, zone‑aware) for stability |
| **Data Storage** | • **Inverted index** – fast full‑text search (term → doc) <br>• **BKD trees** – numeric & geo <br>• **Doc values** – column‑arised storage for sorting & aggregations | • Disable `_source` if you only need fields you’ll query <br>• Use **keyword** sub‑field for exact match |
| **Query DSL (Examples)** | • `match` – full‑text, analyzer used <br>• `term` – exact, no analyzer <br>• `bool` – combine `must`, `filter`, `must_not`, `should` <br>• `range` – numeric/date <br>• **Aggregations** – `terms`, `histogram`, `sum`, `avg`, `min`, `max` | ```json\n{ "query": { "bool": { "must": { "match": { "title": "sports" } }, "filter": { "range": { "price": { "gte": 100 } } } } } }\n``` |
| **Observability** | • Log ingestion (CloudWatch → OpenSearch) <br>• Trace analytics & APM <br>• Dashboards for real‑time monitoring | • Create **index templates** with ILM policy: `hot → warm → cold → delete` |
| **Security** | • Fine‑grained role‑based access (Field‑Level) <br>• IAM & SAML integration <br>• Encryption at rest (KMS) & in transit (TLS) <br>• Audit logs | • Use **SAML** for single‑sign‑on in corporate environments |
| **AWS‑Specific Enhancements** | • **UltraWarm** – 70 % cheaper than hot, uses S3 caching <br>• **Cold Storage** – cheapest, queryable from S3 <br>• **Auto‑Tune** – auto‑adjust memory & thread pools <br>• **Serverless** – no cluster, pay for CU usage | • For large log volumes, design **hot‑warm** or **hot‑warm‑cold** lifecycle <br>• Enable **Serverless** for spiky traffic |
| **Performance Optimisation** | • **Bulk API** for ingestion <br>• Increase `refresh_interval` (default 1 s) for heavy writes <br>• Disable replicas temporarily during bulk <br>• Prefer `filter` over `match` (filters cache) <br>• Use **keyword** for exact queries | • Monitor **search‑throttle** and **search‑slow‑log** <br>• Keep shard count modest (≤ 12 per index for 500M logs/day) |
| **Shard Management** | • Primary shards assigned at index creation <br>• Replica shards auto‑rebalanced on node changes <br>• Use **awareness attributes** (availability zone) to avoid hotspotting | • Avoid “shard explosion”: <br> - Target 20–40 GB per shard <br> - Use rollover or index‑templates <br> - Consolidate indices |
| **High Availability** | • Replicas + zone‑aware placement <br>• Dedicated master nodes <br>• Multi‑AZ clusters <br>• Automatic failover and shard reassignment | • Regularly run **cluster health** checks and watch for `yellow` status |
| **KNN / Vector Search** | • Stores high‑dimensional embeddings <br>• Uses HNSW, IVF algorithms <br>• Applications: recommendation, semantic search, image similarity | • Create KNN index with `m` and `ef_search` parameters for tuning <br>• Use **OpenSearch Python client** (`knn_vector`) for inserts/queries |
| **Serverless OpenSearch** | • No cluster provisioning <br>• Auto‑scales compute & storage <br>• Pricing: `IndexingCU` + `QueryCU` | • Ideal for unpredictable or bursty workloads (e.g., log spikes) |
| **Interview‑Ready Questions** | 1. Difference between OpenSearch & Elasticsearch (license, AWS features). <br>2. Document life‑cycle (analyze → index → replica → refresh). <br>3. Match vs term query. <br>4. Indexing optimisation techniques. <br>5. Cluster design for 500 M logs/day (hot‑warm, ILM, shard sizing). <br>6. Shard allocation & rebalancing. <br>7. HA mechanisms. <br>8. Shard explosion causes & prevention. <br>9. KNN search purpose & algorithms. <br>10. Serverless operation model. | • Prepare concise answers and code snippets. |

---

### Quick‑Start Checklist
1. **Create Service** → `OpenSearch Service` → choose version 2.x (OpenSearch)  
2. **Define Index Template** → mappings + ILM policy  
3. **Enable Security** → IAM policy + fine‑grained role  
4. **Setup Data Ingestion** → Kinesis → OpenSearch or Logstash  
5. **Monitor** → CloudWatch metrics + OpenSearch Dashboards  
6. **Tune** → bulk settings, refresh interval, shard count  
7. **Back‑Up** → snapshots to S3  

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


Nice question — there are actually **two different things people usually mean when they say “DFS / BFS in OpenSearch”**. Below I’ll explain both meanings, when each is used, how they affect search behavior, and give practical **Node.js** examples you can use in interviews.

# 1) **DFS = “Distributed Frequency Search” (search_type)**

This is *not* the graph algorithm — here DFS stands for **Distributed Frequency Search** (term-frequency aggregation across shards). It’s an OpenSearch/Elasticsearch search mode used to get *globally consistent relevance scores* (IDF) across a distributed index.

What it does and why it matters

* By default OpenSearch uses **query_then_fetch**: each shard scores results using *local* term/document frequencies; those local scores are merged — fast but can give slightly inconsistent ranking across shards.
* `dfs_query_then_fetch` first performs a tiny pre-query to collect global term and document frequencies from every shard, computes global IDF, and then executes the real query using those global stats. That yields **more accurate, consistent scoring** across shards at the cost of an extra round trip and higher latency. ([OpenSearch Documentation][1])

When to use

* Use when correct relative scoring across shards is important (e.g., small-shard skew causes unexpected ranking differences). Avoid for latency-sensitive requests.

Node.js example (set `search_type` to `dfs_query_then_fetch`):

```js
const { Client } = require('@opensearch-project/opensearch');
const client = new Client({ node: 'http://localhost:9200' });

async function searchWithDfs() {
  const resp = await client.search({
    index: 'products',
    search_type: 'dfs_query_then_fetch',   // <-- uses global IDF
    body: {
      query: {
        match: { description: 'wireless headphones' }
      },
      size: 10
    }
  });
  return resp.body.hits.hits;
}
```

(Interview note: mention the tradeoff — more accurate scoring vs extra latency). ([OpenSearch Documentation][1])

---

# 2) **BFS / DFS as graph traversal — where they appear in OpenSearch (vector search / graph features)**

Short answer: OpenSearch itself doesn’t run a plain textbook BFS/DFS for vector search, but graph-traversal ideas *are* central in these subsystems:

### a) **HNSW (k-NN) vector search — graph traversal style**

* HNSW builds a **hierarchical graph** of vectors. Searching is done by *greedy graph routing + beam/neighbor exploration* — you start at an entry point in the top layer, greedily move towards closer neighbors, descend layers, then run a beam-like search at layer-0 (examining a candidate queue of size `ef_search`). This is **neither pure BFS nor pure DFS** — it’s a specialized greedy/beam graph search designed for fast approximate nearest neighbor retrieval. Key hyperparameters are `M` (edges per node), `ef_construction`, and `ef_search` (controls candidate queue size during search). Increasing `ef_search` raises recall at the cost of latency. ([Pinecone][2])

* In OpenSearch you create `knn`/`knn_vector` fields (HNSW / IVF / other engines) and then use the `knn` query. You can pass search-time parameters like `method_parameters.ef_search` (or engine-specific equivalents) to trade recall vs latency. ([OpenSearch Documentation][3])

Node.js k-NN example (set `method_parameters` on the query):

```js
// assume client is the OpenSearch client
const knnResp = await client.search({
  index: 'products',
  body: {
    size: 5,
    query: {
      knn: {
        embedding: {
          vector: [0.13, 0.52, -0.30],
          k: 5,
          method_parameters: { ef_search: 128 }   // increase candidate queue for better recall
        }
      }
    }
  }
});
console.log(knnResp.body.hits.hits);
```

(Interview note: explain `M`, `ef_construction`, `ef_search` tradeoffs — memory/index time/recall/latency). ([Amazon Web Services, Inc.][4])

### b) **Graph traversal / exploration features**

* If you integrate OpenSearch with a graph DB or use graph-analytics connectors, classic **BFS / DFS** are used conceptually for relationship traversal (e.g., expand neighbors breadth-first to find shortest-path-like results, or depth-first for deep path discovery). OpenSearch itself has connectors/plugins to embed search into graph workflows rather than providing a full native graph DB traversal engine. Cite/mention if you’ve used GraphDB connector or built graph features on top. ([GraphDB][5])

---

# 3) How to explain this in an interview (short talking points)

* **Be explicit** which “DFS” you mean: *Distributed Frequency Search* vs *Depth-First Search*. Employers will like that clarification.
* For DFS (Distributed Frequency Search): describe that it computes global IDF via a small pre-query and show the `search_type` example; explain the latency vs accuracy tradeoff. ([OpenSearch Documentation][1])
* For BFS/DFS (graph traversal): explain that vector indices (HNSW) are graph structures and searches are greedy/beam traversals (not classic BFS/DFS), and mention `M`, `ef_construction`, `ef_search` and how they affect recall/latency. Give the k-NN example and mention hybrid search possibilities (filters + knn). ([Pinecone][2])
* If asked for implementation detail: show the `search_type` snippet and the `knn` query snippet (above) — that demonstrates both meanings practically.

---
