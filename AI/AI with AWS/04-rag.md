# 04 — RAG (Week 4)

The biggest single topic in Domain 1, and the one most real GenAI systems are actually made of.

Give the model your documents, so it answers from them instead of from memory.

---

## The one-line version

> **RAG is an open-book exam.** The model didn't memorise your notes — you looked them up
> and put them on the desk in front of it.

Everything below is engineering around that sentence.

---

## Two pipelines, not one

The single most clarifying thing about RAG: **there are two completely separate pipelines**,
and people conflate them constantly.

```
INGESTION  (offline, occasional — a batch job)
  documents → chunk → embed → store vectors + metadata
                                      ↓
                                 vector store
                                      ↓
RETRIEVAL  (online, every single query — the hot path)
  question → embed → search → top chunks → build prompt → model → answer + citation
```

They run at different times, cost different money, and fail differently.

- **Ingestion** is a batch job. Slow is fine. Runs when documents change.
- **Retrieval** is in your request path. Every millisecond and every token counts.

> **Remember:** ingestion is your ETL. Retrieval is your query. You'd never confuse those
> two in a data pipeline — don't here either.

---

## Chunking — the single biggest quality lever

You cannot embed a whole document. You split it into chunks, and **how you split decides
how good your system is.** More RAG quality is won and lost here than anywhere else.

### Why it matters — a concrete example

Say your `00-glossary.md` gets split at a fixed 500 tokens, and the boundary lands here:

```
── chunk 7 ends ──────────────────────
   ...Top-k: sort by probability, keep the top k, throw the rest away.
── chunk 8 begins ────────────────────
   Top-p: walk down the sorted list adding up probabilities...
```

Now someone asks: **"What's the difference between top-k and top-p?"**

Retrieval returns chunk 7. The model answers confidently about top-k, says nothing useful
about top-p, and **you have no idea anything went wrong** — the answer looks fine.

**Half the answer, full confidence. That's the RAG failure mode.**

### The size tradeoff

| | Small chunks (~200 tokens) | Large chunks (~1500 tokens) |
|---|---|---|
| Retrieval precision | ✅ Sharp — the vector means one thing | ❌ Diluted — one vector for many ideas matches nothing well |
| Context for the model | ❌ Fragments that may not stand alone | ✅ Complete thoughts |
| Cost per query | ✅ Fewer input tokens | ❌ More input tokens, every query |
| Risk | Splitting an idea in half | Burying the answer in noise |

There's no universal right answer. **There is a right answer for your corpus, and you find
it by measuring** — which is what the golden set is for.

### Overlap

Repeat the last N tokens of each chunk at the start of the next (typically 10–20%).

```
chunk 1: [========================]
chunk 2:                  [========================]
                          ^^^^^^^ overlap
```

It buys insurance against exactly the top-k/top-p split above, at the cost of storing and
embedding duplicated text. Cheap insurance. Use it.

### Strategies, and what Bedrock offers

| Strategy | How it splits | Good for |
|---|---|---|
| **Fixed-size** | Every N tokens | Uniform prose; the simple default |
| **Default** | Fixed-size with sensible presets | Getting started |
| **Semantic** | At topic shifts, detected by embedding similarity | Mixed-topic documents |
| **Hierarchical** | Small chunks for *searching*, larger parents returned for *context* | The best of both — often the strongest option |
| **None** | You've pre-chunked it yourself | Your own custom logic |

**Hierarchical is worth understanding properly**, because it dissolves the size tradeoff:
search over precise child chunks, then hand the model the surrounding parent. Precise
retrieval, full context.

> ⚠️ **Your notes are markdown with headings — use that.** Document-structure-aware
> chunking that splits on `##` boundaries will beat any token-count rule on this corpus,
> because the headings already mark where ideas begin and end.

### Store metadata with every chunk

Non-negotiable. At minimum: **source filename, and the heading it came from.**

Without it you cannot cite, and without citation you cannot tell a real answer from a
hallucination. **Metadata is what makes an answer verifiable.**

---

## Embeddings

Turn each chunk into a vector (note 00). A few things that matter in practice:

**Dimensions** — more dimensions capture more nuance but cost more storage and slightly
slower search. Some models let you choose. Default is usually fine; don't optimise this first.

