In Retrieval-Augmented Generation, chunking is the process of splitting documents into smaller pieces before embedding and indexing them in a vector database.

Chunking directly affects:

- Retrieval accuracy
- Context relevance
- Token efficiency
- Latency and cost
- Hallucination rate

Poor chunking is one of the most common reasons a RAG system performs badly even when embeddings and LLMs are good.

---

# Quick Strategy Comparison

| Strategy         | Complexity | Quality    | Speed  | Best For              | Cost      |
| ---------------- | ---------- | ---------- | ------ | --------------------- | --------- |
| Fixed-Size       | ⭐         | ⭐⭐       | ⭐⭐⭐ | MVPs, logs            | Low       |
| Recursive        | ⭐⭐       | ⭐⭐⭐     | ⭐⭐   | Documentation, PDFs   | Low       |
| Semantic         | ⭐⭐⭐⭐   | ⭐⭐⭐⭐⭐ | ⭐     | Enterprise, legal     | High      |
| Sentence-Based   | ⭐         | ⭐⭐⭐     | ⭐⭐   | Blogs, conversations  | Low       |
| Structure-Aware  | ⭐⭐       | ⭐⭐⭐⭐   | ⭐⭐   | Tech docs, markdown   | Medium    |
| Sliding Window   | ⭐⭐       | ⭐⭐⭐     | ⭐     | QA systems            | Medium    |
| Agentic/Adaptive | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐     | Production enterprise | Very High |

---

# Why Chunking Matters

LLMs do not retrieve entire documents efficiently. They retrieve chunks.

If chunks are:

- too small → loss of context
- too large → noisy retrieval
- overlapping poorly → broken semantics

The retriever only sees embeddings of chunks, not the original full document.

---

# Main Chunking Strategies

## 1. Fixed-Size Chunking

Split text every N characters or tokens.

Example:

- 500 tokens
- 100 token overlap

```text
Chunk 1: tokens 1–500
Chunk 2: tokens 401–900
Chunk 3: tokens 801–1300
```

### Advantages

- **Simple implementation**: Straightforward logic, no complex parsing required. Can be implemented in 5 lines of Python code.
- **Deterministic and predictable**: Every run produces identical chunks. Great for reproducibility and debugging.
- **Very fast preprocessing**: O(n) complexity. No need for embeddings or semantic analysis. Can chunk millions of documents in minutes.
- **Easy to parallelize**: Each document chunk is independent. Split documents across workers without coordination.
- **Token budgeting friendly**: Exact token counts known upfront. Easy to optimize for LLM context windows.
- **Works for unstructured data**: Works on any text input without assumptions about structure.
- **Low memory footprint**: Minimal preprocessing state. Suitable for edge computing environments.

### Disadvantages

- **Semantic boundary violations**: Randomly splits in the middle of sentences, paragraphs, or ideas.
- **Poor handling of structured content**: Destroys formatting and structure. Tables, code blocks become meaningless.
- **Quality degradation with heterogeneous documents**: Same chunk size doesn't work for different content types.
- **Context loss at boundaries**: Important context might be split across chunks.
- **Lower retrieval precision**: High noise in chunks. Vector DB returns fragments of irrelevant information.
- **Overlap management complexity**: Adding overlap increases storage without guaranteed quality improvement.

### Best Use Cases

- **System logs and structured logs**: Each log line is typically independent.
- **Streaming data**: IoT sensor data, application logs where data comes as a stream.
- **Fast MVPs and prototypes**: Need to iterate quickly on RAG evaluation.
- **Homogeneous corpora**: Data with consistent structure like news articles.
- **Baseline benchmarks**: Use as a control to measure improvements from better strategies.
- **Large-scale systems with resource constraints**: Time/memory are bottlenecks.
- **Real-time indexing**: New documents arrive constantly. Fast chunking is critical.

### When to Avoid

- Legal documents (need full context of clauses)
- Technical documentation (code examples need semantic units)
- Scientific papers (references and context span sections)

---

## 2. Recursive Chunking

Split hierarchically:

```text
Paragraphs
  ↓
Sentences
  ↓
Words
```

The algorithm tries larger separators first.

Typical separators:

```python
["\n\n", "\n", ".", " ", ""]
```

### Advantages

