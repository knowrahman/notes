# 01 — Bedrock Basics (Week 2)

**Goal:** call a foundation model from your own code, and break the "AI is magic" spell.

By the end of this week you'll have a working CLI, you'll have seen your own token counts,
and every term from the glossary will have turned into something you actually typed.

---

## What Bedrock actually is

**Bedrock is a managed API in front of many companies' foundation models.**

That's it. No servers, no GPUs, no model downloads. You call an AWS API, you get tokens
back, you're billed per token. Models from Anthropic, Meta, Mistral, Cohere, Amazon and
others all sit behind the same endpoint.

> **Remember:** Bedrock is to foundation models what RDS is to databases — AWS runs the
> engine, you just connect to it. And like RDS, the engines come from different vendors.

---

## Step 1 — Model access (do this first, it blocks everything)

**Nothing works until you request access to a model.** This catches everyone on day one.

1. Bedrock console → **Model access** (left sidebar)
2. **Modify model access** → tick the models you want
3. Submit. Most are approved instantly; some ask a few questions about your use case

### Two things that will trip you up

**Access is per-region.** You enable a model in `us-east-1`, then run your code against
`ap-south-1`, and get an error that reads like the model doesn't exist. It exists — you
just didn't enable it *there*.

**Not every model exists in every region.** Newer models land in `us-east-1` and
`us-west-2` first. Check what's actually available where you are:

```bash
aws bedrock list-foundation-models \
  --region us-east-1 \
  --query 'modelSummaries[].modelId' \
  --output table
```

Run this now and copy a model ID out of it. **Use a real ID from your own account** rather
than one from a blog post — they change, and a stale ID is a confusing error message.

> 💡 **Cheap while learning:** pick a small/fast model. You're testing plumbing, not
> intelligence. Save the big models for when the answer quality actually matters.

---

## Step 2 — Play in the console first (10 minutes, no code)

Bedrock console → **Playgrounds** → **Chat**.

Do the exercises from the end of the glossary here — move the temperature slider, set
max_tokens to 15 and watch it cut off. Ten minutes in the playground makes the next section
feel obvious instead of abstract.

The playground also shows you the **API request** it built. Look at it. That's the JSON
you're about to send yourself.

---

## Step 3 — The two APIs (and which one to learn)

Bedrock gives you two ways to call a model. This distinction is exam material.

### `InvokeModel` — the raw, model-specific one

Sends a JSON body in **whatever shape that particular model's vendor invented.**

```jsonc
// One vendor wants this:
{ "anthropic_version": "...", "messages": [...], "max_tokens": 500 }

// A different vendor wants this:
{ "inputText": "...", "textGenerationConfig": { "maxTokenCount": 500 } }

// A third wants something else again:
{ "prompt": "...", "max_gen_len": 500 }
```

Switch models and **you rewrite your request-building and response-parsing code.**

### `Converse` — the unified one ✅

One request shape that works across **every** model on Bedrock. AWS translates it to the
vendor's format for you.

```js
{
  modelId: "<any model>",
  messages: [{ role: "user", content: [{ text: "Hello" }] }],
  system:   [{ text: "You are a helpful assistant." }],
  inferenceConfig: { maxTokens: 500, temperature: 0 }
}
```

Change `modelId` to a completely different vendor's model. **Everything else stays the same.**

### Which to use

| Use | When |
|---|---|
| **`Converse`** | Default. Always. Learn this one. |
| `InvokeModel` | Only when you need a model-specific feature Converse doesn't expose |

> **Remember:** `Converse` is an ORM for foundation models. One interface, many backends,
> and you can swap the backend without rewriting your app. `InvokeModel` is raw SQL —
> more control, but now you're writing vendor-specific dialect.

**This is why Converse matters for real systems:** model choice is not a one-time decision.
A cheaper or better model ships every few months. With Converse, switching is a config
change. With InvokeModel, it's a refactor.

---

## Step 4 — Your first call

```bash
mkdir bedrock-cli && cd bedrock-cli
npm init -y
npm install @aws-sdk/client-bedrock-runtime
```

Credentials work exactly like every other AWS SDK call — env vars, a named profile,
or an IAM role. Nothing new here.

`ask.js`:

