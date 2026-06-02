### Query Understanding & Reranking

**Query Understanding and Reranking are critical stages in retrieval systems (especially RAG) because they directly impact retrieval quality. Even if embeddings and vector databases are strong, poor query understanding or weak ranking can result in irrelevant context reaching the LLM, which reduces answer quality.**

---

## 1. What is Query Understanding?

**Query understanding is the process of analyzing and improving the user query before retrieval happens.**

The goal is to understand:

- What the user is asking (**intent detection**)
- Which important concepts or entities matter (**entity extraction**)
- Whether the query is ambiguous or incomplete
- How to improve retrieval quality using rewriting or expansion

In production RAG systems, users often ask vague, short, or incomplete questions. Query understanding helps convert those inputs into retrieval-friendly queries.

### Why Query Understanding is Needed

Users rarely ask questions in an ideal searchable format.

Example:

**User Query:**
_"leave policy"_

Problems:

- Too short
- Missing context
- Could refer to HR policy, sick leave, maternity leave, vacation leave, or legal compliance

After query understanding:

**Expanded Query:**

> “employee leave policy including annual leave, sick leave, and HR guidelines”

This increases retrieval precision and recall.

---

## 2. Query Understanding Pipeline

A typical query understanding pipeline includes:

```text
User Query
    ↓
Intent Detection
    ↓
Entity Extraction
    ↓
Query Rewriting / Expansion
    ↓
Query Routing
    ↓
Retrieval
```

Each stage improves retrieval quality.

---

## 3. Query Rewriting

**Query rewriting means reformulating the user query into a clearer and retrieval-optimized version while preserving intent.**

The rewritten query should be:

- clearer
- more specific
- semantically richer
- easier for dense/sparse retrievers to match

### Why Query Rewriting Matters

Users may write:

- incomplete queries
- conversational queries
- typo-heavy queries
- ambiguous questions

Example:

**Original Query:**

> “vacation rules”

Rewritten query:

> “company vacation leave policy and employee leave rules”

This improves document matching.

### Query Rewriting Approaches

#### A) Rule-Based Rewriting

Simple deterministic logic.

Example:

```text
WFH → work from home
PTO → paid time off
AI → artificial intelligence
```

Advantages:

- Fast
- Cheap
- Predictable

Disadvantage:

- Limited flexibility

---

#### B) LLM-Based Rewriting

Use an LLM to rewrite queries semantically.

Example:

User query:

> “refund issue”

LLM rewrite:

> “customer refund policy and refund request handling process”

Benefits:

- Better semantic understanding
- Handles ambiguity
- More context aware

Challenge:

- Adds latency and cost

---

#### C) Context-Aware Rewriting

Uses chat history.

Example:

Conversation:

```text
User: Tell me about GPT-4 pricing
User: What about enterprise?
```

Rewritten query:

> “GPT-4 enterprise pricing”

Without rewriting, retrieval quality drops because _“enterprise”_ alone lacks context.

---

## 4. Query Expansion

**Query expansion improves retrieval by adding related terms, synonyms, or alternate phrasings to the query.**

The purpose is to improve **recall** (finding more relevant documents).

Example:

Query:

> “car insurance”

Expanded query:

```text
car insurance
automobile insurance
vehicle insurance
motor insurance
coverage policy
```

This helps retrieve documents using different wording.

### Query Expansion Techniques

#### A) Synonym Expansion

Add related words.

Example:

```text
doctor → physician
car → automobile
policy → regulation
```

Useful for keyword retrieval (BM25).

---

#### B) Semantic Expansion

Use embeddings or LLMs to add conceptually related phrases.

Example:

Query:

> “remote work policy”

Expansion:

```text
work from home
hybrid work
telecommuting
WFH guidelines
employee remote policy
```

---

#### C) Multi-Query Expansion

Generate multiple retrieval queries.

Instead of one query:

```text
How to reset password?
```

Generate:

```text
password reset process
forgot password instructions
change account password
authentication recovery
```

Then merge retrieval results.

This increases recall significantly.

---

## 5. Intent Detection

**Intent detection identifies what the user wants to achieve.**

Why it matters:

Different intents require different retrieval logic.

Examples:

| Query                     | Intent         |
| ------------------------- | -------------- |
| “refund policy”           | informational  |
| “refund my order”         | transactional  |
| “compare GPT-4 vs Claude” | comparison     |
| “best vector DB”          | recommendation |