- **Semantic boundary respect**: Tries to split on natural boundaries first (paragraphs, then sentences, then words).
- **Balanced semantic preservation**: Chunks are naturally coherent. A paragraph about authentication stays together.
- **Language-aware splitting**: Understands sentence boundaries. Keeps grammatical units intact.
- **Flexible and adaptive**: Automatically falls back to smaller separators if needed to respect size limits.
- **Handles diverse content**: Works on plain text, markdown, code. No special parsing required.
- **Better than fixed-size quality**: Measurably better retrieval quality (typically 20-30% improvement in Recall@5) at minimal computational cost.
- **Good default strategy**: Recommended as "first thing to try" in most RAG frameworks (LangChain, LlamaIndex use this as default).
- **Tunable separators**: Can customize separators for different languages or formats.

### Disadvantages

- **Chunk size variability**: Can't guarantee chunk size distribution. Some chunks might be 200 tokens, others 800.
- **Memory overhead**: Requires splitting multiple times until size target is met.
- **Less deterministic than fixed-size**: Different input can produce wildly different chunk distributions.
- **Doesn't understand semantic meaning**: Still splits at static separator boundaries, not at actual topic shifts.
- **Imperfect for code**: Code has different structure than prose. Separator-based splitting might break logical code blocks.
- **Overlap complexity**: Overlap handling requires careful management to avoid duplication or gaps.

### Best Use Cases

- **Documentation and wikis**: Markdown docs naturally have paragraph/sentence structure.
- **Blog posts and articles**: News content, blog articles, tutorials. Structure is naturally hierarchical.
- **PDFs with text extraction**: After PDF → text conversion, recursive splitting works well.
- **General knowledge bases**: Company wikis, Notion exports, Confluence pages.
- **Mixed content types**: When you have diverse documents but want a single strategy.
- **When semantic chunking is too expensive**: Get 80% of semantic benefits at 20% of the cost.
- **Production RAG systems**: Good balance of quality and performance for most use cases.

### Real-World Example

```markdown
# Authentication Guide

Paragraph 1: How to set up authentication...
(150 tokens) → stays together

Paragraph 2: OAuth 2.0 explained...
(300 tokens) → splits into 2 sentences → ~150 tokens each

Very long paragraph with 500+ tokens
→ splits into 5 sentences → ~100 tokens each
```

---

## 3. Semantic Chunking

Chunks are created based on meaning instead of length.

Technique:

1. Compute sentence embeddings
2. Measure semantic similarity between consecutive sentences
3. Split when similarity drops below threshold (detects topic shift)

Example:

```text
Sentence 1: "Authentication is the process..." (about auth)
Sentence 2: "Users log in with username..." (about auth) → HIGH similarity
Sentence 3: "Vector databases store embeddings..." (about vectors) → LOW similarity → SPLIT HERE
```

### Advantages

- **Highest retrieval quality**: Chunks are semantically coherent units. Recall@5 typically 10-40% better than recursive chunking.
- **True semantic boundaries**: Detects actual topic shifts, not just syntactic separators.
- **Perfect for QA systems**: Questions map to cohesive answer units. Reduces noise in retrieved context.
- **Excellent for complex documents**: Scientific papers, legal contracts where topics don't align with paragraph structure.
- **Interpretability**: Retrieved chunks are obviously relevant to user. Less need for reranking.
- **Handles implicit topics**: Detects thematic shifts even without explicit headings or markers.

### Disadvantages

- **Computationally expensive**: Must embed all sentences first. Preprocessing time: hours to days at scale.
- **Cost multiplier**: Embedding costs add 20-30% to total pipeline cost. Not practical for streaming data.
- **Hyperparameter sensitivity**: Similarity threshold is tricky to tune. Too low = over-chunking; too high = under-chunking.
- **Dependency on embedding model**: Quality depends entirely on embedding model quality. Need to retune if you switch models.
- **Latency overhead**: Preprocessing is slow. Can't do real-time chunking of new documents.
- **Storage overhead**: Storing embeddings for all sentences during preprocessing adds storage needs.
- **Not great for code**: Code semantics aren't well-captured by general-purpose embeddings.

### Best Use Cases

- **Enterprise RAG systems**: When retrieval quality directly impacts revenue/user satisfaction.
- **Legal document analysis**: Contracts, regulations where partial context is dangerous.
- **Medical/scientific knowledge bases**: Research papers where precision is critical.
- **High-value QA systems**: Customer support, legal Q&A where answer quality matters most.
- **Small to medium document sets**: <1M documents. Preprocessing cost is one-time. <$10K in embedding costs acceptable.
- **Offline systems**: Batch processing where latency doesn't matter. Preprocess once, serve forever.

