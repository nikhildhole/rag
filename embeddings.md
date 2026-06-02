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
