# Learning Plan — AWS Certified Generative AI Developer, Professional (AIP-C01)

**Starting point:** strong backend / AWS serverless, *zero* AI background.
**Assumed pace:** ~8–10 hrs/week → ~14 weeks. Halve the weeks at 16–20 hrs/week; add ~4 if under 6.

---

## The exam (verify against the official guide before booking)

| | |
|---|---|
| Code | AIP-C01 |
| Format | 75 questions, multiple choice / multiple response |
| Time | 180 minutes |
| Score | 100–1000 scaled, **750 to pass**, compensatory (no per-domain minimum) |
| Cost | $300 USD |
| Audience | Developers with ~2+ years of cloud experience |

### Domain weights — this is your study budget

| Domain | Weight | Weeks here |
|---|---|---|
| 1. Foundation Model Integration, Data Management, Compliance | **31%** | 3–5 |
| 2. Implementation and Integration | **26%** | 6–8 |
| 3. AI Safety, Security, and Governance | **20%** | 9–10 |
| 4. Operational Efficiency and Optimization | **12%** | 11–12 |
| 5. Testing, Validation, and Troubleshooting | **11%** | 11–12 |

Domains 1 + 2 are **57% of the exam**. Spend your time accordingly — do not let the
security domain (which feels comfortable to an AWS person) eat the hours that
belong to model selection and RAG.

---

## Read this before Week 1

**Your gap is not AWS. It is AI vocabulary.**

You already know IAM, Lambda, DynamoDB, API Gateway, Step Functions. That is most
of the "AWS" in this exam. What will hurt is a sentence like "we'll set the
temperature to 0.2 and use top-p sampling with a 200k context window and cosine
similarity over Titan embeddings" landing in week one — if none of those words mean
anything, you spend your study time transcribing instead of understanding.

So: **Week 1 is pure vocabulary, no AWS.** Everything after that is you doing what
you already do well — wiring services together — with a new component in the middle.

**On the AI Practitioner cert (AIF-C01):** you need its *content*, not its
*certificate*. Week 1–2 below covers the same ground. Sit it only if you want a
cheap confidence checkpoint ~week 6; skip it if you'd rather keep momentum.

**On the LLM being unreliable:** this is the one genuinely new engineering idea.
Every backend instinct you have assumes a function returns the same thing for the
same input. Foundation models do not. Nearly every "best practice" in this exam —
guardrails, validation, evals, retries, structured output, human-in-the-loop —
exists to build reliable systems on an unreliable component. Once that clicks, most
answers become obvious.

**You have a head start you may not realise:** `AI/agentic AI/agentic AI.md` in this
repo is already ~2,000 lines on exactly the mental model Domain 2 tests. Reread it
in Week 6 — you wrote most of the agentic AI answer already.

---

## How to use video material

Do **not** watch front-to-back and hope.

1. **Section by section**, following the material's own order.
2. After each section, **close the video and build the smallest possible thing**
   that uses the idea. 60% hands-on, 40% video is the right ratio at Professional level.
3. **Write a note in this repo** for each topic — same style as your existing notes.
   If you can't write the note, you didn't learn the section.
4. **Do not pause to master everything.** Some things only click on the second pass
   in Week 13. Mark them `TODO` and move.

---

## Breadth first, then depth

Most video material front-loads **breadth** — one fast sweep across Bedrock overview,
fine-tuning, RAG, Knowledge Bases, chunking, guardrails, prompt management, Flows and
prompt engineering, often inside the opening hours. That's Phases 2–4 of this plan
compressed into a couple of hours.

**That isn't a conflict with this plan — the two orderings do different jobs:**

| | A breadth sweep | This plan |
|---|---|---|
| Job | The **map** — see the whole terrain fast | The **depth pass** — one topic per week, built and written up |
| Pace | Watch it straight through | A week per cluster |
| Output | Recognition | Working code and your own notes |

**So do the breadth sweep in one or two sittings, right after Week 2.** No notes, no
pausing to master anything, no building. You're buying a mental index, so that when Week 4
spends five days on chunking you already know where it sits. Then work the phases below as
the depth pass.

