# AI with AWS (Backend Engineer View)

Notes on building AI features on AWS — what each service actually is,
when to reach for it, and the pitfalls that cost time.

> Working toward **AWS Certified Generative AI Developer – Professional (AIP-C01)**.
> Start with the [learning plan](./learning-plan.md) — this file is the topic index.

---

## The mental model

AWS does not give you "an AI". It gives you three separate things:

1. **A model endpoint** — something that turns text/images into tokens (Bedrock, SageMaker).
2. **A retrieval layer** — something that finds the right context to feed the model (Knowledge Bases, OpenSearch, Kendra).
3. **An orchestration layer** — something that decides what happens next (Lambda, Step Functions, Bedrock Agents).

Most "AI project" complexity lives in layers 2 and 3, not layer 1.

---

## Amazon Bedrock

*Managed, serverless access to foundation models. No infrastructure to run.*

- [ ] Model access & region availability (models are region-scoped)
- [ ] `InvokeModel` vs `InvokeModelWithResponseStream`
- [ ] The Converse API (unified message format across providers)
- [ ] Tool use / function calling
- [ ] Guardrails (content filters, denied topics, PII redaction)
- [ ] Knowledge Bases (managed RAG)
- [ ] Bedrock Agents (managed action loop)
- [ ] Provisioned throughput vs on-demand — when the cost flips
- [ ] Pitfalls: throttling, token quotas, cold-start on large prompts

---

## Amazon SageMaker

*When you need your own model, your own weights, or your own training loop.*

- [ ] Studio / notebooks vs training jobs
- [ ] Real-time endpoints vs serverless inference vs batch transform
- [ ] Bring-your-own-container
- [ ] Fine-tuning and where it beats prompting
- [ ] Pitfalls: endpoints bill by the hour whether or not you call them

---

## RAG on AWS

- [ ] Chunking strategy (the part that actually decides quality)
- [ ] Embeddings — Titan Embeddings, Cohere on Bedrock
- [ ] Vector stores: OpenSearch Serverless, Aurora pgvector, Pinecone
- [ ] Amazon Kendra vs a raw vector search
- [ ] Re-ranking and why top-k alone is not enough
- [ ] Evaluating retrieval separately from generation

---

## Orchestration

- [ ] Lambda for single-turn inference (watch the 15-min ceiling)
- [ ] Step Functions for multi-step / human-in-the-loop flows
- [ ] SQS + DLQ for retryable model calls
- [ ] Streaming responses to the client (Lambda response streaming, API Gateway limits)
- [ ] Idempotency — LLM calls are expensive to accidentally repeat

---

## AI-adjacent managed services

- [ ] **Textract** — document/OCR extraction
- [ ] **Comprehend** — entities, sentiment, PII detection
- [ ] **Transcribe / Polly** — speech in and out
- [ ] **Rekognition** — image and video analysis
- [ ] **Translate**

Rule of thumb: if a managed service already solves it, do not prompt an LLM for it.
It is cheaper, deterministic, and testable.

---

## Security & governance

- [ ] IAM policies for `bedrock:InvokeModel` (scope to specific model ARNs)
- [ ] VPC endpoints / PrivateLink — keeping traffic off the public internet
- [ ] KMS encryption for prompts and outputs at rest
- [ ] CloudTrail: what is and is not logged for model invocations
- [ ] Model invocation logging to S3 / CloudWatch
- [ ] Data residency and whether prompts are used for training (they are not, on Bedrock)

---

## Cost

- [ ] Priced per input + output token — output is usually the expensive half
- [ ] Prompt caching
- [ ] Batch inference for non-interactive workloads
- [ ] Budget alarms before, not after, the first big bill

---

## Open questions / to revisit

-
