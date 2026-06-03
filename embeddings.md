# Embeddings

Embeddings are numerical vector representations of data (text, images, audio, or multimodal content) that encode semantic meaning. In RAG, embeddings are used to represent documents and queries in a shared vector space so that semantically similar content can be retrieved efficiently.

The key intuition:

> Similar meaning → nearby vectors in embedding space.

Example:

Query:
"How do I reset my password?"

Document:
"I forgot my login credentials"

Even though wording differs, embeddings place these texts close together because their semantic meaning is similar.

---

## How Embeddings Work in RAG

RAG retrieval typically follows this process:

1. Split documents into chunks.
2. Convert chunks into embeddings using an embedding model.
3. Store embeddings in a vector index or vector database.
4. Convert the user query into an embedding.
5. Perform similarity search to retrieve nearest chunks.
6. Pass retrieved context to the LLM for grounded generation.

Pipeline:

Raw documents
→ Chunking
→ Embedding generation
→ Vector index / database
→ Query embedding
→ Similarity search
→ Retrieved context
→ LLM response

---

## Similarity Search

Retrieval depends on comparing vector similarity between the query embedding and document embeddings.

Common similarity measures:

### Cosine Similarity

Measures directional similarity between vectors.

\cos(\theta)=\frac{A\cdot B}{|A||B|}

Interpretation:

- Near 1 → highly similar
- Near 0 → unrelated
- Near -1 → opposite direction

Why it matters:

Cosine similarity ignores vector magnitude and compares semantic direction, making it robust for retrieval.

### Dot Product

Measures vector alignment while considering magnitude.

Pros:

- Fast
- Often used in optimized ANN systems

Cons:

- Sensitive to vector magnitude

### Euclidean Distance (L2)

Measures geometric distance between vectors.

Used less frequently in modern semantic retrieval but still relevant in ANN systems.

---

## Types of Embeddings

### Dense Embeddings

Dense embeddings are transformer-generated vectors where most dimensions contain values.

Characteristics:

- Semantic understanding
- Strong at synonym and paraphrase matching
- Works well for natural language retrieval

Example:

Query:
"cancel my subscription"

Retrieved chunk:
"terminate recurring billing"

Advantages:

- Semantic similarity
- Better recall
- Natural-language understanding

Limitations:

- Weak for exact identifiers
- Less interpretable
- Can miss keywords

Common use cases:

- Knowledge assistants
- Enterprise search
- Semantic document retrieval

---

### Sparse Embeddings

Sparse embeddings represent token importance across a high-dimensional space where most dimensions are zero.

Examples:

- BM25
- TF-IDF
- Learned sparse retrievers (e.g., SPLADE)

Characteristics:

- Strong keyword precision
- Exact term matching
- Explainable ranking

Example:

Query:
"ERR_504_X2"

Sparse retrieval performs well because exact token matching matters.

Advantages:

- Precise keyword matching
- Better for IDs, legal text, product names
- High interpretability

Limitations:

- Weak semantic understanding
- Sensitive to wording mismatch

Example failure:

Query:
"heart attack treatment"

Document:
"myocardial infarction therapy"

Sparse retrieval may fail because terms differ.

---

### Multimodal Embeddings

Multimodal embeddings map multiple data types into a shared semantic space.

Supported modalities:

- Text
- Image
- Audio
- Video

Example:

Text:
"red sports car"

Retrieved:
an image of a red sports car

Use cases:

- Search screenshots in PDFs
- Image retrieval using text
- Video/audio semantic search
- Enterprise document retrieval with diagrams

Common model:
CLIP (Contrastive Language–Image Pretraining)

---

## Embedding Dimensions

Embedding vectors vary in dimensionality.

Examples:

- 384
- 768
- 1536
- 3072

Tradeoffs:

Smaller vectors:

- Faster retrieval
- Lower storage cost
- Lower latency

Larger vectors:

- Richer semantic representation
- Higher memory and compute cost

Higher dimensions do not always guarantee better retrieval.

---

## Hybrid Retrieval

