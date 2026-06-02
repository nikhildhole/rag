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

Covers:

- Storage layers in RAG:
  - Vector DB for semantic retrieval
  - Document store for chunk text and metadata
  - Object store for raw files
  - Relational DB for application/business data

- ANN indexing strategies:
  - HNSW
  - IVF / PQ
  - DiskANN
  - ScaNN

- Sparse and keyword retrieval:
  - BM25
  - token-based matching

- Hybrid retrieval:
  - combining dense + sparse search

- Metadata indexing:
  - filtering
  - permissions
  - freshness
  - source ranking

- Operational concerns:
  - incremental updates
  - reindexing
  - sharding and scaling

- Storage architecture and production tradeoffs

See: [storage\_&_indexing.md](storage_&_indexing.md)

## Retrieval Strategies

Retrieval strategies define how a RAG system finds relevant information from stored knowledge before passing context to the LLM for answer generation.

Key retrieval approaches include:

- **Dense retrieval**: semantic vector-based search using embeddings and similarity measures (cosine similarity, dot product, Euclidean distance).
- **Sparse retrieval**: keyword-based retrieval such as BM25 or TF-IDF for exact term matching.
- **Hybrid retrieval**: combines dense and sparse retrieval to improve precision and recall.
- **Metadata filtering**: narrows search using metadata such as source, department, date, permissions, or document type.
- **Query rewriting & expansion**: improves retrieval by rewriting ambiguous queries, adding synonyms, or generating multiple search variations.
- **Reranking**: reorders retrieved results using cross-encoders or LLM-based ranking for higher relevance.
- **Advanced retrieval methods**: includes multi-stage retrieval, hierarchical retrieval, parent-child retrieval, graph retrieval, and self-query retrieval.

The goal of retrieval strategies is to improve relevance, reduce noise, increase retrieval accuracy, and provide better context for grounded LLM responses.

See: [storage\_&_indexing.md](storage_&_indexing.md)

## Query Understanding & Reranking

### Query Understanding

Query understanding improves retrieval quality by analyzing and optimizing user queries before retrieval.

#### Query Rewriting

- Reformulates unclear or incomplete queries into retrieval-friendly queries.
- Improves semantic understanding and retrieval accuracy.
- Can be rule-based, LLM-based, or context-aware.

Example:
`"vacation rules"` → `"company vacation leave policy and employee leave rules"`

#### Query Expansion

- Adds related terms, synonyms, or alternate phrasings.
- Improves retrieval recall and coverage.

Example:
`"remote work policy"` → `WFH, hybrid work, telecommuting, remote employee policy`

#### Intent Detection

- Identifies user purpose or goal.
- Helps select retrieval logic and downstream workflows.

Examples:

- Informational → document retrieval
- Transactional → API/workflow routing
- Comparison → comparative retrieval

#### Entity Detection

- Extracts important entities or keywords from a query.
- Helps filtering, boosting relevance, and metadata search.

Example:
`"AWS pricing for GPU instances"`
Entities → AWS, GPU instances, pricing

#### Query Routing

- Routes queries to relevant knowledge sources or indexes.
- Improves retrieval speed and relevance.

Example:

- HR policy → HR index
- Expense reimbursement → Finance index

---

### Reranking

Reranking improves relevance by reordering retrieved results after retrieval.

#### Cross-Encoder Reranking

- Scores query and document together for better relevance.
- Produces more accurate ranking than embedding similarity alone.
- More compute-intensive but highly effective.

#### LLM-Based Reranking

- Uses LLM reasoning to rank retrieved chunks.
- Adds contextual understanding and safety-aware ranking.
- Useful for complex retrieval scenarios.

#### Freshness-Based Ranking

- Prioritizes newer and updated documents.
- Ensures recent policies and information rank higher.

#### Diversity Ranking

- Reduces duplicate or repetitive chunks.
- Improves context variety and coverage.

#### Metadata-Based Ranking

- Uses metadata such as:
  - source trust
  - permissions
  - recency
  - department
  - authority

Example:
Official HR policy > informal internal discussion

### End-to-End Flow

User Query
→ Intent Detection
→ Entity Extraction
→ Query Rewrite / Expansion
→ Query Routing
→ Retrieval (Dense / Sparse / Hybrid)
→ Top-K Results
→ Reranking (Cross-Encoder / LLM / Metadata)
→ High-Quality Context
→ LLM Response

See: [query*understanding*&\_reranking.md](query_understanding_&_reranking.md)

## Context Assembly

Context assembly is the process of preparing retrieved information before passing it to the LLM. The goal is to provide relevant, clean, and structured context within the token limit for better grounded responses.

Key steps:

- **Collect retrieved chunks**: gather top results from dense, sparse, or hybrid retrieval.
- **Deduplicate overlapping content**: remove repeated or highly similar chunks to reduce redundancy.
- **Rank and order context**: arrange chunks by relevance, chronology, or source trust.
- **Apply metadata filters**: narrow results using metadata such as source, timestamp, department, language, or permissions.
- **Manage token budget**: keep high-signal chunks and truncate or compress low-value context to fit model limits.
- **Context compression**: summarize or reduce long/noisy chunks while preserving useful information.
- **Attach source references**: preserve provenance for transparency, debugging, and source-backed answers.
- **Construct final prompt**: combine system instructions, user query, and retrieved context before sending to the LLM.