### When to Avoid

- **High-velocity streaming data**: New data arriving constantly. Preprocessing overhead is unacceptable.
- **Cost-sensitive applications**: Every embedding call costs money. 2-3x cost multiplier is significant.
- **Code-heavy corpora**: General embeddings don't understand code semantics.
- **Very large document sets**: >10M documents. Preprocessing costs become prohibitive (>$100K+).

### Implementation Considerations

```python
# Typical implementation
sentences = split_into_sentences(document)
embeddings = model.embed_batch(sentences)

chunks = []
current_chunk = [sentences[0]]

for i in range(1, len(sentences)):
    similarity = cosine_similarity(embeddings[i-1], embeddings[i])

    if similarity < THRESHOLD:  # Topic shift detected
        chunks.append(' '.join(current_chunk))
        current_chunk = [sentences[i]]
    else:
        current_chunk.append(sentences[i])
```

**Tuning the threshold**:

- Similarity < 0.5 = aggressive splitting (many small chunks)
- Similarity < 0.7 = moderate splitting
- Similarity < 0.85 = conservative splitting (fewer, larger chunks)

---

## 4. Sentence-Based Chunking

Each chunk contains a fixed number of sentences.

Example:

```text
Chunk = 5 sentences
Overlap = 1 sentence
```

### Advantages

- **Cleaner boundaries than fixed-token**: Respects sentence structure. Never splits mid-sentence.
- **Easy implementation**: Requires basic sentence tokenization. No embeddings needed.
- **Good semantic preservation**: Sentences on the same topic usually maintain coherence.
- **More predictable than recursive**: Fewer edge cases. Distribution is more uniform.
- **Deterministic and reproducible**: Same sentences always produce same chunks.
- **Works across languages**: As long as you have a sentence tokenizer, strategy works.
- **Faster than semantic chunking**: Only requires sentence segmentation (milliseconds).
- **Better than fixed-size for readability**: Returned chunks are grammatically complete.

### Disadvantages

- **Highly uneven token sizes**: A chunk of 5 short sentences might be 100 tokens. 5 long sentences might be 800 tokens.
- **Sentence length variance**: Long sentences explode token count. A single long sentence might exceed budget by 5x.
- **Doesn't handle very long sentences**: In legal/technical genres sentences can be 500+ tokens each.
- **Fixed number doesn't match content**: 5 sentences is arbitrary. Some topics need 3, others need 10.
- **Overlap complexity**: When overlapping, must be careful with sentence ordering and duplication.
- **Less semantic than semantic chunking**: Doesn't detect actual topic shifts. Multiple unrelated sentences stay together.
- **Language dependent**: Sentence tokenization quality varies by language.

### Best Use Cases

- **Conversational data**: Chat logs, interviews, Q&A datasets.
- **Blog posts and articles**: News articles, blog posts naturally have sentence structure aligned with topics.
- **Educational content**: Textbooks, lecture notes, tutorials where sentence = concept unit.
- **Twitter/social media threads**: Natural sentence structure, manageable length.
- **Customer support tickets**: Questions and answers are usually sentence-bound.
- **When sentence tokenization is already available**: If you're already parsing text for other reasons.
- **Moderate quality + speed tradeoff**: Better than fixed-size, faster than semantic.

### Practical Example

```text
Document about machine learning:

Sentence 1: "Machine learning is a subset of AI." (15 tokens)
Sentence 2: "It enables systems to learn from data." (12 tokens)
Sentence 3: "The main categories are supervised and unsupervised learning." (10 tokens)
Sentence 4: "Supervised learning requires labeled data." (8 tokens)
Sentence 5: "The model learns patterns from input-output pairs." (11 tokens)
→ Chunk 1: 56 tokens (sentences 1-5)

Sentence 6: "Unsupervised learning works without labels." (7 tokens)
...
→ Chunk 2: (sentences 6-10)
```

**Problem**: If sentence 6 is replaced with a 250-token legal clause, chunk 2 becomes very unbalanced.

---

## 5. Document Structure-Aware Chunking

Uses document hierarchy to create chunks. Respects:

- Headings (H1, H2, H3, etc.)
- Sections and subsections
- Markdown formatting
- HTML/XML tags
- Code blocks as semantic units
- Lists and nested lists