```js
import {
  BedrockRuntimeClient,
  ConverseCommand,
} from "@aws-sdk/client-bedrock-runtime";

const client = new BedrockRuntimeClient({ region: "us-east-1" });

const response = await client.send(new ConverseCommand({
  modelId: "PUT_A_REAL_MODEL_ID_HERE",

  // The conversation so far. Just one user turn for now.
  messages: [
    { role: "user", content: [{ text: "Explain what DynamoDB is in two sentences." }] },
  ],

  // Standing instructions — separate from the conversation.
  system: [
    { text: "You are a concise technical assistant. No preamble." },
  ],

  // The dials from the last note.
  inferenceConfig: {
    maxTokens: 500,
    temperature: 0,
  },
}));

console.log(response.output.message.content[0].text);
console.log("\n---");
console.log("stopReason:", response.stopReason);
console.log("tokens in:", response.usage.inputTokens);
console.log("tokens out:", response.usage.outputTokens);
console.log("latency:", response.metrics.latencyMs, "ms");
```

```bash
node ask.js
```

That's it. That's the whole thing. **You just did the AI part** — everything else in this
certification is engineering around this one call.

---

## Step 5 — Read the whole response, not just the text

Beginners grab `content[0].text` and ignore the rest. The rest is where your production
bugs live.

```jsonc
{
  "output": {
    "message": {
      "role": "assistant",
      "content": [{ "text": "DynamoDB is a fully managed NoSQL..." }]
    }
  },
  "stopReason": "end_turn",
  "usage": {
    "inputTokens": 28,
    "outputTokens": 47,
    "totalTokens": 75
  },
  "metrics": { "latencyMs": 892 }
}
```

### `stopReason` — always check it

Straight from the last note. On Bedrock the values you'll meet are:

| Value | Meaning | Action |
|---|---|---|
| `end_turn` | Finished naturally | ✅ Happy path |
| `max_tokens` | **Truncated.** Your ceiling was too low | Don't parse it. Raise the limit |
| `stop_sequence` | One of your stop strings fired | Expected — verify it wasn't accidental |
| `tool_use` | It wants you to run a function | Week 6 material |
| `guardrail_intervened` | A guardrail blocked it | Week 9 material |

```js
if (response.stopReason === "max_tokens") {
  throw new Error("Truncated — do not trust this output");
}
```

### `usage` — this is your bill

`inputTokens` + `outputTokens`, every single call. Log them from day one. When someone asks
"why is this feature costing so much", this is the only way to answer.

Remember: **output tokens usually cost several times more than input tokens.** A chatty
model is an expensive model.

---

## Step 6 — Streaming

The model generates one token at a time. Non-streaming means waiting for *all* of them
before the user sees *anything* — a 10-second wall of silence, then a wall of text.

Streaming shows tokens as they arrive. Same total time, but it *feels* instant.

```js
import { ConverseStreamCommand } from "@aws-sdk/client-bedrock-runtime";

const response = await client.send(new ConverseStreamCommand({
  modelId: "PUT_A_REAL_MODEL_ID_HERE",
  messages: [{ role: "user", content: [{ text: "Explain S3 storage classes." }] }],
  inferenceConfig: { maxTokens: 1000, temperature: 0 },
}));

for await (const event of response.stream) {
  // Each text fragment as it's produced
  if (event.contentBlockDelta) {
    process.stdout.write(event.contentBlockDelta.delta.text);
  }
  // Why it stopped
  if (event.messageStop) {
    console.log("\n\nstopReason:", event.messageStop.stopReason);
  }
  // Token counts arrive at the END — you can't know them upfront
  if (event.metadata) {
    console.log("tokens:", event.metadata.usage);
  }
}
```

> **Remember:** streaming doesn't make it faster, it makes it *feel* faster. Time-to-first-token
> drops from 10 seconds to under one. Total time is unchanged.

**The catch that matters later:** you can't validate what you haven't received yet. If you're
streaming to the user and the response turns out to be truncated or blocked, you've already
shown them half of it. Streaming and validation pull against each other — remember this for
the guardrails week.

---

## Step 7 — Multi-turn (statelessness, made real)

Time to *feel* the thing from the glossary. **Bedrock stores nothing.** There is no session,
no conversation ID. If you want the model to remember, you resend everything.

```js
// You own this array. Bedrock does not.
const messages = [];

async function chat(userText) {
  messages.push({ role: "user", content: [{ text: userText }] });

  const res = await client.send(new ConverseCommand({
    modelId: MODEL_ID,
    messages,                                  // ← the ENTIRE history, every time
    inferenceConfig: { maxTokens: 500, temperature: 0.3 },
  }));

  // Push the reply back on, or the next turn has amnesia
  messages.push(res.output.message);

  console.log(res.output.message.content[0].text);
  console.log(`[${res.usage.inputTokens} in / ${res.usage.outputTokens} out]`);
}

await chat("My name is Rahman.");
await chat("What is my name?");        // works — turn 1 was resent
```

**Now watch the input token count across turns.** It climbs every single time, because you're
resending a longer history on each call:

```
turn 1 → 12 in
turn 2 → 47 in
turn 3 → 98 in
turn 4 → 171 in      ← you are re-paying for the whole conversation, forever
```

That single observation explains a whole category of design decisions you'll meet later:
summarising old turns, truncating history, storing conversations in DynamoDB, and prompt
caching. **Go and watch those numbers climb.** It makes the idea permanent in a way reading
can't.

> **Remember:** the `messages` array is *your* state, in *your* database. Bedrock is a pure
> function: messages in, message out, nothing kept.

---

## Step 8 — System prompt vs. user message

| | Goes in | Contains |
|---|---|---|
| **System** | `system: [{ text }]` | Who the model is, rules, output format, tone |
| **User** | `messages[]` role `user` | The actual request |
| **Assistant** | `messages[]` role `assistant` | What it replied (you push this back) |

A system prompt worth stealing:

```js
system: [{
  text: `You are an assistant for a software engineering notes repository.

Rules:
- Answer ONLY from the documents provided in the user message.
- If the answer is not in them, say "I don't know" — never guess.
- Always cite the filename you used.
- Be concise. No preamble, no "Great question!".`
}]
```

Every line there is doing a job. "Only from the documents" and "never guess" are your first
hallucination defences, and you'll build on them properly in the RAG week.

> **Remember:** system = configuration, user = request. Same split as middleware vs. handler.

---

## Step 9 — The IAM policy

Home turf. The one thing to notice is that **`Converse` is authorised by the
`bedrock:InvokeModel` action** — there's no separate `bedrock:Converse` permission.

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "bedrock:InvokeModel",
      "bedrock:InvokeModelWithResponseStream"
    ],
    "Resource": "arn:aws:bedrock:us-east-1::foundation-model/MODEL_ID_HERE"
  }]
}
```

**Scope it to specific model ARNs, not `*`.** A wildcard lets any bug or any injected
instruction reach your most expensive model. This exact point comes back in the security
domain — start the habit now.

---

## Common errors (bookmark this)

| Error | What it really means |
|---|---|
| `AccessDeniedException` | You didn't request model access in the console — **or** your IAM policy is missing the action |
| `ValidationException: model identifier is invalid` | Wrong region, typo'd ID, or the model isn't available where you're calling |
| `...on-demand throughput isn't supported` | This model needs an **inference profile ID** (a cross-region ID prefixed like `us.`), not the bare model ID |
| `ThrottlingException` | You hit a quota. Back off and retry — and check Service Quotas |
| `ResourceNotFoundException` | Usually the region again |

**Nine out of ten first-week errors are one of two things: region, or model access.**
Check both before debugging anything else.

---

## Cost control (set this up now)

Bedrock is pay-per-token with no free tier for most models. A loop with a bug will happily
spend real money.

1. **Billing → Budgets → create a budget** with an alert. Even $5. Do it today.
2. Log `usage` on every call — you can't manage what you don't measure.
3. Use a small model while learning.
4. Keep `maxTokens` sane — not to save money directly, but to bound a runaway.

---

## 🔨 This week's build

**A CLI that talks to a model.** Small, but it's the foundation of the spine project.

**Must have:**
- [ ] `node ask.js "your question"` → prints the answer
- [ ] Streams the output token by token
- [ ] Prints token counts and latency after the answer
- [ ] Checks `stopReason` and warns loudly on `max_tokens`
- [ ] A system prompt you wrote yourself

**Then push further:**
- [ ] `--temp 0.9` flag, so you can feel the dial from the command line
- [ ] `--model <id>` — then run the *same* prompt against two different vendors' models.
      This is Converse's whole value proposition, felt in ten seconds.
- [ ] Interactive chat mode holding the `messages` array — and print the running input
      token count each turn so you watch the cost climb
- [ ] A running total of tokens for the session

---

## ✍️ Write your note

Create `01-bedrock-basics-notes.md` — your own version, not a copy of this. Include:

- The exact model IDs available in your region
- The gotcha that cost you the most time (there will be one)
- Your input-token counts across a 5-turn conversation
- The difference between Converse and InvokeModel, in your own words

---

## ✅ Checkpoint

Move on when you can:

1. Write a `ConverseCommand` from memory — `modelId`, `messages`, `system`, `inferenceConfig`
2. Explain why `Converse` is preferred over `InvokeModel`
3. Say what `stopReason: "max_tokens"` means and why you must never parse that response
4. Explain why turn 5 of a chat costs more than turn 1
5. Name the two things that cause nine out of ten first-week errors

---

*Next: `02-model-selection.md` — Domain 1 begins. Choosing the right model, and the
prompting that makes it behave.*
