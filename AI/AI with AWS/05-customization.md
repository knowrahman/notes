# 05 — Customization (Week 5)

Fine-tuning, continued pre-training, distillation — and the decision table the exam keeps
asking about. This closes Domain 1.

---

## The four ways to change a model's behaviour

Learn this table. Then learn it again. Scenario questions in Domain 1 are very often "which
of these four?" wearing a costume.

| Approach | Changes | Needs | Cost | Reach for it when |
|---|---|---|---|---|
| **Prompt engineering** | How it responds *this time* | Nothing | Free, instant | Always try first |
| **RAG** | What it **knows right now** | Documents | Low, ongoing | Private, current, or changing facts |
| **Fine-tuning** | How it **behaves by default** | Labelled input→output pairs | High | Consistent format, tone or task behaviour |
| **Continued pre-training** | Its **understanding of a domain** | A large *unlabelled* corpus | Very high | Specialist vocabulary the base model doesn't know |

### The decision path

```
Is the output nearly right, just needs steering?
    └─ YES → Prompt engineering. Stop here.

Does it need facts it doesn't have — private, current, or changing?
    └─ YES → RAG. Stop here.

Does it need to consistently behave, format or sound a certain way,
and you have hundreds of good examples?
    └─ YES → Fine-tuning.

Does it not understand the domain's *language* at all
(specialist jargon, an under-represented language)?
    └─ YES → Continued pre-training. Rare. Expensive.
```

**Work top to bottom. Never skip a step.** Reaching for fine-tuning when a better prompt
would do is the most common expensive mistake in this field, and the most common wrong
answer on this exam.

> **Remember:** RAG changes what's **on the desk**. Fine-tuning changes what's **in the
> habits**. Continued pre-training changes what **language it thinks in**.

### Fine-tuning vs. continued pre-training — the discriminator

These two get confused constantly. The tell is **the data**:

| | Fine-tuning | Continued pre-training |
|---|---|---|
| Data | **Labelled** pairs: input → desired output | **Unlabelled** raw text |
| Volume | Hundreds to low thousands | Very large corpora |
| Teaches | A *task* or a *style* | A *vocabulary* and a *domain* |
| Example | "Write support replies in our voice" | "Understand clinical trial terminology" |

**If the scenario mentions labelled examples → fine-tuning. If it mentions a large body of
unlabelled domain documents → continued pre-training.**

---

## How fine-tuning actually works: LoRA

A model has billions of parameters. Retraining all of them needs a data-centre and would
cost more than the original training run. So nobody does that.

**LoRA — Low-Rank Adaptation — freezes the original model and trains a small set of extra
matrices ("adapters") injected alongside its layers.**

```
     Base model  (frozen — billions of weights, untouched)
          +
     LoRA adapter (trained — a tiny fraction of that)
          ↓
     Your customised model
```

What that buys you:

- **Cheap and fast** — you're training a small fraction of the parameters
- **Small artifacts** — the adapter is megabytes, not the whole model
- **Swappable** — one base model, many adapters for different tasks
- **Non-destructive** — the base model is untouched, so nothing is "forgotten"

> **Remember:** **LoRA is a diff, not a fork.** You don't copy the whole repository to change
> one behaviour — you keep the base and apply a small patch on top. Same instinct, same
> reasons.

You won't implement LoRA. You need to know *what it is*, *why fine-tuning is affordable at
all*, and that the adapter is separate from the base model.

---

## Distillation

**Use a big expensive model to teach a small cheap one.**

```
Teacher (large, accurate, expensive)
   ↓  generates high-quality answers for your task
Training data
   ↓  fine-tunes
Student (small, fast, cheap) → nearly teacher quality, on this narrow task
```

**Bedrock Model Distillation** automates it: you supply prompts, it uses the teacher to
generate responses and produces a fine-tuned student.

**When it's the right answer:** you've proven a large model handles the task, but its cost or
latency doesn't survive production volume. Distillation gets you most of the quality at a
fraction of the cost — **on that narrow task only**. The student doesn't become generally
smart.

> **Remember:** distillation is an apprenticeship. The apprentice learns one job very well,
> not everything the master knows.

**Exam tell:** "works well but too expensive/slow at scale" → distillation.

---

## Fine-tuning in practice

### The data

JSONL, one example per line — prompt and desired completion:

```jsonl
{"prompt": "Customer: my order is late", "completion": "I'm sorry about the delay..."}
{"prompt": "Customer: how do I return this?", "completion": "You can start a return..."}
```

Split into **training** and **validation** sets. Validation is how you detect overfitting —
the model memorising your examples instead of learning the pattern.

### Quality beats quantity, decisively

- A few hundred *excellent* examples beat thousands of mediocre ones
- **The model learns your mistakes as faithfully as your intentions.** Inconsistent labels
  in, inconsistent behaviour out
- Cover your edge cases — a fine-tuned model handles what it saw

### Where the real cost hides