Example:

```markdown
# Authentication ← H1

## OAuth Setup ← H2 (new chunk when too large)

Configuration steps...

### Step 1: Register App ← H3

Details...

### Step 2: Get Token ← H3

Details...

# API Keys ← H1 (new chunk at H1)

API key management...
```

### Advantages

- **Excellent semantic coherence**: Chunks are defined by document structure. Everything under a heading is thematically related.
- **Matches human understanding**: Humans wrote documents with hierarchies for a reason. Respect that structure.
- **Very high retrieval quality**: Chunks are meaningful units. Retrieved chunks are almost always relevant.
- **Great for citation/attribution**: Can cite as "Section: API Keys" instead of vague chunk numbers.
- **Handles heterogeneous content**: Code blocks stay intact, tables stay together, list items group properly.
- **User-friendly chunks**: When returned to users, chunks map to document sections they recognize.
- **Good for multi-modal docs**: Mixing text, code, diagrams. Structure markers delineate each type clearly.
- **Relatively efficient**: Doesn't require embeddings. Just structured parsing.

### Disadvantages

- **Requires clean structure**: Only works if document is well-structured. Messy documents with inconsistent headings break this.
- **Not all documents have clear hierarchy**: Some documents are just walls of text. No heading structure to exploit.
- **Parsing complexity**: Different formats have different structure rules. Markdown, HTML, PDFs, Word docs require different parsers.
- **Variable chunk sizes**: A section with 2 lines vs 2000 lines. Can't control chunk size distribution.
- **Over-chunking problem**: If a section is still too large (3000 tokens), need recursive fallback.
- **Empty or very short sections**: Some sections might be 1-2 sentences. Too small, needs aggregation.
- **Brittle to structural changes**: If document structure changes, all chunks change.

### Best Use Cases

- **Markdown documentation**: GitHub docs, Sphinx docs, Obsidian vaults. Markdown structure is reliable.
- **API documentation**: OpenAPI, REST docs, SDK references. Endpoint/method names are natural boundaries.
- **Technical documentation**: Kubernetes docs, software guides, hardware manuals. Well-structured by nature.
- **Notion databases and exports**: Notion uses consistent heading hierarchy.
- **Book chapters**: Books naturally have chapters, sections, subsections.
- **Code repositories with docs**: README.md, contributing guides, architectural docs.
- **When document structure is reliable**: If you trust the document structure, use it.

### When to Avoid

- **Unstructured text dumps**: Converted from PDFs or word docs with poor structure.
- **Dynamically generated docs**: If structure changes frequently, maintaining parsing logic is hard.
- **Highly variable content types**: One document might be Markdown, another HTML, another plain text.
- **Time-sensitive updates**: If document structure changes weekly, reindexing burden is high.

### Implementation Patterns

**Pattern 1: Heading-based chunking**

```
- Split at H1 (or H2) boundaries
- If section > MAX_SIZE, recursively split at H3
- If still > MAX_SIZE, fall back to sentence/fixed chunking
```

**Pattern 2: Hierarchical grouping**

```
- Group H3 sections under their H2 parent
- Chunk = H2 + all H3 sections under it
- If chunk > MAX_SIZE, split each H3 into its own chunk
```

**Pattern 3: Flexible depth**

```
- Find the heading depth that produces good chunk sizes
- Usually H2 or H3, not H1 (too broad)
- Preserve heading context in each chunk
```

---

## 6. Sliding Window Chunking

Creates overlapping windows of fixed size. Each new window starts partway through the previous one.

Example:

```text
Window size = 500 tokens
Stride = 250 tokens (50% overlap)
```

Visualization:

```text
[-----chunk1-----]  tokens 0-500
          [-----chunk2-----]  tokens 250-750
                    [-----chunk3-----]  tokens 500-1000
                              [-----chunk4-----]  tokens 750-1250

Overlap region: consecutive chunks share tokens
```

### Advantages

- **Prevents boundary information loss**: When an answer spans chunk boundaries, sliding window ensures full context is captured.
- **Improves recall**: Recall@K increases 10-15% because information isn't lost at boundaries.
- **Reduces answer fragmentation**: Important concepts that span chunk boundaries are available in their entirety somewhere.
- **Better for QA systems**: Typical RAG returns top-5 chunks. Overlapping windows mean more complete answers.
- **Handles multi-sentence facts**: Facts that span multiple sentences are captured in their entirety.
- **Works with any underlying chunking**: Can apply sliding window on top of fixed-size, recursive, or semantic chunking.

