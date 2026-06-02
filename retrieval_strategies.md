## Retrieval Strategies

Retrieval strategy determines **how the system finds relevant information** from stored knowledge before the LLM generates a response. This stage has a major impact on answer quality because poor retrieval leads to irrelevant or incomplete context.

The objective is to maximize:

- **Recall** → retrieve all relevant information.
- **Precision** → avoid irrelevant chunks.
- **Latency** → retrieve quickly at scale.
- **Context quality** → deliver useful evidence to the LLM.

A good retrieval strategy balances all four.

---

## Why Retrieval Strategy Matters

Retrieval is the bridge between user questions and stored knowledge.

Example:

Knowledge base contains:

```text
HR_policy.pdf

"Employees are eligible for remote work after six months."
```

User asks:

> “When can I work remotely?”

If retrieval only relies on exact keywords, it may fail because the document says **“eligible for remote work”** rather than **“work remotely.”**

A semantic retrieval strategy would still find the right chunk because it understands similarity in meaning.

---

## Dense Retrieval (Semantic Retrieval)

Dense retrieval uses **embeddings** and vector similarity search to retrieve semantically relevant content.

Instead of exact word matching, text is converted into vector representations.

### How it works

Flow:

```text
Document chunk
→ embedding vector
→ vector database

User query
→ query embedding
→ nearest-neighbor similarity search
→ top-k relevant chunks
```

Similarity is typically measured using:

- cosine similarity
- dot product
- Euclidean distance

Common ANN (Approximate Nearest Neighbor) search methods:

- HNSW
- IVF / PQ
- DiskANN
- ScaNN

### Example

Document chunk:

```text
"Employees become eligible for remote work after six months."
```

User query:

```text
"When can I work from home?"
```

Even though **remote work** and **work from home** are different words, semantic similarity enables retrieval.

### Advantages

- Handles synonyms and paraphrasing.
- Strong semantic understanding.
- Good for natural language questions.
- Useful in enterprise search and conversational systems.

### Limitations

- May retrieve semantically similar but incorrect chunks.
- Weak for exact identifiers (IDs, numbers, codes, names).
- Can miss precise keyword matches.
- Requires embedding computation and vector infrastructure.

### Best use cases

- Question answering
- Semantic document search
- Conversational RAG
- Long-form knowledge retrieval

Example vector search:

```text
Query embedding
→ nearest neighbors
→ top 5 chunks
```

---

## Sparse Retrieval (Lexical / Keyword Retrieval)

Sparse retrieval focuses on **exact term matching** rather than semantic similarity.

The most common method is:

- BM25

Other sparse approaches include:

- TF-IDF
- inverted index search
- keyword matching

### How BM25 works

BM25 ranks documents based on:

- keyword frequency
- rarity of terms
- document length normalization

Simplified intuition:

```text
More matching terms
+ rarer terms
+ better keyword density
= higher ranking
```

### Example

Document:

```text
Error code: DB_CONN_TIMEOUT_445
```

User query:

```text
What is DB_CONN_TIMEOUT_445?
```

Dense retrieval might fail because the identifier is unusual.

BM25 succeeds because of exact matching.

### Advantages

- Excellent for IDs, product names, error codes, legal clauses.
- Fast and interpretable.
- Strong keyword precision.
- No embedding model required.

### Limitations

- Poor synonym handling.
- Sensitive to wording differences.
- Misses semantic meaning.

Example failure:

Document:

```text
remote work policy
```

Query:

```text
work from home rules
```

Keyword search may fail because words differ.

### Best use cases

- Documentation lookup
- Error code search
- Compliance/legal systems
- Structured enterprise search

---

## Hybrid Retrieval

Hybrid retrieval combines **dense retrieval + sparse retrieval**.

This is one of the most common production approaches because it balances precision and recall.

Idea:

```text
Semantic relevance (dense)
+
Keyword precision (sparse)
=
better retrieval
```

Architecture:

```text
User query
   ├── Dense retrieval
   └── Sparse retrieval
          ↓
     merge candidates
          ↓
       reranking
          ↓
     top chunks to LLM
```

### Example

User query:

```text
database timeout issue DB_CONN_TIMEOUT_445
```

Dense retrieval finds:

```text
"database connectivity failures"
```

Sparse retrieval finds:

```text
"DB_CONN_TIMEOUT_445 troubleshooting guide"
```

Hybrid retrieval returns both.

### Fusion approaches

#### Score fusion

Combine ranking scores:

```text
final_score =
α(dense_score)
+
β(sparse_score)
```

Example:

