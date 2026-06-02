## Evaluation

Evaluation is the process of measuring how well a RAG system performs in retrieval quality, answer quality, reliability, latency, and overall user experience.

Since RAG involves both retrieval and generation, evaluation is usually split into two parts:

- **Retrieval evaluation**: checks whether the system retrieves the correct and relevant information.
- **Generation evaluation**: checks whether the final answer is accurate, useful, and grounded in retrieved context.

The goal is to continuously measure system quality, identify failures, and improve performance.

### Retrieval Evaluation

Retrieval evaluation measures whether relevant chunks are successfully retrieved for a query.

Common metrics:

- **Recall@K**: measures whether relevant information appears in the top K retrieved results.

Example:

```text id="s4b2k9"
Top 5 retrieved chunks
Relevant chunk found → success
```

Interpretation:

- Recall@5 = relevant document appears in top 5

- Higher recall → lower chance of missing important context

- **Precision@K**: measures how many retrieved chunks are actually relevant.

Example:

```text id="ev81xn"
Retrieved: 5 chunks
Relevant: 4 chunks
Precision@5 = 4/5
```

Higher precision means less noisy retrieval.

- **MRR (Mean Reciprocal Rank)**: evaluates how early the first correct result appears.

Example:

```text id="0n2mbj"
Correct result rank = 2
MRR = 1/2
```

Higher MRR indicates better ranking quality.

- **nDCG (Normalized Discounted Cumulative Gain)**: evaluates ranking quality while considering graded relevance.

Useful when multiple documents have different relevance levels.

Things to evaluate:

- retrieval relevance
- chunk ordering quality
- metadata filtering quality
- reranking effectiveness
- retrieval latency

### Generation Evaluation

Generation evaluation measures answer quality after the LLM produces a response.

Common evaluation areas:

- **Factuality**: is the answer correct and supported by retrieved evidence?
- **Hallucination rate**: does the model invent unsupported information?
- **Answer relevance**: does the response address the user query?
- **Completeness**: is important information missing?
- **Helpfulness**: is the answer understandable and useful?
- **Grounding quality**: does the answer rely on retrieved context?

Example:

Question:
“What is the refund policy?”

Good answer:
uses retrieved policy text and answers correctly.

Bad answer:
hallucinates policy details not present in context.

### Human Evaluation

Automatic metrics are useful but often insufficient.

Human reviewers may evaluate:

- correctness
- readability
- usefulness
- citation quality
- trustworthiness

Example questions:

- Was the answer accurate?
- Was the retrieved context relevant?
- Did the answer hallucinate?
- Was important information missing?

Human evaluation is especially important in production systems.

### A/B Testing

Compare multiple RAG strategies to determine what works better.

Examples:

Compare:

- chunk size A vs chunk size B
- reranker enabled vs disabled
- embedding model A vs embedding model B
- prompt strategy A vs B

Measure:

- answer quality
- latency
- user satisfaction
- retrieval performance

### Production Monitoring

Evaluation is continuous in production.

Common signals to monitor:

- query latency
- retrieval failures
- hallucination rate
- answer quality drift
- embedding drift
- retrieval quality drift
- cost per request

Example:

If users suddenly receive irrelevant answers:

Possible causes:

- bad embeddings
- stale index
- retrieval regression
- metadata filtering issue

### Debugging Failures

When answers are poor, evaluate each pipeline stage:

```text id="p1w8zu"
Question
   ↓
Retrieval quality?
   ↓
Reranking issue?
   ↓
Context assembly issue?
   ↓
Prompt problem?
   ↓
Generation failure?
```

Example:

Bad answer may be caused by:

- wrong chunks retrieved
- poor chunking
- context overflow
- weak reranking
- hallucinating generation

### Why Evaluation Matters

Without evaluation, it is difficult to know whether failures come from retrieval, prompt design, embeddings, or generation.

Good evaluation helps:

- improve accuracy
- reduce hallucinations
- increase trust
- optimize latency and cost
- maintain system quality over time

Simple explanation:

Evaluation measures whether the RAG system retrieves the right information and generates accurate, grounded, and helpful answers.
