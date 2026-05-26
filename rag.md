# RAG

Brief overview of a Retrieval-Augmented Generation (RAG) system: components, trade-offs, and practical recommendations for building a production-ready pipeline.

## 1. Document Sources / Ingestion

- Supported formats: Q&A logs, PDF, HTML/websites, emails, chat transcripts, audio/video (requires transcription).
- Notes: normalize encodings, preserve source metadata (timestamps, authors, URLs), and record provenance for citations.

## 2. Document Processing

- Steps: cleaning, language detection, metadata extraction, deduplication, OCR/text extraction and parsing.
- Tip: apply lightweight normalization (unicode, punctuation) and keep an audit trail for each document.

## 3. Chunking Strategies

See detailed strategies in [chunking_strategies.md](chunking_strategies.md)

Main approaches:

- **Fixed-size chunking**: simple, predictable token counts; use overlap to preserve context.
- **Recursive chunking**: split on semantic boundaries (headings, paragraphs) for higher-quality retrieval.
- **Semantic chunking**: compute embeddings to detect topic shifts and split at semantic boundaries.
- **Sentence-based chunking**: fixed number of sentences per chunk with clean semantic boundaries.
- **Document structure-aware chunking**: leverages markdown/HTML hierarchy and headings.
- **Sliding window**: useful for sequence continuity; tune chunk size and overlap for target LLM context window.
- **Agentic/adaptive chunking**: dynamic chunk sizes based on topic density, entropy, and document type.

## 4. Embedding Models

- Dense embeddings (transformer-based) are general-purpose; choose model size vs cost trade-offs.
- Sparse embeddings (e.g., token/term-based) help with interpretability and keyword matching.
- Multimodal embeddings for image/audio+text use-cases; consider domain-specific fine-tuning when necessary.

## 5. Storage Layer

- Recommended components: Vector DB (search), Document store (source retrieval/metadata), Object storage (raw files), Relational DB (app data).
- Examples: choose a vector DB that supports your scale and desired ANN index types and metadata filtering.

## 6. Indexing

- Index types: ANN (approximate nearest neighbor), BM25/keyword, or hybrid indexes combining both.
- Operational concerns: incremental indexing, sharding/partitioning, and reindexing strategies for embeddings updates.

## 7. ANN Algorithms

- Common options: HNSW (fast, memory-heavy), IVF/PQ (disk-friendly), ScaNN, DiskANN.
- Choose based on latency, memory, and recall trade-offs; benchmark on realistic query loads.

## 8. Retrieval / Search Algorithms

- Dense retrieval: nearest-neighbor on embeddings for semantic matches.
- Sparse retrieval: BM25/term-based for exact keyword signals.
- Hybrid retrieval: combine dense + sparse for better recall and precision; add metadata filters.

## 9. Query Understanding

- Techniques: query rewriting, expansion, intent detection, entity extraction, decomposition, and routing to specialized indexes or models.
- Use lightweight rewriting to map user intent to better retrieval queries.

## 10. Reranking

- Cross-encoder rerankers (pairwise) improve ordering at the cost of compute.
- LLM reranking can incorporate broader context and safety checks.
- Consider diversity and freshness when reranking results.

## 11. Context Assembly

- Deduplicate retrieved chunks, order by relevance and chronology, and attach citations/provenance.
- Token budgeting: prioritize high-signal chunks and apply context compression when near the LLM token limit.

## 12. Generation Layer

- Compose prompts with system + user + retrieved context; use templates and guardrails for safety and factuality.
- Options: direct generation, chain-of-thought, or grounding prompts that ask the model to cite sources.

## 13. Evaluation

- Retrieval metrics: Recall@K, MRR, nDCG; evaluate on held-out queries with labeled positives.
- Generation metrics: factuality/faithfulness, hallucination rate, answer relevance and helpfulness.
- Use human evaluation and online A/B testing to measure end-to-end impact.

## 14. Scalability & Latency

- Techniques: caching, batching, streaming responses, horizontal scaling, GPU acceleration, async pipelines.
- Measure p95/p99 latencies and tune index/configuration to meet SLOs.

## 15. Monitoring & Observability

- Track query traces, latency, retrieval quality drift, embedding drift, error rates, and cost.
- Log retrieved ids + embeddings fingerprints to help debug and detect regressions.