> ⚠️ **This is the one that catches everyone.** Training is a one-off bill. **Serving is not.**
>
> A custom model on Bedrock generally can't be called on-demand like a base model — it
> typically needs **provisioned throughput**, which bills **by the hour, continuously,
> whether or not anyone calls it.**
>
> So a fine-tuned model idling overnight still costs money. A base model idling costs nothing.
> Check the serving requirements for your model *before* you commit to fine-tuning — the
> hosting bill usually dwarfs the training bill.

**That single fact reframes the whole decision.** RAG's cost scales with use; fine-tuning's
cost largely doesn't. At low or spiky volume, RAG usually wins on economics alone.

### When fine-tuning is the wrong answer

- **To add facts** → RAG. Fine-tuning is unreliable at recall and stale the moment facts change
- **To fix a badly-written prompt** → fix the prompt
- **When the data changes often** → you've signed up for a retraining treadmill
- **When you have fewer than a few hundred good examples** → not enough signal
- **When you haven't tried prompting and RAG first** → you're paying to skip the cheap steps

---

## Data pipelines for GenAI

Everything above needs data, laid out deliberately:

- **S3 layout** — separate raw, processed and training data. Version it; you must be able to
  say which data produced which model
- **Incremental sync** — reprocess what changed, not the whole corpus every night
- **Provenance** — track which documents went into which model or index. When something goes
  wrong, "where did this come from?" must be answerable
- **Strip PII before it reaches training data.** Anything in a fine-tuning set is baked into
  model behaviour, and you cannot un-bake it

### Bedrock Data Automation

Extracts **structured output from unstructured files** — documents, images, audio, video —
using configurable blueprints, rather than you chaining Textract → prompt → parse by hand.

**When it wins:** standard document-extraction work at volume. It's purpose-built,
consistent, and you're not maintaining prompt glue.
**When to hand-roll:** genuinely unusual formats or logic the blueprints don't express.

> Tie back to note 00's rule: **if a managed service already solves it, don't prompt an LLM
> for it.** Cheaper, deterministic, testable.

### Bedrock Marketplace

Access to models beyond the built-in catalogue — specialised and third-party. Know it exists
and that it's the answer when a scenario needs a niche or domain-specific model.

---

## Compliance and data handling

Domain 1 includes compliance, and these are the facts to have straight:

**What Bedrock does with your data**
- Your prompts and completions are **not used to train the base models**
- Your fine-tuning data stays yours; the resulting custom model is private to your account
- Encrypt custom models and training data with **your own KMS key**

**Data residency**
- Inference runs in the region you call — that's your primary residency control
- ⚠️ **Cross-region inference profiles can process requests in another region in the group.**
  Fine for resilience, potentially unacceptable for residency. **This is a compliance
  decision, not a performance one**, and it's a favourite exam discriminator

**PII**
- Strip or mask it **before** it reaches the model wherever you can
- Guardrails can detect and redact it in flight (Week 9)
- Model invocation logging captures prompts and completions — useful for audit, and itself a
  store of sensitive data that needs encrypting and access-controlling

> **Remember:** the region you call is your residency answer — **unless** a cross-region
> profile quietly widened it.

---

## 🔨 Build

Fine-tuning is expensive to practise, so **do the analysis, not the training** — that's the
exam-relevant skill anyway.

- [ ] Take three scenarios and write the one-sentence justification for each:
      1. A chatbot answering from a policy wiki that changes weekly
      2. Support replies that must match a house voice across 50 agents
      3. A model that must understand your industry's specialist jargon
- [ ] For your notes assistant: **argue in writing why RAG beats fine-tuning here.** If you
      can write that paragraph convincingly, you understand Domain 1
- [ ] Price it out: estimate a month of your RAG assistant vs. a month of provisioned
      throughput for a fine-tuned equivalent. **Real numbers.** The gap is the lesson
- [ ] *Optional, if budget allows:* a tiny fine-tune on ~200 examples — then **delete the
      provisioned throughput immediately afterwards**

---

## ✍️ Write your note

`05-customization-notes.md`:
- The four-option decision table, rewritten in your own words
- Your RAG-vs-fine-tune cost comparison, with numbers
- The one sentence that best distinguishes fine-tuning from continued pre-training

---

## ✅ Checkpoint

1. Name the four customization approaches and what each one changes.
2. In what order do you try them, and why that order?
3. What's the data-shaped difference between fine-tuning and continued pre-training?
4. Explain LoRA in one sentence to a backend engineer.
5. What is distillation for, and what's the exam tell for it?
6. Why is a fine-tuned model's *hosting* cost usually the deciding factor?
7. Give three situations where fine-tuning is the wrong answer.
8. A scenario requires data never leave `eu-central-1`. What must you check besides the region you call?

---

## 🎉 Domain 1 complete

That's **31% of the exam** — the largest block — covered: model selection, prompting, RAG,
and customization.

Before moving on, spend an hour going back over notes 02–05 and your own write-ups. Domain 1
underpins everything that follows; the agents in Phase 3 are models you chose, prompted and
grounded.

*Next: `06-tool-use.md` — Phase 3 opens. Domain 2, and the model starts calling your code.*