Production RAG systems often combine dense and sparse retrieval.

Dense retrieval:

- semantic relevance

Sparse retrieval:

- exact keyword matching

Example query:

"AWS IAM role policy for S3 bucket"

Sparse helps:

- AWS
- IAM
- S3

Dense helps:

- semantic meaning

Benefits:

- Better precision
- Better recall
- Stronger production reliability

---

## Embedding Model Selection

Embedding quality depends on the model and domain.

Consider:

- Language support
- Latency
- Embedding size
- Cost
- Domain specialization

Examples:

- General-purpose embeddings
- Domain-specific legal/medical embeddings
- Multilingual embeddings
- Multimodal embeddings

Choosing an embedding model is often a tradeoff between quality, latency, and cost.

---

## Common Embedding Problems

### Vocabulary mismatch

Query:
"car"

Document:
"automobile"

Dense retrieval helps.

### Exact identifier failure

Query:
"ERROR_293"

Sparse retrieval helps.

### Context fragmentation

Poor chunking breaks related information across chunks and hurts retrieval.

### Embedding drift

If documents change over time but embeddings are not regenerated, retrieval quality degrades.

### Poor retrieval quality

Causes:

- weak chunking
- bad embedding model
- no reranking
- poor indexing strategy

---

## Production Considerations

Important operational concerns:

- Batch embedding generation
- Embedding caching
- Incremental re-indexing
- ANN search for scalability
- Metadata filtering
- Monitoring retrieval quality drift
- Latency optimization

Typical production stack:

Documents
→ Chunking
→ Embeddings
→ Vector DB (ANN index)
→ Retrieval
→ Reranking
→ Prompt assembly
→ LLM generation

---

## Questions

1. What are embeddings?
2. How are embeddings used in RAG?
3. Dense vs sparse embeddings?
4. Why cosine similarity?
5. Dot product vs cosine similarity?
6. Why hybrid retrieval?
7. Why are embeddings stored in vector databases?
8. What causes poor retrieval quality?
9. Why does chunking affect embeddings?
10. What is embedding drift?

---

## Common Embedding Models

- **OpenAI — ada-002**: **Dimension:** 1536. **When to use / Why:** Good general-purpose English embeddings for production RAG when using a managed API. Balanced quality and cost, widely used as a reliable default. **Notes:** Cloud-only, store model name and version with vectors for reproducibility.

- **OpenAI — text-embedding-3-small**: **Dimension:** 1536 (provider docs may update). **When to use / Why:** Modern general-purpose embedding with competitive quality; good default for many retrieval tasks. **Notes:** Check provider for best-fit variant and costs.

- **sentence-transformers/all-MiniLM-L6-v2 (Hugging Face)**: **Dimension:** 384. **When to use / Why:** Low-latency, low-cost, and tiny memory footprint — ideal for on-device or local deployments and fast indexing. **Tradeoffs:** Lower capacity for subtle semantic distinctions.

- **sentence-transformers/all-mpnet-base-v2**: **Dimension:** 768. **When to use / Why:** Strong general-purpose semantic quality for English; good balance of performance and size for many production systems.

- **LaBSE (Language-agnostic BERT Sentence Embeddings)**: **Dimension:** 768. **When to use / Why:** Multilingual and cross-lingual retrieval — use when you need consistent embeddings across many languages.

- **CLIP (e.g., ViT-B/32) — multimodal**: **Dimension:** 512. **When to use / Why:** Image⇄text retrieval and multimodal RAG. Use CLIP-style models to embed images and short text into a shared space.

- **Domain-specific / Fine-tuned models**: Dimensions vary. **When to use / Why:** Fine-tune or train a sentence-transformer when your domain (legal, medical, product catalogs) contains jargon or specialized semantics — yields the best retrieval quality for that domain.

### Practical notes on model selection

