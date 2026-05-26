# RAG

## What is RAG?

- RAG = Retrieval-Augmented Generation.
- Combines retrieval from a document collection with an LLM for generation.
- Goal: ground responses in source data, reduce hallucinations, and support up-to-date or proprietary knowledge.

## Core RAG Components

1. Document ingestion
2. Document processing and chunking
3. Embedding and vector storage
4. Retrieval/search layer
5. Context assembly and prompt construction
6. Generation and answer synthesis
7. Evaluation and monitoring

---

- **Why use RAG?**
  - Scale knowledge beyond model pretraining.
  - Update knowledge quickly without retraining.
  - Provide citations and provenance.
- **Main risks**
  - Retrieval errors, stale data, hallucinations, cost from many retrieval calls.
  - Token limits in prompt context.
- **Strong answer structure**
  - Define the concept.
  - Mention architecture.
  - Note trade-offs.
  - Give an example.

## Document Ingestion

- Sources: PDFs, HTML, emails, chat logs, support tickets, transcripts, audio/video (with transcription).
- Important: normalize encodings, preserve metadata (source, author, timestamp, URL), track provenance.

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

- Dense embeddings: transformer-based, semantic similarity.
- Sparse embeddings: token/term-based, good for keyword or interpretability.
- Use multimodal embeddings when data includes text+image/audio.

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
- Attach citations to support answers.

## Generation Layer

- Build prompts with:
  - system instructions,
  - user query,
  - retrieved evidence.
- Use grounding prompts to force citation and limit hallucination.
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
