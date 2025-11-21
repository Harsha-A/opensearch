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