> ⚠️ **The gotcha that will bite you:** you must use the **same embedding model** for
> ingestion and for queries. Vectors from different models are not comparable — you'll get
> silent garbage, not an error.
>
> And **changing your embedding model means re-embedding your entire corpus.** Choose it
> deliberately; it's the most expensive thing to change later.

**Cost** — embedding is cheap per token and it's a one-time cost per document version. It's
not where your money goes. Your money goes to the vector store sitting there, and to the
retrieved tokens on every query.

---

## Vector stores

Where the vectors live and get searched. Bedrock Knowledge Bases can use several:

| Store | When it fits |
|---|---|
| **OpenSearch Serverless** | The default. Managed, scales, **and see the warning below** |
| **Aurora PostgreSQL + pgvector** | You already run Postgres. Vectors sit beside your relational data — one database, one backup, one set of credentials |
| **Neptune Analytics** | GraphRAG — when relationships between entities matter, not just similarity |
| **Pinecone / Redis / MongoDB Atlas** | Existing investment or specific features |

> ### 💸 STOP — read this before the Knowledge Base lab
> **An OpenSearch Serverless collection bills continuously**, with a minimum capacity floor,
> whether or not you ever query it. Left running, it costs **hundreds of dollars a month**
> and it is by far the most expensive mistake available in this whole syllabus.
>
> - **Delete the collection the moment the lab ends.** Not tomorrow. That evening.
> - Set a calendar reminder for the same day.
> - Check Cost Explorer the next morning.
>
> If you'd rather not risk it at all: **Aurora Serverless + pgvector** is a gentler place to
> learn, and you already know Postgres.

**For the exam:** pgvector when they already have Postgres and want to avoid another system.
Neptune Analytics when the scenario stresses *relationships*. OpenSearch Serverless as the
general managed default.

---

## Bedrock Knowledge Bases

The managed version of everything above: point it at S3, pick a chunking strategy and an
embedding model, and it handles ingestion, chunking, embedding, storage and retrieval.

**Ingestion:** documents in S3 → you trigger a **sync** → it chunks, embeds and indexes.
Sync again when documents change; it processes incrementally.

### The two APIs — know this cold

| API | What it does | Use when |
|---|---|---|
| **`Retrieve`** | Returns the matching chunks. **You** build the prompt and call the model | You want control over the prompt, want to mix in other context, or need custom logic |
| **`RetrieveAndGenerate`** | Does retrieval *and* generation, returns an answer **with citations** | You want a working RAG endpoint with minimal code |

> **Remember:** `Retrieve` gives you the ingredients. `RetrieveAndGenerate` gives you the meal.
> Exam scenarios that stress "custom prompt logic" or "combine with other data sources" point
> at `Retrieve`.

---

## Why top-k alone is not enough

Naive RAG is: embed the question, take the 5 nearest chunks, done. It works in a demo and
disappoints in production. Three fixes, in order of value:

### 1. Hybrid search (semantic + keyword)

Semantic search understands meaning but is **bad at exact strings**.

Ask *"how do I fix a ValidationException?"* — semantic search finds chunks about errors and
validation generally. **Keyword search finds the chunk containing the literal word
`ValidationException`.** You want both.

> **Rule:** any corpus with error codes, function names, product SKUs, version numbers or
> acronyms needs hybrid search. That's most technical corpora, including yours.

### 2. Metadata filtering

Filter before or during search — by source, date, document type, tenant.

```
query: "what's our refund policy?"
filter: { department: "billing", effective_date: { $gte: "2026-01-01" } }
```

> ⚠️ **Security, not just relevance.** In a multi-tenant system you must filter by tenant
> **as part of retrieval**, never by discarding results afterwards. Chunks that reach the
> prompt have already leaked — the model has seen them, and it may quote them. This is a
> real exam scenario and a real breach.

### 3. Re-ranking

Retrieve broadly (say 25 chunks), then run a **re-ranker** that scores each chunk against
the query more carefully, and keep the best 5.

Embedding similarity is a fast, rough proxy for relevance. A re-ranker is slower and more
accurate. Cast a wide net cheaply, then judge carefully.

> **Remember:** retrieve wide, rank narrow.

---

## Kendra vs. a vector store

**Amazon Kendra** is managed enterprise search: connectors for SharePoint, Confluence, S3,
Salesforce and friends, natural-language queries, and — the differentiator — **it respects
source document ACLs**, so a user only ever retrieves what they were already allowed to read.