- **Dimension tradeoffs:** Smaller dims (e.g., 384) → faster, cheaper, less storage; larger dims (e.g., 768–1536+) → richer semantics but higher cost and index size. Pick based on retrieval quality targets, storage budget, and latency.
- **Local vs managed:** Use Hugging Face / sentence-transformers for offline/local inference and reproducibility. Use managed APIs for operational simplicity and regular model updates.
- **Versioning:** Always record model name + version + preprocessing pipeline in vector metadata so you can re-run or re-embed if needed.
- **Benchmarking:** Evaluate candidates on a small labeled dev set (precision@k, recall@k, MRR) and also measure downstream generation quality in your RAG prompts.

---

## Embedding Drift

### What is embedding drift?

Embedding drift is the gradual or sudden change in the properties or distribution of embeddings over time such that retrieval quality degrades. Drift can cause previously relevant documents to rank lower or for semantically similar items to move farther apart in vector space.

### Why embedding drift occurs

- **Document/content drift (data drift):** The corpus evolves (new topics, vocabulary, styles) but old embeddings are not refreshed.
- **Concept drift:** The meaning or significance of terms changes (e.g., new product names, slang, changing technical usage).
- **Preprocessing or pipeline changes:** Tokenization, normalization, chunking, or stopword handling changed between embedding runs.
- **Model updates or swapping:** Upgrading/downgrading embedding models (or provider rolled new weights) changes the embedding geometry.
- **Partial re-embedding:** Only some documents get re-embedded (e.g., new items), mixing vectors from different model versions.

### How to detect embedding drift

- **Embedding metadata audits:** Track model name/version and preprocessing used for each vector; detect mixed versions.
- **Statistical distribution monitoring:** Compute summary statistics over embeddings (mean vector, norms, variance) and track shifts over time. Large changes indicate drift.
- **Similarity-to-anchors:** Keep a set of stable anchor texts and monitor their average cosine similarity to nearest neighbors in the corpus. Falling similarity signals drift.
- **Nearest-neighbor stability:** For a set of probe queries, track how often the top-k results change (rank persistence). Sudden or consistent changes signal drift.
- **Downstream performance metrics:** Monitor retrieval metrics (precision@k, recall@k, MRR) on a labeled dev set and monitor generation quality (ROUGE, human evals) if available.
- **Embedding norm / compactness checks:** Track per-vector norms and pairwise distance distributions; dramatic shifts can indicate model or preprocessing changes.
- **Visual checks:** Periodically reduce embeddings via PCA/UMAP/t-SNE and inspect clustering; useful for investigations though not production-only signals.

### How to fix or mitigate embedding drift

- **Version and metadata discipline:** Persist model name, version, and preprocessing used with each vector. Refuse to mix vectors from different model versions without intentional migration.
- **Re-embedding strategy:** Schedule periodic full re-embeddings or incremental re-embedding strategies based on content churn. For high-change domains, re-embed nightly or weekly; for stable domains, re-embed less frequently.
- **Anchors and A/B tests:** Maintain anchor queries and run A/B comparisons when changing models to quantify impact before rolling out.
- **Canary / staged rollouts:** Re-embed a subset of the corpus and validate retrieval/generation before a global re-index.
- **Hybrid retrieval fallback:** Use a hybrid dense+sparse approach or a keyword fallback to reduce user-facing regressions while re-embedding.
- **Automated drift alerts:** Set thresholds for distributional metrics (e.g., mean cosine drop, KL divergence of projection) and alert when exceeded.
- **Preprocessing stability:** Keep preprocessing deterministic and documented; include the preprocessing code or hash in vector metadata.
- **Incremental rolling reindex with consistency guarantees:** Reindex in a way that maintains availability (e.g., index new vectors in parallel and switch alias after validation).

### Operational checklist

- Store `model_name`, `model_version`, `preprocessing_hash`, and `embed_timestamp` with every vector.
- Maintain a small labeled dev set for continuous retrieval evaluation.
- Monitor: average cosine to anchors, distributional metrics, precision@k on dev queries, embedding norms.
- Automate periodic re-embedding or create an on-change re-embedding pipeline for high-churn content.
- Run regression checks whenever updating the embedding model or preprocessing.

---