> ### 💸 Read this before your first Knowledge Base lab
> **The OpenSearch Serverless collection a Knowledge Base creates bills continuously**,
> with a minimum capacity floor, whether or not you ever query it. Left running it costs
> hundreds of dollars a month, and it is by far the most expensive mistake available in
> this whole syllabus.
>
> **Delete the collection the moment each lab ends.** Put a calendar reminder on the same
> day. Check Cost Explorer the morning after any Knowledge Base lab.

---

## The spine project

One project runs the whole plan, growing each phase: **an assistant over this notes repo.**

You have a public markdown corpus with real structure (`backend/`, `cloud/`, `iot/`…).
It is an ideal RAG dataset and you already know whether an answer is right.

| Phase | What the project becomes |
|---|---|
| 1 | CLI that sends a prompt to a model and prints the reply |
| 2 | Answers questions about *one* note pasted into the prompt |
| 3 | Real RAG — embeds all notes, retrieves, cites the source file |
| 4 | An agent that can search notes, fetch a file, and open a GitHub issue |
| 5 | Guardrailed, IAM-scoped, logged, private-networked |
| 6 | Load-tested, cost-tracked, with an eval suite scoring its answers |

By exam day you will have built one of nearly every service the exam asks about.

---

## Phase 0 — AI from zero (Week 1) · *no AWS*

**Goal:** stop being surprised by words.

- What a foundation model actually is; training vs. inference
- **Tokens** — why cost, limits, and latency are all measured in them
- **Context window** — the working memory, and why it's the central constraint
- **Embeddings** — text as vectors; cosine similarity; why this powers search
- **Inference parameters** — temperature, top-p, top-k, max tokens, stop sequences
- Why output is non-deterministic, and what a **hallucination** actually is
- **Prompting vs. RAG vs. fine-tuning** — the three ways to make a model know your data
- Modalities: text, image, embedding, multimodal
- What "an agent" means (tool use in a loop)

**Build:** nothing. Read, watch, and write the glossary.
**Deliverable:** `AI/AI with AWS/00-glossary.md` — every term above in your own words.
**Checkpoint:** explain to a non-technical friend why an LLM can confidently state a
wrong fact. If you can't, stay here another few days.

---

## Phase 1 — First contact with Bedrock (Week 2)

**Goal:** call a model from your own code. Break the "AI is magic" spell early.

- Bedrock model access — requesting models, **region availability** (models are region-scoped)
- The console playground — feel temperature and max-tokens by moving them
- **Converse API** vs `InvokeModel` — and why Converse is the one to learn
- `ConverseStream` / `InvokeModelWithResponseStream` — token-by-token output
- System prompt vs user message vs assistant turn; multi-turn conversation state
- Reading the response: content, stop reason, **token usage**

**Build:** a Node or .NET CLI — `ask "question"` → streamed answer. Print token counts.
**Deliverable:** `01-bedrock-basics.md`
**Checkpoint:** you can explain the exact shape of a Converse request and response
without looking, and you've seen your own token bill.

---

## Phase 2 — Domain 1: models, data, compliance (Weeks 3–5) · **31%**

The biggest block. Give it the full three weeks.

### Week 3 — Choosing and steering a model
- Model selection criteria: modality, context window, latency, throughput, **cost per
  input vs. output token**, licensing, region
- **Multimodal models and pipelines** — text + image + audio + video in, and when a
  multimodal model beats a chain of single-purpose ones
- When a smaller/faster model is the correct answer (most of the time)
- Prompt engineering that's actually tested: role prompting, few-shot examples,
  chain-of-thought, delimiters, output-format instruction