| Use Kendra when | Use a vector store when |
|---|---|
| Many enterprise sources with existing connectors | You control the documents |
| **Per-user permissions must be honoured** | Everyone sees the same corpus |
| You want search infrastructure, not to build it | You want control and lower cost |

Kendra costs meaningfully more. **The word that should make you think "Kendra" in an exam
scenario is *permissions* or *existing enterprise connectors*.**

---

## Debugging RAG: retrieval or generation?

The most valuable diagnostic skill in this entire plan. When an answer is wrong, ask **one
question first:**

> **Was the correct chunk actually retrieved?**

Log your retrieved chunks. Then look.

**❌ The right chunk wasn't retrieved → a search problem.** The model never stood a chance.
- Chunking split the answer → change strategy, add overlap
- Question phrased differently from the document → hybrid search, query rewriting
- Right chunk ranked 8th, you took 5 → raise top-k, add re-ranking
- Wrong embedding model for the domain

**✅ The right chunk was retrieved, answer still wrong → a generation problem.**
- Prompt doesn't say "answer only from the documents" → note 03
- No escape hatch, so it filled the gap by inventing
- Too many chunks, answer buried in noise → retrieve fewer, better
- Temperature isn't 0

> **Remember:** **most RAG failures are search failures, not model failures.** Reaching for a
> bigger model to fix bad retrieval is the classic expensive mistake — and a classic wrong
> exam answer.

You'll formalise this split into real metrics in Week 12. Start doing it by hand now.

---

## Where the money goes

Tying back to note 02:

| | When | Rough weight |
|---|---|---|
| Embedding the corpus | Once per document version | Small |
| **Vector store** | **Continuously, whether used or not** | **Often the biggest line** |
| Retrieved tokens in the prompt | Every single query | Grows with traffic |
| Generation | Every query | Output tokens are the pricey ones |

**This is why chunking is a cost lever.** Retrieving 5 × 1500-token chunks instead of
5 × 600 sends an extra 4,500 input tokens on *every* query. Tune retrieval and you cut
quality problems and the bill at the same time.

---

## 🔨 Build — the spine project gets real

Do this **twice**. That's the point.

**Path A — by hand (understand it):**
- [ ] Walk the repo, split notes on `##` headings, keep 15% overlap
- [ ] Store `{ text, file, heading }` per chunk
- [ ] Embed all chunks; save vectors to a local JSON file (no infra, no bill)
- [ ] On a query: embed it, cosine-similarity against all chunks, take top 5
- [ ] Build the prompt from note 03 — delimiters, escape hatch, "cite the filename"
- [ ] **Print the retrieved chunks alongside every answer.** Non-negotiable — this is how
      you learn to see retrieval failures

**Path B — managed (compare it):**
- [ ] Notes to S3, create a Knowledge Base, sync
- [ ] Try `Retrieve`, then `RetrieveAndGenerate`
- [ ] **Then delete the OpenSearch collection.** Today.

**Then measure:**
- [ ] Run your 20-question golden set through both
- [ ] Change chunk size (300 / 600 / 1200) and rerun. **Watch the score move.**
- [ ] Add hybrid search or re-ranking. Watch it move again.

Seeing your own score change when you change chunk size is the moment RAG stops being
theory.

---

## ✍️ Write your note

`04-rag-notes.md`:
- Your chunk-size experiment — sizes vs. scores, as a table
- One question that failed, and whether it was retrieval or generation
- Hand-rolled vs. Knowledge Base: what the managed version did for you, and what it took away

---

## ✅ Checkpoint

1. Draw both RAG pipelines and say when each runs.
2. Why does chunk size trade retrieval precision against answer completeness?
3. What does hierarchical chunking solve?
4. What happens if you embed queries with a different model than your documents?
5. `Retrieve` vs. `RetrieveAndGenerate` — when does each win?
6. Give a concrete query where semantic search fails and keyword search succeeds.
7. Why must tenant filtering happen during retrieval rather than after?
8. An answer is wrong. What is the first thing you check?
9. What makes Kendra the right answer in a scenario?
10. Have you deleted the OpenSearch collection?

---

*Next: `05-customization.md` — fine-tuning, LoRA, distillation, and the decision table the
exam keeps asking about.*
