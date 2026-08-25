# 02 — Choosing a Model (Week 3, part 1)

**Domain 1 — 31% of the exam.** The single biggest block, and "which model should this
workload use?" is the exam's favourite question shape.

Week 2 taught you to call *a* model. This note is about calling the *right* one.

---

## The mental shift

There is no "best model". There is only **the cheapest model that clears the bar for this
specific job.**

A backend analogy you already believe: you don't put every table in Aurora Serverless with
max capacity "just to be safe". You size to the workload. Same discipline here — except the
cost difference between a small model and a frontier model can be **20–30×**, and it applies
to every single request forever.

> **Remember:** picking a model is capacity planning, not shopping. You're not asking
> "which is best", you're asking "what is the least I can get away with".

---

## The seven criteria

Work through these in order. The first one that rules a model out saves you evaluating the rest.

### 1. Modality — can it even accept your input?

| Modality | Input → output |
|---|---|
| Text | text → text |
| **Multimodal** | text + images (sometimes audio/video) → text |
| Embedding | text → vector *(not a chat model — different job entirely)* |
| Image generation | text → image |

**Multimodal pipelines** are worth understanding as a design choice: an insurance claim with
photos can either go to one multimodal model, or through a chain (extract text → analyse
image → combine). One model is simpler; a chain is cheaper and each step is separately
testable. **The exam likes the "simpler vs. cheaper and testable" tradeoff.**

> ⚠️ Embedding models are not chat models. You cannot ask one a question. They exist only
> to turn text into vectors for retrieval — that's Week 4.

### 2. Region — is it available where you're running?

Straight from Week 2. Models are region-scoped and the newest land in a few regions first.
A model that's perfect but unavailable in your compliance region is not an option.

Also here: **cross-region inference profiles** — an ID (prefixed like `us.`) that lets AWS
serve your request from any region in a group. Better throughput and resilience, but the
request may be processed outside your primary region, **which is a data-residency decision,
not just a performance one.** Exam scenarios use this as a discriminator.

### 3. Context window — does your input fit?

Add it up honestly: system prompt + conversation history + retrieved documents + room for
the answer.

> ⚠️ **A big context window is a capability, not a plan.** Filling 200k tokens costs 200k
> tokens' worth of money on *every call*, and long contexts often *reduce* answer quality —
> models can lose track of material buried in the middle. "Just stuff everything in" is
> almost always the wrong exam answer; retrieve less, better.

### 4. Quality — is it actually good enough?

The only honest measure is **your task, your data**. Public leaderboards tell you very
little about whether a model can classify *your* support tickets.

This is what the golden-set harness below is for.

### 5. Cost — and where it actually goes

Priced per million tokens, **input and output billed separately, output typically several
times more expensive.**

Worked example — a RAG assistant, 100k requests/month:

```
Per request:
  input   3,000 tokens   (system prompt + retrieved chunks + question)
  output    300 tokens   (the answer)

Per month:
  input   300,000,000 tokens
  output   30,000,000 tokens
```

Two things fall out of this, and both are exam-relevant:

**(a) Input dominates by volume — 10× the output here.** In RAG, most of your bill is the
documents you retrieved. Cutting retrieved context from 3,000 to 1,500 tokens **halves your
biggest line item**. That makes chunking and retrieval tuning a *cost* lever, not just a
quality one. Remember this in Week 4.

**(b) Output dominates by unit price.** So a model that rambles costs more even at identical
quality. "Be concise" in the system prompt is a cost optimisation.

Run this arithmetic with real prices before choosing. A 20× cheaper model that's good enough
turns a $4,000/month feature into $200.

### 6. Latency — which kind matters?

Two different numbers, and confusing them is a classic mistake:

| | What it is | Matters for |
|---|---|---|
| **Time to first token** | How long before *anything* appears | Chat UIs — fix with streaming |
| **Total generation time** | Until the last token | Batch jobs, anything you must parse before acting |

A chat UI can hide a slow model behind streaming. **A Step Functions task that parses JSON
cannot** — it waits for the whole thing. Same model, completely different acceptability.

### 7. Throughput and licensing

- **On-demand** — pay per token, shared capacity, subject to quotas and throttling
- **Provisioned throughput** — reserved capacity, guaranteed rate, billed by time whether
  you use it or not *(the cost crossover is Week 11)*
- Check the provider's terms if you're in a regulated industry — they differ by vendor, and
  Bedrock doesn't flatten that

---

## The rule the exam keeps testing

> **Start with the smallest model that could plausibly work. Move up only when you have
> evidence it isn't good enough.**

Most production GenAI tasks — classification, extraction, routing, summarising, answering
from a retrieved document — are handled well by small, fast, cheap models. Frontier models
earn their price on genuinely hard reasoning, long-horizon agentic work, and nuanced writing.