- **Structured output** — forcing JSON, schema validation, what to do when it's malformed
  *(you've already done this in your agentic AI notes — reuse it)*
- Prompt templates and versioning; **Bedrock Prompt Management** — versions, aliases, and
  shifting traffic between prompt versions without a redeploy

### Week 4 — Retrieval (RAG)
- Why RAG exists: private/fresh data without retraining
- **Chunking** — size, overlap, semantic vs. fixed; *the single biggest quality lever*
- Embedding models; dimensions; cost of embedding a corpus
- Vector stores: **OpenSearch Serverless**, Aurora **pgvector**, Neptune Analytics, third-party
- **Bedrock Knowledge Bases** — managed ingestion, sync, retrieve vs. `RetrieveAndGenerate`
- Metadata filtering; hybrid (keyword + semantic) search; **re-ranking**
- Kendra vs. a raw vector search — when each wins
- Why top-k alone is not enough

### Week 5 — Customization and data
- **Fine-tuning vs. continued pre-training vs. distillation vs. RAG** — decision criteria
  (this comparison is exam gold; make a table and memorise it)
- **How fine-tuning actually works: LoRA (Low-Rank Adaptation)** — why you train a small
  adapter instead of the whole model, and what that buys in cost and time
- Training data format, quality, volume; where fine-tuning is the *wrong* answer
- Model evaluation before selection
- Data pipelines: S3 layout, ingestion, sync, incremental updates
- **Bedrock Data Automation** — structured output from documents, images, audio and
  video; where it beats hand-rolling Textract + prompting
- **Bedrock Marketplace** — reaching models beyond the built-in catalogue
- **Compliance:** data residency, region selection, what Bedrock does and doesn't do
  with your prompts, PII handling before it reaches the model

**Build:** the spine project becomes real RAG over this repo — chunk, embed, store,
retrieve, cite the source file. Then rebuild it with a Knowledge Base and compare.
**Deliverables:** `02-model-selection.md`, `03-prompting.md`, `04-rag.md`, `05-customization.md`
**Checkpoint:** given a business scenario, you pick RAG / fine-tune / prompt and defend it in
two sentences. Practise this — it is the exam's favourite question shape.

---

## Phase 3 — Domain 2: implementation and integration (Weeks 6–8) · **26%**

**Reread `AI/agentic AI/agentic AI.md` before starting.** Much of this is already yours.

### Week 6 — Tool use
- Function calling / tool use: schema definition, the request-execute-return loop
- Why the model *proposes* and your code *decides* — the supervision principle
- Parallel tool calls, tool errors, retries

### Week 7 — Agents
- **Bedrock Agents:** action groups, OpenAPI schemas, Lambda executors
- Session state, memory, prompt overrides
- Agents + Knowledge Bases together
- Multi-agent collaboration: supervisor/sub-agent patterns
- Model Context Protocol (MCP) — how tools get exposed to models
- When an agent is over-engineering and a plain chain is right

**Bedrock AgentCore** — the production agent platform. Modular services you can adopt
individually, and the answer to "how do I actually run an agent in production":

| Service | What it solves |
|---|---|
| **Runtime** | Serverless hosting with micro-VM **session isolation** |
| **Gateway** | Turns your APIs and Lambdas into **MCP tools** for the agent |
| **Memory** | Context that survives across sessions |
| **Identity** | Who the user is; which third-party tokens the agent may use on their behalf |
| **Observability** | What the agent actually did (OpenTelemetry) |
| **Code Interpreter** | Sandboxed code execution |
| **Browser** | Web automation |
| **Evaluations** | Whether the agent is doing a good job |

Know **Bedrock Agents vs. AgentCore**: the managed single-agent construct vs. the
infrastructure layer for running agents — including ones built with other frameworks — at
production scale. Expect scenario questions that hinge on session isolation, identity
delegation, or cross-session memory; those point at AgentCore.

### Week 8 — Application architecture
- **Bedrock Flows** — the visual/API workflow builder chaining prompts, Knowledge Bases,
  Agents, Guardrails, Lambdas and conditional logic into one invocable flow. Know when a
  Flow beats Step Functions (Flows for GenAI-native chaining; Step Functions for general
  distributed orchestration, long waits and human-in-the-loop)
- Lambda for inference (mind the 15-min ceiling and the payload limits)
- **Step Functions** for multi-step, long-running, human-in-the-loop flows
- SQS + DLQ for retryable model calls; **idempotency** (LLM calls are expensive to repeat)
- **Streaming to the client:** Lambda response streaming, API Gateway limits, AppSync,
  WebSockets — know which one supports streaming and which doesn't
- Async / batch inference jobs
- Conversation state in DynamoDB *(home turf — lean on it)*

**Build:** the assistant becomes an agent that can search notes, read a file, and open a
GitHub issue. Then orchestrate a multi-step version in Step Functions.
**Deliverables:** `06-tool-use.md`, `07-bedrock-agents.md`, `08-orchestration.md`
**Checkpoint:** you can draw the full request path from client → streamed answer, and say
which service breaks first under load.

---

## Phase 4 — Domain 3: safety, security, governance (Weeks 9–10) · **20%**

Your most comfortable domain. Don't over-invest — but the *AI-specific* half is new.

### Week 9 — Safety (the new half)
- **Bedrock Guardrails:** content filters, denied topics, word filters, **PII detection and
  redaction**, contextual grounding checks
- Guardrails on input vs. output, and applying one to an agent or Knowledge Base
- **Automated Reasoning checks** — formal/logical verification of claims against policy,
  a distinctly AWS answer to hallucination. Know what it can and can't prove
- **Token-level redaction** — masking sensitive spans rather than blocking the response
- **Prompt injection and jailbreaking** — including *indirect* injection via retrieved
  documents *(directly relevant: your RAG corpus is public markdown)*
- Responsible AI: bias, fairness, transparency, **AWS AI Service Cards**
- Human-in-the-loop review; Amazon A2I

### Week 10 — Security and governance (familiar half, AI flavour)
- IAM: scoping `bedrock:InvokeModel` to **specific model ARNs**, not `*`
- **VPC endpoints / PrivateLink** — keeping inference off the public internet
- **KMS** for prompts, outputs, custom models, Knowledge Base data at rest
- **CloudTrail** — what is and isn't captured for invocations
- **Model invocation logging** to S3 / CloudWatch — the audit trail
- **AWS Well-Architected Tool — Generative AI Lens**: the six-pillar framing applied to
  GenAI workloads. Exam scenarios love "which pillar / which best practice"
- Cost allocation tags, multi-account patterns, SCPs on model access
- Secrets for third-party model providers

**Build:** put a guardrail in front of the assistant. Try to break it with an injection
hidden inside a note file. Lock the Lambda role to one model ARN. Turn on invocation logging.
**Deliverables:** `09-guardrails-safety.md`, `10-security-governance.md`
**Checkpoint:** you can name the mitigation for indirect prompt injection through RAG.

---

## Phase 5 — Domains 4 & 5: operations and evaluation (Weeks 11–12) · **23%**

### Week 11 — Operational efficiency (12%)
- **Token economics:** output tokens usually dominate cost; prompt compression
- **Prompt caching** — what's cacheable and the savings
- **Provisioned throughput vs. on-demand** — where the cost curve crosses
- **Batch inference** for non-interactive work
- **Intelligent Prompt Routing** — sending easy requests to a cheaper model automatically
- **Cross-region inference profiles** — capacity and resilience, and the reason a bare
  model ID sometimes fails with an on-demand throughput error
- Latency: model choice, streaming, first-token vs. total time, cold starts
- Quotas and throttling; retries with exponential backoff; cross-region inference profiles
- Observability: CloudWatch metrics, invocation logs, tracing a request end to end
- Budgets and alarms *before* the first big bill

### Week 12 — Testing and troubleshooting (11%)
- **Evaluation:** Bedrock model evaluation, automatic vs. human, **LLM-as-a-judge**
- Evaluating **retrieval separately from generation** — the key diagnostic split
- RAG metrics: faithfulness, answer relevance, context precision/recall
- Building a regression suite for prompts; treating prompts as versioned code
- Troubleshooting: bad answer → is it retrieval, prompt, model, or parameters?
- Debugging agents: wrong tool, bad arguments, infinite loops
- A/B testing models and prompts in production

**Build:** an eval suite — 30 questions about this repo with known answers. Score the
assistant. Change the chunk size, rerun, see the number move. Add cost and latency tracking.
**Deliverables:** `11-operations-cost.md`, `12-evaluation-testing.md`
**Checkpoint:** you have a number that says how good your system is, and you can move it deliberately.

---

## Phase 6 — Exam run (Weeks 13–14)

- **Week 13:** second pass over all your video material at 1.5–2×. It will feel easy —
  that's the point. Note every moment of hesitation; those are your weak spots.
- Take a **full practice exam under real conditions** (180 min, no pausing). Expect to fail
  the first one. That is normal and useful.
- Score by domain, drill the weakest, retake.
- **Week 14:** practice exams until consistently 80%+, then book.
- Reread your own notes the day before. Nothing new in the final 48 hours.

**Exam-day tactics:** questions are long scenarios — read the *last* line first to see what's
actually being asked, then reread the scenario for the constraints (cost? latency? compliance?
no internet access?). The constraint is almost always the discriminator between two plausible
answers. Flag and move on anything over ~2.5 minutes; you have 2.4 min/question average.

---

## Bedrock feature coverage map

Every Bedrock surface, and the week it lands in. Use this to check nothing has slipped —
if you meet something not on this list, add it.

| Bedrock feature | Covered in |
|---|---|
| Model access, regions, model IDs | Phase 1 · Wk 2 |
| **Converse API** / ConverseStream / InvokeModel | Phase 1 · Wk 2 |
| Inference parameters, streaming, multi-turn | Phase 1 · Wk 2 |
| Model selection & comparison criteria | Phase 2 · Wk 3 |
| Prompt Management — versions, aliases, traffic | Phase 2 · Wk 3 |
| **Knowledge Bases** (managed RAG) | Phase 2 · Wk 4 |
| Embeddings, vector stores, chunking, re-ranking | Phase 2 · Wk 4 |
| Fine-tuning / continued pre-training / **distillation** | Phase 2 · Wk 5 |
| **Data Automation**, Marketplace | Phase 2 · Wk 5 |
| Tool use / function calling | Phase 3 · Wk 6 |
| **Bedrock Agents** — action groups, multi-agent | Phase 3 · Wk 7 |
| **AgentCore** — Runtime, Gateway, Memory, Identity, Observability | Phase 3 · Wk 7 |
| MCP | Phase 3 · Wk 7 |
| **Flows**, Step Functions, streaming to clients | Phase 3 · Wk 8 |
| **Guardrails**, prompt injection, responsible AI | Phase 4 · Wk 9 |
| IAM, PrivateLink, KMS, CloudTrail, invocation logging | Phase 4 · Wk 10 |
| Provisioned throughput, prompt caching, batch | Phase 5 · Wk 11 |
| Intelligent Prompt Routing, cross-region inference profiles | Phase 5 · Wk 11 |
| **Evaluations** — automatic, human, LLM-as-a-judge | Phase 5 · Wk 12 |
| RAG metrics, prompt regression testing | Phase 5 · Wk 12 |

**Deliberately not scheduled:** Bedrock Studio and other low-code/console-only surfaces —
worth 20 minutes of clicking, not a study week.

---

## Weekly rhythm

| | |
|---|---|
| Mon–Thu | ~1 hr/night — video + notes |
| Sat | 3–4 hrs — build the phase project |
| Sun | 1 hr — write up the week's note, review last week's |

Non-negotiables:
- **Build something every week.** Reading about Bedrock does not teach Bedrock.
- **Write the note.** Your repo is proof this works for you — keep doing it.
- **Set an AWS budget alarm in Week 2.** Bedrock is pay-per-token and a runaway loop is expensive.
- **Use the free/cheap models while learning.** Save the big ones for when it matters.

---

## Progress tracker

- [ ] Phase 0 — Glossary written (Wk 1)
- [ ] Phase 1 — First Bedrock call from own code (Wk 2)
- [ ] Phase 2 — RAG over this repo, working and cited (Wk 3–5)
- [ ] Phase 3 — Agent with tools + Step Functions flow (Wk 6–8)
- [ ] Phase 4 — Guardrails, IAM scoping, logging (Wk 9–10)
- [ ] Phase 5 — Eval suite with a real score (Wk 11–12)
- [ ] Phase 6 — Practice exams 80%+ (Wk 13–14)
- [ ] Exam booked
- [ ] Passed

---

## Sources

- [AWS Certified Generative AI Developer – Professional (AWS)](https://aws.amazon.com/certification/certified-generative-ai-developer-professional/)
- [Official AIP-C01 exam guide (AWS docs)](https://docs.aws.amazon.com/aws-certification/latest/ai-professional-01/ai-professional-01.html)
- [AIP-C01 study path (Tutorials Dojo)](https://tutorialsdojo.com/aws-certified-generative-ai-developer-professional-study-path-aip-c01-exam-guide/)

> Domain weights above came via secondary sources — re-check them against the official
> PDF exam guide when you start, and adjust the week allocation if AWS has revised them.