Intent can determine:

- retriever type
- prompt strategy
- ranking logic
- downstream workflow

Example:

**Informational intent**

→ retrieve documents

**Transactional intent**

→ route to APIs or business systems

---

## 6. Entity Detection

**Entity detection identifies important keywords or business entities from the query.**

Examples:

Query:

> “AWS pricing for GPU instances”

Detected entities:

```text
AWS
GPU instances
pricing
```

Why it matters:

Entities can:

- improve filtering
- boost ranking
- route retrieval

Example:

Query:

> “HR leave policy in India”

Detected entities:

```text
Department = HR
Region = India
Topic = leave policy
```

Retrieval can then apply metadata filters.

---

## 7. Query Routing

**Query routing decides where to search.**

In enterprise RAG, data may exist in multiple indexes:

```text
HR documents
Engineering docs
Support tickets
Finance database
Product manuals
```

Instead of searching everything, route intelligently.

Example:

Query:

> “expense reimbursement”

Route to:

> Finance index

Query:

> “PTO policy”

Route to:

> HR index

Benefits:

- lower latency
- better relevance
- reduced noise

---

## 8. What is Reranking?

**Reranking is the process of reordering retrieved results to improve relevance after the initial retrieval stage.**

Most systems retrieve many chunks first:

```text
Top 50 results
```

Then reranking selects the best ordering.

Pipeline:

```text
User Query
    ↓
Retriever (Vector DB / BM25)
    ↓
Top-K Results
    ↓
Reranker
    ↓
Best Context Chunks
    ↓
LLM
```

This is important because retrieval often returns noisy results.

---

## 9. Why Reranking is Needed

Initial retrieval may retrieve semantically similar but weakly relevant chunks.

Example:

Query:

> “leave policy after maternity”

Retrieved results:

1. Employee benefits overview
2. Vacation leave policy
3. Maternity leave eligibility ← actually relevant
4. Holiday calendar

Without reranking:

The LLM may receive poor context.

With reranking:

```text
1. Maternity leave eligibility
2. Employee leave policy
3. Benefits handbook
```

Answer quality improves significantly.

---

## 10. Cross-Encoder Reranking

**Cross-encoder rerankers are among the most common reranking approaches.**

How it works:

Instead of embedding query and document separately:

Cross-encoders evaluate:

```text
(query + document)
```

together.

Example:

Query:

> “refund policy”

Document A:

> “Refund requests allowed within 30 days”

Document B:

> “Payment methods accepted”

Cross-encoder scores:

```text
A → 0.95
B → 0.11
```

Then reorder.

Advantages:

- Higher accuracy
- Better semantic relevance

Disadvantage:

- Slower
- More compute expensive

Common models:

- BERT rerankers
- Cohere rerank
- BGE reranker

---

## 11. LLM-Based Reranking

LLMs can also rank retrieval results.

Example prompt:

```text
Rank these chunks by relevance to:
"remote work leave policy"
```

Benefits:

- Better contextual understanding
- Handles nuanced intent
- Safety-aware ranking

Example:

Can prioritize:

- trusted sources
- recent documents
- policy-approved content

Challenge:

- expensive
- higher latency

---

## 12. Diversity, Freshness, and Metadata-Based Reranking

Reranking is not only semantic relevance.

Production systems often include:

### A) Freshness Ranking

Prefer newer documents.

Example:

```text
HR policy 2025
HR policy 2022
```

Rank 2025 higher.

---

### B) Diversity Ranking

Avoid repeated chunks.

Bad:

```text
same paragraph repeated 5 times
```

Good:

```text
policy overview
exceptions
eligibility
compliance
```

This improves context coverage.

---

### C) Metadata-Based Ranking

Boost by:

- source trust
- document authority
- permissions
- department
- recency

Example:

Official HR policy PDF > employee Slack message

---

## 13. End-to-End Flow

```text
User Query
      ↓
Intent Detection
      ↓
Entity Extraction
      ↓
Query Rewrite / Expansion
      ↓
Query Routing
      ↓
Retriever (Vector DB / BM25 / Hybrid)
      ↓
Top-K Results
      ↓
Cross-Encoder / LLM Reranker
      ↓
High-Quality Context
      ↓
LLM Response
```