### Disadvantages

- **Storage overhead**: 50% overlap = 50% more vectors to store. For 1M chunks, now storing 1.5M vectors.
- **Embedding cost multiplier**: 50% overlap = 50% more embedding API calls. Directly increases embedding cost.
- **Retrieval latency**: More vectors to search during ANN query. Search time increases by 30-40%.
- **Indexing complexity**: Managing overlapping chunks during index updates is complex.
- **Redundancy in results**: Top-K results often return overlapping chunks. Need deduplication in post-processing.
- **Token budget waste**: In LLM context, overlapping chunks lead to duplicate context. Waste tokens on repeated information.
- **Increased memory during indexing**: More vectors in flight during batch indexing operations.

### Best Use Cases

- **QA systems**: Highest recall is priority. Cost of overlap is worth it for answer quality.
- **Dense fact retrieval**: When documents pack many facts closely together. Overlap helps retrieve all related facts.
- **Legal/compliance search**: Can't miss relevant information. Better to have redundancy than miss something important.
- **Medical information retrieval**: Missing a dose instruction is worse than redundancy.
- **When top-K must be comprehensive**: If you return top-5 results to user and they must see full answer.
- **Small document sets**: When storage/cost of overlap is acceptable (< 10M chunks).

### When to Avoid

- **Cost-sensitive systems**: Every embedding API call counts. 50% more calls = 50% more cost.
- **Latency-critical systems**: Real-time systems where p99 latency matters.
- **Very large corpora**: 100M+ chunks. Overlap becomes expensive.
- **Information retrieval where redundancy hurts**: News search, product search where duplicate results hurt UX.

### Configuring Overlap

**Common configurations:**

| Stride %         | Overlap % | Use Case                      | Cost Impact |
| ---------------- | --------- | ----------------------------- | ----------- |
| 75% (stride=125) | 25%       | Light overlap, minimal cost   | +25%        |
| 50% (stride=250) | 50%       | Moderate overlap, balanced    | +50%        |
| 25% (stride=375) | 75%       | Heavy overlap, maximum recall | +75%        |
| 10% (stride=450) | 90%       | Extreme overlap (rare)        | +90%        |

**Recommendation**: Start with 50% overlap (stride = window_size / 2). Measure recall improvement vs cost impact. Optimize from there.

---

## 7. Agentic / Adaptive Chunking

Modern advanced approach where chunk size and boundaries are determined dynamically.

Chunk size changes based on:

- **Topic density**: Dense technical sections need smaller chunks. Narrative sections can be larger.
- **Syntax and structure**: Code blocks chunk differently from prose. Preserve function/class boundaries.
- **Entropy and complexity**: High-entropy sections (lots of new concepts) get smaller chunks. Repetitive sections larger.
- **Semantics**: Detect topic shifts (like semantic chunking) but also adjust size based on complexity.
- **Document type**: A code file, PDF, blog post, legal contract chunk differently. Auto-detect and adapt.
- **Query characteristics**: Some implementations adapt based on query. Long queries might benefit from smaller chunks.

Often implemented with:

- **LLMs**: Use an LLM to analyze document and suggest chunk boundaries.
- **Classifiers**: Train a model to detect ideal chunk boundaries.
- **Semantic segmentation models**: Models that identify topic/subtopic shifts.
- **Heuristic systems**: Rules engine combining multiple signals (heading depth, sentence count, semantic shift, etc).

### Advantages

- **Best retrieval quality**: Adapts to content. Code chunks at function boundaries. Prose chunks at paragraphs.
- **Context-aware**: Understands what makes a good chunk for different content. One size doesn't fit all.
- **Multi-format support**: Single strategy handles code, prose, tables, lists, metadata.
- **Minimal semantic overhead**: Doesn't require embedding ALL sentences like semantic chunking.
- **Handles heterogeneous corpora**: Blogs + code + docs + video transcripts. Each formatted appropriately.
- **Operationally robust**: No fragile structure assumptions. Doesn't fail on unstructured data.
- **Future-proof**: As you add new document types, system adapts.

### Disadvantages

