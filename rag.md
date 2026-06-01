# RAG

## What is RAG?

“RAG stands for Retrieval-Augmented Generation. It is a technique where, before generating an answer, the system retrieves relevant information from external documents.

In practice, we convert documents into embeddings and store them in a vector database. When a user asks a question, the query is also converted into an embedding, and semantic search is used to retrieve the most relevant document chunks. Those retrieved chunks are added as context to the LLM, which then generates a response based on that information.

We use RAG to provide business-specific, private, or up-to-date answers and reduce hallucinations.”

## How does it work?

RAG works by retrieving relevant information before the LLM generates an answer.

Typically, documents from a knowledge source (PDFs, databases, websites, support tickets, internal company docs, etc.) are first processed and split into smaller chunks. These chunks are converted into embeddings using an embedding model and stored in a vector database for efficient retrieval.

When a user asks a question, the query is also converted into an embedding. The system performs semantic search to find the most relevant chunks by comparing the query embedding with stored document embeddings.

The retrieved chunks are then added to the prompt as context and sent to the LLM. Instead of answering only from its training knowledge, the model generates a response grounded in the retrieved information.

Simple flow:

Documents
→ Chunking
→ Embeddings
→ Vector DB storage

User query
→ Query embedding
→ Semantic retrieval of relevant chunks
→ Context added to prompt
→ LLM generates grounded answer

## Why do we use RAG?

RAG is used because standalone LLMs have limitations: they may hallucinate, lack access to private business knowledge, or not know recently updated information.

By retrieving external information at query time, RAG allows the model to answer using relevant and current context instead of relying only on what it learned during training.

Benefits of RAG include:

- **Access to external knowledge**: use company documents, databases, policies, or domain-specific information.
- **Up-to-date responses**: update knowledge by changing documents instead of retraining the model.
- **Reduced hallucination**: answers are grounded in retrieved evidence.
- **Business-specific answers**: provide responses tailored to organizational data and workflows.
- **Source-backed responses**: retrieved chunks can be referenced for transparency and trust.
- **Cost efficiency**: avoids expensive model retraining whenever information changes.

Example:

Without RAG:
An LLM may answer from outdated or generic knowledge.

With RAG:
The system retrieves relevant business documents and the LLM answers using that retrieved context, improving accuracy and relevance.

## Core RAG Components

1. Document ingestion
2. Document processing and chunking
3. Embedding and vector storage
4. Retrieval/search layer
5. Context assembly and prompt construction
6. Generation and answer synthesis
7. Evaluation and monitoring

---

## Document Ingestion

Document ingestion is the process of collecting and importing data into a RAG system so it can later be processed, indexed, and retrieved.

The goal of ingestion is to gather information from different sources and prepare it for downstream stages such as chunking, embedding generation, and retrieval.

Common data sources include:

- PDFs and documents
- HTML/web pages
- Emails
- Chat logs and support tickets
- Knowledge base articles
- Transcripts
- Audio/video files (after transcription)
- Databases, APIs, or internal business systems

During ingestion, the system extracts raw content and associated metadata while preserving traceability.

Important ingestion tasks include:

### Data extraction

Extract readable content from raw sources.

Examples:

- Parse text from PDFs or Word documents
- Scrape HTML content
- Convert audio/video into text using transcription
- Read structured records from databases

### Metadata collection

Metadata helps retrieval, filtering, ranking, and source attribution.

Common metadata fields:

- source document
- file name
- author
- timestamp/date
- URL
- department or category
- document version
- access permissions

Example:

```text
Chunk:
"Employees are eligible for remote work after 6 months"

Metadata:
source = HR_policy.pdf
page = 14
department = HR
timestamp = 2025-01-15
```

### Data normalization

Different sources may use inconsistent formats or encodings.

Typical normalization tasks:

- normalize text encoding (UTF-8)
- remove corrupted characters
- standardize whitespace and formatting
- clean HTML noise
- unify timestamps and date formats

This improves downstream chunking and embedding quality.

### Deduplication

Remove duplicate or repeated content to avoid redundant retrieval and wasted storage.

Example:

The same FAQ document may exist in:

- email attachment
- internal wiki
- PDF export

Keeping duplicates can bias retrieval.

### Traceability and lineage

Track where each piece of information came from.

This is important for:

- citations/source-backed responses
- debugging retrieval failures
- compliance and auditing
- trust and explainability

Simple flow:

Raw sources
→ Extract content
→ Normalize + clean
→ Attach metadata
→ Deduplicate
→ Ready for chunking and embeddings

## Document Processing & Chunking

- Clean text, detect language, extract metadata, dedupe, OCR where needed.
- Chunking is critical for retrieval quality.
- Reference: [chunking_strategies.md](chunking_strategies.md)
- Common chunking styles:
  - Fixed-size chunks with overlap.
  - Recursive/semantic chunks using headings, paragraphs, topic boundaries.
  - Sentence-based chunks for clean context.
  - Sliding window for long documents.
  - Adaptive chunking based on density or structure.

## Embeddings

Embeddings convert text, images, or other data into numerical vector representations that capture semantic meaning. In RAG, document chunks and user queries are embedded into the same vector space to enable similarity-based retrieval.

Common embedding types:

- **Dense embeddings**: transformer-based vectors for semantic similarity and natural language understanding.
- **Sparse embeddings**: token/term-based representations (e.g., BM25/TF-IDF) for keyword precision and interpretability.
- **Multimodal embeddings**: represent text, image, audio, or video in a shared space for cross-modal retrieval.

Embeddings are typically stored in a vector database and used for nearest-neighbor search during retrieval.

See: [embeddings.md](embeddings.md)

## Storage & Indexing

- Storage layers:
  - Vector DB for ANN search.
  - Document store for metadata and source retrieval.
  - Object store for raw files.
  - Relational DB for application data.
- Index choices:
  - ANN indexes (HNSW, IVF/PQ, ScaNN, DiskANN).
  - Sparse/BM25 indexes.
  - Hybrid indexes combining semantic and keyword retrieval.
- Operational concerns: incremental updates, sharding, and reindexing.

## Retrieval Strategies

- Dense retrieval: semantic nearest-neighbor on vectors.
- Sparse retrieval: BM25 / exact keyword matching.
- Hybrid retrieval: combine both for better precision+recall.
- Use metadata filters and query rewriting for better relevance.

## Query Understanding & Reranking

- Query work:
  - Rewrite/expand queries.
  - Detect intent and entities.
  - Route to specialized indexes.
- Reranking:
  - Cross-encoder rerankers improve ordering.
  - LLM rerankers can add safety and context-aware relevance.
  - Consider diversity, recency, and freshness in reranking.

## Context Assembly

- Collect top retrieval results.
- Deduplicate overlapping chunks.
- Order by relevance, chronology, or source trust.
- Enforce token budget: keep high-signal chunks and compress or truncate low-value context.
- Attach source references to support answers.

## Generation Layer

- Build prompts with:
  - system instructions,
  - user query,
  - retrieved evidence.
- Use grounding prompts to ask the model to reference sources and limit hallucination.
- Options: direct answer, chain-of-thought, multi-step reasoning.

## Evaluation

- Retrieval metrics: Recall@K, MRR, nDCG.
- Generation metrics: factuality, hallucination rate, answer relevance, helpfulness.
- Mention human evaluation and A/B testing for production systems.

## Scalability & Observability

- Scale with caching, batching, streaming, async pipelines, horizontal scaling.
- Measure p95/p99 latency and tune indexes.
- Monitor: query latency, retrieval quality drift, embedding drift, error rates, cost.
- Debugging: log retrieved IDs, prompt state, and embedding fingerprints.
