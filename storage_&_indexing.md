## Storage & Indexing

Storage and indexing determine how efficiently a RAG system stores knowledge and retrieves relevant information at query time. A production-grade RAG system rarely relies on a single database. Instead, it uses multiple storage layers, each optimized for a different purpose: semantic search, metadata lookup, raw content storage, and transactional application data.

The overall objective is to:

- Retrieve relevant information quickly
- Support semantic and keyword search
- Maintain traceability to source documents
- Enable scalable updates as knowledge changes
- Keep latency low while preserving retrieval quality

A common architecture looks like:

```text
Raw documents
    ↓
Chunking + metadata extraction
    ↓
Embedding generation
    ↓
Storage layers
 ├─ Vector DB (semantic retrieval)
 ├─ Document store (chunk text + metadata)
 ├─ Object store (raw source files)
 └─ Relational DB (application/business state)
```

### Why multiple storage systems?

A vector database alone is usually not enough.

For example:

A vector DB may store embeddings for similarity search, but it is inefficient for storing large PDFs, document versions, permissions, user interactions, or relational business data.

Therefore, production systems separate concerns:

- **Vector DB** → semantic similarity search
- **Document store** → chunk text + metadata retrieval
- **Object store** → original files
- **Relational DB** → application-level state

---

## 1. Vector Database

A vector database stores embeddings and enables efficient similarity search.

In RAG, every chunk is converted into a high-dimensional vector using an embedding model. The system later compares the query vector against stored vectors to retrieve semantically relevant chunks.

Example:

```text
Chunk:
"Employees can work remotely after six months"

Embedding:
[0.12, -0.41, 0.72, ...]

User query:
"When can employees work from home?"

→ Similarity search finds nearby vectors
```

The vector DB supports:

- Approximate nearest-neighbor (ANN) search
- Metadata filtering
- Fast retrieval at scale
- Similarity ranking

Popular vector databases include:

- Pinecone
- Weaviate
- Milvus
- Qdrant
- Chroma (commonly for local/smaller systems)
- pgvector (Postgres extension)

### Why not brute-force search?

Naively comparing a query vector against millions of embeddings is expensive.

Example:

```text
1 query × 10 million vectors
```

Performing cosine similarity against every vector becomes computationally slow.

Instead, systems use ANN indexes to trade a tiny amount of accuracy for dramatically faster search.

This improves latency while preserving high retrieval quality.

---

## 2. ANN (Approximate Nearest Neighbor) Indexing

ANN indexing is the core optimization that makes semantic retrieval scalable.

Instead of comparing against every vector, ANN structures organize vectors to efficiently search nearby regions.

Tradeoff:

```text
Slightly lower recall
        ↓
Much faster retrieval
```

Typical production systems accept this tradeoff.

### HNSW (Hierarchical Navigable Small World)

HNSW is one of the most widely used ANN indexing algorithms.

It organizes vectors into a graph where similar vectors are connected. During search, traversal starts broadly and progressively narrows toward the nearest neighbors.

Think of it like:

```text
Large map
   ↓
Navigate highways
   ↓
Move to local roads
   ↓
Reach destination
```

Advantages:

- Very fast retrieval
- High recall quality
- Excellent for real-time RAG systems

Tradeoffs:

- Higher memory usage
- Slower index construction

Used in:

- Qdrant
- Weaviate
- pgvector
- Milvus

---

### IVF (Inverted File Index)

IVF partitions vectors into clusters.

At query time:

1. Find nearest cluster(s)
2. Search only inside those clusters

Instead of:

```text
Search 10M vectors
```

You do:

```text
Search nearest cluster
→ maybe 50K vectors
```

Advantages:

- Faster for very large datasets
- Lower memory usage

Tradeoffs:

- Slightly worse recall than HNSW
- Requires tuning cluster count

Best for:

- Massive-scale retrieval systems

---

### PQ (Product Quantization)

PQ compresses vectors to reduce storage and speed up retrieval.

Instead of storing exact high-dimensional vectors, it stores compressed approximations.

Benefit:

```text
Lower RAM usage
```

Useful when storing billions of vectors.

Tradeoff:

```text
Higher compression
→ lower precision
```

Often combined with IVF:

```text
IVF + PQ
```

for large-scale systems.

---

### DiskANN

DiskANN is optimized for retrieval from SSD storage instead of RAM.

Useful when vector collections become too large to fit in memory.

Advantages:

- Cost-efficient scaling
- Handles very large datasets

Tradeoffs:

- Slightly higher latency than RAM-based retrieval

Good for:

- Enterprise-scale retrieval systems

---

### ScaNN

ScaNN is a retrieval library developed by Google for high-performance vector search.

Strengths:

- Optimized ANN retrieval
- High throughput
- Efficient vector compression

Often used in:

- Research systems
- Large recommendation/search systems

---

## 3. Sparse Indexes (Keyword Retrieval)

Dense embeddings are semantic but sometimes miss exact keywords.

Example:

Query:

```text
Error code 0x80070005
```