- **Very complex to implement**: No off-the-shelf solution. Requires building custom system.
- **Expensive preprocessing**: If using LLM to analyze every document: ~$0.10 per document. At scale: huge cost.
- **Slower than simpler approaches**: Analysis time. Classifier-based approaches are 10-100x slower than fixed-size chunking.
- **Hard to debug**: When chunking fails, hard to understand why. Multiple heuristics interacting.
- **Latency for new documents**: Can't be real-time. New documents need analysis before chunking.
- **Requires tuning and testing**: No universal parameters. Each domain needs tuning and validation.
- **Operationally hard**: Updating chunking logic in production is risky. Changes affect all downstream chunks.
- **Overkill for most use cases**: Best-in-class retrieval, but 80% of apps work fine with recursive chunking.

### Best Use Cases

- **Large enterprise RAG systems**: Cost is absorbed. Quality matters more than efficiency.
- **Multi-domain knowledge bases**: Need to handle diverse content types well.
- **High-stakes applications**: Legal, medical, financial where retrieval quality impacts risk/liability.
- **Large-scale systems**: 100M+ documents where even 5% quality improvement = millions in value.
- **When budget allows**: If you have ML/data team to build and maintain custom chunking system.

### When to Avoid

- **Small budgets**: Cost of development and preprocessing is prohibitive.
- **Real-time systems**: Latency of LLM analysis is unacceptable.
- **MVP/prototyping**: Too complex. Start with recursive, add sophistication later.
- **Simple use cases**: Blog search, news search, FAQ retrieval. Overkill.

### Example Implementation Concepts

**LLM-based approach (expensive but effective):**

```
1. Analyze document with LLM: "Suggest optimal chunk boundaries for this document"
2. Parse LLM response: [0-250, 250-600, 600-1200, ...]
3. Create chunks at suggested boundaries
4. Cost: ~$0.10 per document
```

**Heuristic approach (cheaper but less flexible):**

```
1. Detect content type (code, prose, mixed)
2. For code: chunk at function/class boundaries
3. For prose: use semantic chunking threshold
4. For mixed: apply type-specific rules
5. Cost: preprocessing only, minimal API calls
```

**Hybrid approach (balanced):**

```
1. Use structure-aware chunking as baseline
2. If chunks are very unbalanced, apply semantic adjustments
3. If document type is unknown, use classifier for guidance
4. Cost: low-moderate, quality: high
```

### Real-World Example

**Document: Python tutorial with prose and code**

```
# Tutorial Section
Prose about functions (200 tokens)
→ chunk as-is (single semantic unit)

# Code Example 1
def function_1():
    # 100 tokens
    complex logic
→ chunk separately (function boundary)

# Code Example 2
def function_2():
    # 300 tokens (very long)
→ split at 150 tokens (semantic boundary within function)

# Prose explanation
of the two functions (150 tokens)
→ chunk separately (prose, not code)
```

**Result**: Each chunk is optimal for its content type. Not just fixed-size = 200 tokens everywhere.

---

# Chunk Overlap

Overlap preserves continuity between chunks.

Example:

```text
Chunk Size = 500
Overlap = 100
```

Meaning:

- next chunk repeats last 100 tokens

This helps when:

- answers cross chunk boundaries
- concepts span sections

Typical overlap:

- 10–20%

---

# Choosing Chunk Size

There is no universal best size.

Typical ranges:

| Use Case        | Chunk Size                      |
| --------------- | ------------------------------- |
| General QA      | 300–700 tokens                  |
| Code            | 100–300 lines / semantic blocks |
| Legal           | 800–1500 tokens                 |
| Research papers | 500–1200 tokens                 |
| Chat history    | 20–50 messages                  |

---

# Important Tradeoffs

## Smaller Chunks

Pros:

- precise retrieval
- lower noise

Cons:

- missing context
- fragmented meaning

---

## Larger Chunks

Pros:

- richer context
- better reasoning

Cons:

- noisy retrieval
- token waste

---

# Retrieval Failure Examples

## Too Small

```text
"What is the refund policy?"
```

Chunk retrieved:

```text
"...within 30 days of..."
```

Missing important context.

---

## Too Large

Chunk contains:

- refund policy
- shipping
- pricing
- FAQ
- legal disclaimers

Retriever score diluted.

---

# Advanced Modern Strategies

## Parent-Child Retrieval

Store:

- small child chunks for embedding
- larger parent chunks for generation

Flow:

1. retrieve small precise chunks
2. expand to larger parent context