```text
0.7 semantic
+ 0.3 keyword
```

Weights depend on domain.

#### Reciprocal Rank Fusion (RRF)

A popular ranking technique.

Instead of raw scores, combine ranking positions.

Formula:

```text
RRF(d) = Σ 1 / (k + rank_i(d))
```

Where:

- `rank_i(d)` = document rank in a retrieval list
- `k` = smoothing constant (often ~60)

Benefits:

- Stable
- Robust across retrieval methods
- Avoids score normalization issues

### Advantages

- Higher recall and precision.
- Better handling of mixed queries.
- Strong performance in production systems.

### Limitations

- More infrastructure complexity.
- Higher latency.
- Requires tuning and evaluation.

### Best use cases

- Enterprise search
- Customer support systems
- Knowledge assistants
- Multi-domain RAG

---

## Metadata Filtering

Metadata filtering improves relevance by restricting retrieval to a subset of documents.

Instead of searching everything, retrieval is constrained.

Common metadata:

```text
source document
department
date
author
permissions
language
region
document version
```

Example metadata:

```json
{
  "department": "HR",
  "year": 2025,
  "country": "India"
}
```

Query:

> “India leave policy”

Search:

```text
department = HR
country = India
```

before semantic retrieval.

### Example pipeline

```text
Query
→ metadata filtering
→ dense/sparse retrieval
→ reranking
→ context assembly
```

Benefits:

- Higher precision
- Faster search
- Better access control
- Reduced irrelevant context

Example:

Without filter:

```text
All departments searched
```

With filter:

```text
Only HR documents searched
```

---

## Query Rewriting and Expansion

Users often ask vague or incomplete questions.

Query understanding improves retrieval quality.

Techniques include:

### Query rewriting

Rewrite ambiguous queries.

Example:

User query:

```text
vacation rules
```

Rewritten query:

```text
employee paid leave vacation policy
```

Another example:

```text
WFH policy
```

↓

```text
work from home remote work employee policy
```

### Query expansion

Add synonyms and related terms.

Example:

```text
car insurance
```

Expanded:

```text
car insurance auto insurance vehicle coverage
```

### Multi-query retrieval

Generate multiple retrieval queries.

Example:

User asks:

```text
How do we onboard vendors?
```

Generated retrieval queries:

```text
vendor onboarding process
supplier registration workflow
third-party procurement onboarding
```

All results are merged.

Benefits:

- Higher recall
- Better handling of vague language
- Improved semantic coverage

Risks:

- Query drift (too broad)
- Extra latency
- More irrelevant retrieval

---

## Advanced Retrieval Strategies

### Multi-stage retrieval

Retrieve broadly first, refine later.

Pipeline:

```text
Query
→ BM25 top 200
→ dense retrieval top 50
→ reranker top 10
→ LLM context
```

Why?

Dense retrieval across millions of documents is expensive.

This reduces cost.

### Hierarchical retrieval

Retrieve progressively.

Example:

```text
document
→ section
→ paragraph
→ chunk
```

Useful for:

- PDFs
- books
- legal contracts
- technical manuals

Example:

```text
Policy document
  ├── Leave section
  │     ├── Vacation rules
  │     └── Sick leave
```

### Parent-child retrieval

Store small chunks but retrieve larger context.

Example:

Indexed chunk:

```text
Paragraph 3
```

Returned context:

```text
Entire section
```

Prevents context fragmentation.

### Graph retrieval

Use relationships between entities.

Example:

```text
employee
→ department
→ policy
→ manager
```

Useful in:

- knowledge graphs
- enterprise systems
- recommendation systems

### Self-query retrieval

The LLM generates structured retrieval constraints.

Example:

User:

> “Engineering docs from last month about authentication”

Generated filter:

```json
{
  "department": "engineering",
  "date": "last_month",
  "topic": "authentication"
}
```

Then retrieval executes with filters.

---

## Retrieval Tradeoffs

| Strategy           |               Strength |            Weakness | Best For          |
| ------------------ | ---------------------: | ------------------: | ----------------- |
| Dense              | semantic understanding | weak exact matching | conversational QA |
| Sparse             |        exact precision |      poor semantics | IDs, logs, legal  |
| Hybrid             |       balanced quality |     more complexity | production RAG    |
| Metadata filtering |              precision | needs good metadata | enterprise search |
| Query rewriting    |                 recall |         query drift | vague questions   |

In practice:

```text
Production RAG
=
Hybrid retrieval
+ metadata filters
+ reranking
+ query rewriting
```

This combination usually gives the strongest results in real systems.