Semantic search may fail because exact token matching matters.

Sparse retrieval solves this.

The most common method is BM25.

### BM25

BM25 ranks documents based on keyword relevance.

It considers:

- Term frequency
- Document frequency
- Token importance

Example:

Query:

```text
"refund policy for enterprise customers"
```

BM25 favors chunks containing:

```text
refund
enterprise
customers
policy
```

Advantages:

- Precise keyword matching
- Good for IDs, product names, legal clauses, error codes

Weakness:

- Poor semantic understanding

Example:

```text
"vacation leave"
```

may not match

```text
"paid time off"
```

because wording differs.

---

## 4. Hybrid Retrieval (Dense + Sparse)

Modern RAG systems commonly use hybrid retrieval.

Instead of choosing semantic OR keyword retrieval:

```text
Dense retrieval
      +
Sparse retrieval
```

Why?

Because they complement each other.

Example:

Query:

```text
How do I reset password for account 8294?
```

Dense retrieval:

- understands meaning of "reset password"

Sparse retrieval:

- matches exact account ID

Together:

```text
Better precision + better recall
```

Typical ranking pipeline:

```text
User query
    ↓
Dense retrieval (top K)
    +
Sparse retrieval (top K)
    ↓
Merge candidates
    ↓
Reranking
    ↓
Final chunks
```

This is often considered the production default.

---

## 5. Metadata Indexing

Metadata is indexed to improve retrieval quality and filtering.

Examples:

```text
source = HR_policy.pdf
department = HR
timestamp = 2025
country = India
access_level = internal
```

Metadata helps:

### Filtering

Example:

```text
Only search HR documents
```

or

```text
timestamp > Jan 2025
```

### Ranking

Example:

Prefer:

- newer documents
- trusted sources
- official policies

### Access control

Example:

Employees should only retrieve documents they are allowed to access.

Without permission filtering:

```text
Sensitive internal documents
could leak into prompts
```

Metadata filtering is therefore critical in enterprise RAG.

---

## 6. Document Store

The vector DB often stores only embeddings and lightweight metadata.

Actual chunk text is commonly stored separately.

A document store keeps:

- chunk content
- metadata
- citations
- source references
- version history

Examples:

- Elasticsearch
- MongoDB
- PostgreSQL
- OpenSearch

Flow:

```text
Retrieve vector IDs
      ↓
Fetch chunk text
      ↓
Send to LLM
```

This separation improves maintainability and flexibility.

---

## 7. Object Storage

Raw files are stored in object storage.

Examples:

- PDFs
- Images
- Videos
- Spreadsheets
- Original documents

Common systems:

- Amazon S3
- Google Cloud Storage
- Azure Blob Storage

Why store originals?

For:

- citations
- reprocessing
- debugging
- document re-indexing
- compliance

Example:

```text
HR_policy_v3.pdf
```

may later be re-chunked using a better strategy.

---

## 8. Incremental Updates & Reindexing

Knowledge changes frequently.

A production RAG system must support updates without rebuilding everything.

Example:

Policy updated:

```text
Remote work eligibility:
6 months → 3 months
```

Pipeline:

```text
Detect document change
        ↓
Re-chunk changed sections
        ↓
Generate new embeddings
        ↓
Update index
```

Challenges:

- stale embeddings
- duplicate chunks
- inconsistent metadata
- index drift

Common strategies:

- versioning
- incremental indexing
- scheduled reindexing
- event-driven pipelines

---

## 9. Sharding & Scaling

As data grows, a single machine becomes insufficient.

Sharding distributes data across nodes.

Example:

```text
Shard 1 → HR docs
Shard 2 → Finance docs
Shard 3 → Engineering docs
```

Benefits:

- parallel search
- higher throughput
- horizontal scaling

Challenges:

- balancing shards
- query routing
- consistency

---

## 10. Storage Design Tradeoffs

Question:

**"How would you design storage for a production RAG system?"**

Example answer:

> I would separate concerns using multiple storage layers. A vector database would handle semantic retrieval through ANN indexes like HNSW, a document store would manage chunk text and metadata, object storage would preserve raw files, and relational databases would store application state. For retrieval quality, I’d combine dense and sparse indexes using hybrid search and use metadata filtering for permissions, freshness, and source trust.

Key tradeoffs:

| Decision         |                   Benefit |               Tradeoff |
| ---------------- | ------------------------: | ---------------------: |
| HNSW             | High recall + low latency |            High memory |
| IVF              |             Scales better |  Slightly lower recall |
| PQ compression   |        Lower storage cost |      Reduced precision |
| Dense retrieval  |    Semantic understanding | Weak keyword precision |
| Sparse retrieval |            Exact matching |         Weak semantics |
| Hybrid search    |  Better retrieval quality | More system complexity |

### Simple mental model

Think of storage and indexing as:

```text
Object store → raw truth
Document store → readable chunks
Vector DB → semantic lookup
Metadata index → filtering & trust
Relational DB → app/business logic
```

Together, they enable scalable, low-latency, and reliable retrieval for RAG systems.