Very effective.

Used in:

- [LlamaIndex](https://www.llamaindex.ai?utm_source=chatgpt.com)
- advanced enterprise RAG

---

## Late Chunking

Embed entire document first, then derive chunk embeddings afterward.

Benefits:

- embeddings retain global context

Popular in recent retrieval research.

---

## Contextual Chunking

Add metadata/context to each chunk.

Example:

```text
Document: Kubernetes Guide
Section: Networking
Subsection: Services
```

This dramatically improves retrieval quality.

---

# Common Questions

### 1. "How do you choose a chunking strategy?"

**Answer structure:**

- Depends on: document type, use case, latency/cost constraints, team resources
- Start simple (fixed-size or recursive), measure, iterate
- Only move to semantic if retrieval quality is poor and ROI justifies cost

### 2. "What's the difference between fixed-size and semantic chunking?"

**Key points:**

- Fixed-size: splits every N tokens, fast, deterministic, may break semantics
- Semantic: uses embeddings to detect topic shifts, higher quality, slower, requires preprocessing embeddings

### 3. "How do you handle chunks that are too small?"

**Solutions:**

- Increase chunk size
- Use parent-child retrieval (small chunks → expand to parent context)
- Add contextual metadata to chunks
- Increase overlap percentage
- Use sliding window approach

### 4. "What happens if your chunks are too large?"

**Problems & Solutions:**

- Problem: retriever sees too much noise, low precision
- Solutions:
  - Reduce chunk size
  - Use reranking to filter high-quality chunks
  - Improve metadata filtering
  - Consider semantic chunking

### 5. "Tell me about overlap. How much do you use?"

**Answer:**

- Typical: 10–20% overlap
- Higher overlap (30–50%): better for boundary cases, higher storage
- No overlap: risk losing context at boundaries
- Depends on: chunk size, overlap patterns in your data

### 6. "How would you debug a RAG system with poor retrieval?"

**Debugging checklist:**

1. Sample retrieved chunks — are they relevant?
2. Check chunk quality — too large? too small? noisy?
3. Analyze failure cases — what's the pattern?
4. Measure: Recall@5, Recall@10, MRR
5. Try different chunking strategies
6. Add metadata filtering
7. Evaluate embedding quality separately

## Decision Framework

**START HERE:**

```
├─ Do you have clean document structure (markdown, HTML)?
│  ├─ YES → Use structure-aware chunking
│  └─ NO → Continue
│
├─ Do you need highest possible retrieval quality?
│  ├─ YES → Use semantic chunking (budget allowing)
│  └─ NO → Continue
│
├─ Is this MVP or exploration?
│  ├─ YES → Use fixed-size chunking (fast to iterate)
│  └─ NO → Continue
│
└─ Use recursive chunking (good default, balanced)
```

## Implementation Checklist

- [ ] Define chunk size based on use case and token limits
- [ ] Choose 1-2 strategies to test
- [ ] Set overlap: 10–20% recommended
- [ ] Implement metadata preservation (document ID, section, timestamp)
- [ ] Log chunk creation for debugging
- [ ] Measure baseline retrieval metrics
- [ ] A/B test different strategies
- [ ] Monitor chunk quality in production
- [ ] Plan for reindexing strategy

## Common Pitfalls to Avoid

1. **Setting chunk size too small** → Lost context, poor reasoning
2. **Setting chunk size too large** → Noisy retrieval, token waste
3. **No overlap** → Broken semantics at boundaries
4. **Not testing on actual queries** → Theory ≠ practice
5. **Ignoring document structure** → Missed semantic coherence
6. **Choosing expensive strategies without measurement** → Wasting money
7. **Static chunk sizes** → Different doc types need different sizes
8. **Not tracking chunk provenance** → Can't debug failures
9. **Assuming one strategy fits all docs** → Heterogeneous corpora need adaptive approaches
10. **Reindexing the entire vector DB** → Consider incremental updates

## Practical Tips

- **Start simple**: Fixed-size or recursive beats over-engineering
- **Measure first**: Sample retrieval, Recall@K, human evaluation
- **Iterate fast**: Change one variable at a time
- **Monitor drift**: Track retrieval quality over time
- **Test on edge cases**: Very long documents, very short, images in documents
- **Document decisions**: Record why you chose your strategy for future maintainers
- **Budget for preprocessing**: Semantic chunking can be expensive at scale