**"Use the most capable model" is the most common wrong answer on this exam.** When you see
it as an option, check whether the scenario mentions cost, latency or scale — if it does,
it's a trap.

A pattern worth knowing: **route by difficulty.** Send everything to a cheap model, detect
low confidence or complexity, escalate only those to an expensive one. Bedrock's
**Intelligent Prompt Routing** does this for you; you can also do it by hand. Most traffic
is easy, so most traffic should be cheap.

> **Remember:** you don't hire a principal engineer to reset passwords.

---

## How to actually choose: the golden set

Do not choose by vibes, and do not choose by reading a blog post. Build a tiny harness. It
takes an hour and you will reuse it for the rest of this plan.

**1. Write 20 questions with known-good answers.** Use your own notes repo — you know when
an answer is right. Include:
- 12 normal cases
- 4 edge cases (ambiguous, multi-part)
- 4 cases where the honest answer is **"I don't know"** ← these catch hallucination, and
  they're the ones everyone forgets

**2. Run all 20 against each candidate model.** Same prompt, same parameters, temperature 0.

**3. Record four columns per model:**

| Model | Correct /20 | Avg input+output tokens | Avg latency | Est. $/1k requests |
|---|---|---|---|---|
| small | 16 | 2,100 / 240 | 0.9 s | $0.42 |
| mid | 18 | 2,100 / 310 | 1.8 s | $2.10 |
| frontier | 19 | 2,100 / 380 | 3.4 s | $9.80 |

**4. Now the decision is arithmetic, not opinion.** Is that 19th correct answer worth 23×
the cost and 4× the latency? Sometimes yes. Usually no. **The point is that you can now
answer the question instead of guessing.**

> **Remember:** the golden set is the single highest-leverage hour in this whole plan. It
> turns every later decision — model, prompt, chunk size, temperature — from an argument
> into a measurement. You'll formalise it into a real eval suite in Week 12; start it now.

**Bedrock Evaluations** is the managed version of this (automatic metrics, human review,
LLM-as-a-judge). Know it exists and what the three modes are — you'll use it properly in
Week 12. For now, 30 lines of your own code teaches you more.

---

## Model lifecycle — the bit people forget

Models are **versioned and they get deprecated.** AWS publishes end-of-life dates and the
model you launched on will eventually stop being served.

So:

- **Pin the model ID in configuration, never hardcode it** in application code
- Keep your golden set — it's how you'll validate the replacement in an afternoon instead
  of a fortnight
- This is exactly why **Converse** matters: switching models is a config change, not a refactor

> **Remember:** treat the model like a managed database engine version. It will be upgraded,
> whether or not you planned for it. Have the regression test ready.

---

## Exam question shapes

Scenario questions almost always hide the answer in a **constraint**. Train yourself to find it:

| The scenario mentions… | It's steering you toward |
|---|---|
| "millions of requests", "minimise cost" | A **smaller model**, batch, or routing |
| "sub-second response", "interactive" | Small model + **streaming** |
| "data must stay in region X" | Region availability; scrutinise cross-region profiles |
| "must not fabricate", "must cite" | RAG + grounding + guardrails — **not** a bigger model |
| "predictable high volume" | **Provisioned throughput** |
| "overnight", "not time-sensitive" | **Batch inference** |
| "images and text together" | Multimodal, or a chain — check what else is constrained |

**Read the last line of the question first**, then hunt the scenario for the constraint. Two
options will usually both "work"; only one respects the constraint.

---

## 🔨 Build

Extend last week's CLI into a **model bench**:

- [ ] `bench.js` reads a JSON file of 20 questions + expected answers
- [ ] Runs all of them against 2–3 models (`--models a,b,c`)
- [ ] Records tokens in/out, latency, and `stopReason` per call
- [ ] Prints the comparison table above
- [ ] Computes estimated cost per 1,000 requests from real published prices

Grading can be crude for now — substring match on a key fact, or eyeball it. You'll do this
properly in Week 12.

**Then run it.** Seeing a 20× price spread for a 2-answer quality difference, on your own
data, is the moment model selection stops being abstract.

---

## ✍️ Write your note

`02-model-selection-notes.md`:
- Your bench table, with real numbers
- Which model you'd pick for the notes assistant, and the sentence justifying it
- The biggest surprise in the numbers

---

## ✅ Checkpoint

1. Name the seven selection criteria without looking.
2. In a RAG system, which usually costs more in total — input or output tokens? Why?
3. Why can a chat UI tolerate a slow model when a Step Functions task can't?
4. What's wrong with "use the most capable model to be safe"?
5. A scenario says data must not leave `eu-west-1`. Which criterion just became decisive?
6. Why pin the model ID in config rather than code?

---

*Next: `03-prompting.md` — getting the model you chose to actually behave.*