Simple flow:

Retrieved chunks
→ Filter + deduplicate
→ Rank + organize
→ Optimize for token budget
→ Build prompt
→ LLM answer

See: [context_assembly.md](context_assembly.md)

## Generation Layer

The generation layer is the stage where the LLM produces a final response using the retrieved context and user query.

After retrieval and context assembly, the system builds a prompt containing instructions, the user question, and retrieved evidence. The LLM then generates an answer grounded in this provided information instead of relying only on pre-trained knowledge.

Key responsibilities:

- **Prompt construction**: combine system instructions, user query, conversation history (if needed), and retrieved context into a structured prompt.
- **Grounded answer generation**: instruct the model to answer using retrieved evidence to reduce hallucinations.
- **Reasoning over retrieved context**: synthesize information across multiple chunks to form a coherent response.
- **Source-aware responses**: optionally include citations or references to supporting documents.
- **Response formatting**: structure output as text, JSON, tables, summaries, or domain-specific formats.
- **Fallback handling**: manage cases where retrieval quality is poor or evidence is missing.

Typical prompt structure:

```text id="md5y6m"
System Instructions:
Answer only using provided context.

User Query:
What is the company leave policy?

Retrieved Context:
[chunk 1]
[chunk 2]
[chunk 3]

Generate final answer.
```

### Prompt Components

Common prompt inputs include:

- **System instructions**: define rules, constraints, tone, or grounding requirements.
- **User query**: the question or request from the user.
- **Retrieved context**: relevant chunks retrieved from the knowledge base.
- **Conversation history** (optional): maintain context in multi-turn interactions.

### Grounding Strategies

To reduce hallucination and improve reliability, prompts often include grounding instructions such as:

- answer only from provided context
- cite sources when possible
- state uncertainty if information is unavailable
- avoid unsupported assumptions

Example:

> “If the answer is not present in the context, say you do not know.”

### Answer Generation Styles

Depending on the use case, the generation layer may support:

- **Direct answers**: concise factual responses.
- **Summarization**: compress long retrieved content into shorter explanations.
- **Multi-step reasoning**: combine evidence across multiple chunks.
- **Structured generation**: produce JSON, SQL, reports, or templates.

### Handling Missing or Weak Context

When retrieval quality is poor:

- return uncertainty instead of hallucinating
- ask follow-up clarification questions
- retry retrieval or query rewriting
- fallback to general model knowledge (if allowed)

### Post-processing

After generation, systems may:

- validate response format
- filter unsafe output
- attach citations
- improve readability or formatting
- log prompts and responses for debugging

Simple flow:

Retrieved context
→ Prompt construction
→ LLM reasoning + grounded generation
→ Post-processing
→ Final response

Why it matters:

Even with strong retrieval, poor prompt construction or grounding can produce weak answers. The generation layer ensures retrieved information is transformed into useful, coherent, and reliable responses.

## Evaluation

## Evaluation

Evaluation measures how well a RAG system retrieves relevant information and generates accurate, grounded responses. Since RAG involves both retrieval and generation, evaluation is typically divided into retrieval quality and answer quality.

Key areas:

- **Retrieval evaluation**: measure whether relevant chunks are retrieved using metrics like Recall@K, Precision@K, MRR, and nDCG.
- **Generation evaluation**: assess factuality, hallucination rate, answer relevance, completeness, and helpfulness.
- **Human evaluation**: review correctness, readability, trustworthiness, and citation quality.
- **A/B testing**: compare chunking strategies, embedding models, rerankers, prompts, or retrieval pipelines.
- **Production monitoring**: track latency, retrieval quality drift, hallucination rate, cost, and system reliability.

The goal of evaluation is to improve retrieval quality, reduce hallucinations, and ensure reliable, high-quality responses over time.

See: [evaluation.md](evaluation.md)

## Scalability & Observability

## Scalability & Observability

Scalability and observability ensure a RAG system remains fast, reliable, and easy to monitor in production as data and traffic grow.

Key areas:

- **Scalability**: improve performance using caching, batching, streaming responses, async pipelines, and horizontal scaling.
- **Latency monitoring**: measure p95/p99 latency across retrieval, reranking, prompt construction, and LLM response generation.
- **Quality monitoring**: track query latency, retrieval quality drift, embedding drift, hallucination rate, and error rates.
- **Cost monitoring**: observe embedding, storage, vector search, and LLM inference costs.
- **Debugging & traceability**: log retrieved chunk IDs, prompt state, retrieval scores, metadata filters, and embedding versions for troubleshooting.

The goal is to maintain reliable, scalable, and high-performing RAG systems while identifying failures and performance bottlenecks early.

See [scalability\_&_observability.md](scalability_&_observability.md)
