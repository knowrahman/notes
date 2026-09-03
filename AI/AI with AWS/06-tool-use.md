# 06 — Tool Use (Week 6)

**Domain 2 — Implementation and Integration, 26% of the exam.** Second-biggest block, and
this is where the model stops being a text generator and starts participating in your system.

> 📌 **Before you start:** reread `AI/agentic AI/agentic AI.md` in this repo. You wrote ~2,000
> lines on exactly this mental model — the LLM proposes, your system decides. You know this
> better than you think; this week is that idea in Bedrock's API shape.

---

## The one thing to never forget

> **The model proposes. Your code disposes.**

The model **cannot execute anything.** It has no network access, no filesystem, no database.
When it "uses a tool", all it does is emit a structured message that says *"I would like you
to call `get_order_status` with `orderId: 4471`."*

Then **your code** decides whether to allow it, runs it, and hands back the result.

That's the whole security model, and it's why tool use is safe enough to build on.

---

## The loop

```
1. You send: messages + tool definitions
                    ↓
2. Model replies with stopReason: "tool_use"
   and a toolUse block: { name, input, toolUseId }
                    ↓
3. YOUR CODE decides: is this allowed? are the arguments valid?
                    ↓
4. You run the real function
                    ↓
5. You send back: the whole conversation + a toolResult block
                    ↓
6. Model either answers, or asks for another tool → back to 2
```

Two things fall out of this that people miss:

- **The loop is yours.** Bedrock doesn't run it. You write the `while`, you set the iteration
  cap, you decide when to stop.
- **Statelessness still applies.** Every trip resends the entire conversation *including* the
  tool calls and results. A chatty agent burns input tokens fast — exactly the climb you
  watched in Week 2.

---

## Defining a tool

Bedrock Converse takes a `toolConfig`:

```js
const toolConfig = {
  tools: [{
    toolSpec: {
      name: "search_notes",
      // ⚠️ This description is the most important string in the file. See below.
      description:
        "Search the engineering notes repository for content matching a query. " +
        "Use this whenever the user asks about a technical topic that might be " +
        "documented in the notes. Returns matching excerpts with their filenames.",
      inputSchema: {
        json: {
          type: "object",
          properties: {
            query: {
              type: "string",
              description: "The search phrase, in natural language.",
            },
            maxResults: {
              type: "integer",
              description: "How many excerpts to return. Default 5.",
            },
          },
          required: ["query"],
        },
      },
    },
  }],
  // auto = model decides (default) | any = must use some tool | tool = must use this one
  toolChoice: { auto: {} },
};
```

### The description IS the prompt

**This is the single most practical lesson of the week.**

The model chooses which tool to call — and what to pass it — by reading the `name`,
`description`, and the per-property descriptions. Nothing else. There is no type checking,
no documentation site, no intuition.

```
❌ description: "Searches notes"
      → model calls it for everything, or never
      → passes whatever it feels like as the query

✅ description: "Search the engineering notes repository... Use this whenever the
   user asks about a technical topic that might be documented in the notes.
   Returns matching excerpts with their filenames."
      → model knows WHAT it does, WHEN to reach for it, and WHAT comes back
```

> **Remember:** a tool description is a docstring the model actually reads. Say **what it
> does, when to use it, and what it returns.** Most "the agent picked the wrong tool" bugs
> are description bugs, not model bugs.

**Corollary:** if two tools have overlapping descriptions, the model will confuse them.
Make the boundaries explicit — *"Use `search_notes` for topics; use `read_file` only when
you already know the exact filename."*

---

## Running the loop

```js
const messages = [{ role: "user", content: [{ text: userQuestion }] }];

for (let turn = 0; turn < MAX_TURNS; turn++) {         // ← always cap it
  const res = await client.send(new ConverseCommand({
    modelId: MODEL_ID,
    messages,
    toolConfig,
    inferenceConfig: { maxTokens: 2000, temperature: 0 },
  }));

  messages.push(res.output.message);                    // keep the model's turn

  if (res.stopReason !== "tool_use") {
    return res.output.message.content[0].text;          // done — it answered
  }

  // Gather every tool the model asked for (it can ask for several at once)
  const toolResults = [];
  for (const block of res.output.message.content) {
    if (!block.toolUse) continue;
    const { toolUseId, name, input } = block.toolUse;

    try {
      // ⚠️ VALIDATE FIRST. These arguments came from a language model.
      const args = schemas[name].parse(input);
      const result = await handlers[name](args);

      toolResults.push({
        toolResult: {
          toolUseId,
          content: [{ json: result }],
          status: "success",
        },
      });
    } catch (err) {
      // Hand the failure BACK to the model — don't throw out of the loop
      toolResults.push({
        toolResult: {
          toolUseId,
          content: [{ text: `Error: ${err.message}` }],
          status: "error",
        },
      });
    }
  }

  messages.push({ role: "user", content: toolResults });
}

throw new Error("Tool loop hit its turn limit");
```

Four things in there are load-bearing:

**1. Cap the turns.** Without a limit, a confused model can loop until your Lambda times out
or your bill hurts. Ten is generous for most tasks.

**2. Validate every argument.** The model generates `input` as text. It can hallucinate an
order ID, invent a field, or pass a string where you wanted a number. **Treat tool arguments
exactly like an untrusted request body** — because that is precisely what they are.

**3. Return errors as results, not exceptions.** `status: "error"` with a readable message
lets the model recover — retry with a corrected argument, or tell the user honestly. Throwing
out of the loop turns a recoverable hiccup into a 500.

**4. Return all results in one user message.** The model may request several tools in one
turn; batch every `toolResult` into a single reply.

---

## Parallel tool calls

One assistant turn can contain multiple `toolUse` blocks — *"check the weather in Chennai
**and** in Bangalore"*. Run them concurrently and return both results together.

```js
const results = await Promise.all(toolUseBlocks.map(runTool));
```

Splitting them across separate messages works, but it's slower and teaches the model to stop
batching. Keep them together.

---

## Security — the part the exam cares about

The model is an **untrusted planner** with a trusted executor sitting in front of it.

| Do | Don't |
|---|---|
| Validate every argument against a schema | Pass model output straight into a query or shell |
| Scope tool handlers with **their own IAM role**, least-privilege | Give the tool layer your app's full permissions |
| Require confirmation for destructive or costly actions | Let the model delete, refund or email unsupervised |
| Log every tool call: name, arguments, caller, result | Only log the final answer |
| Enforce **the user's** permissions inside the handler | Assume the model respects who's asking |

That last row is the one that bites in real systems: **the model has no concept of the
current user's authorisation.** If it asks for `get_salary(employeeId: 12)`, your handler
must check whether *this caller* may see employee 12. The model won't — it can't.

> **Remember:** authorisation lives in your handler, never in the prompt. A prompt saying
> "only access the current user's data" is a *suggestion*. Your IAM policy and your handler
> are the *enforcement*.

And connect it to Week 4: if a tool returns retrieved documents, that text enters the prompt
— so **tool results are another injection surface.** Delimit them.

---

## When NOT to use tools

Tool use costs you a round trip, latency, and tokens on every call. Skip it when:

- **The workflow is fixed.** If you always call A then B then C, write the code. A model
  deciding a sequence you already know is expensive theatre
- **One deterministic lookup** would do — just do the lookup and put the result in the prompt
- **The task is pure text** — summarising, classifying, rewriting

> **Remember:** tools are for when **the model must decide *which* action to take**. If you
> already know the sequence, you don't need a decision-maker — you need a function.

This is a genuine exam pattern: the over-engineered agentic answer is often wrong when the
scenario describes a fixed pipeline.

---

## How this becomes an agent

Tool use is **one exchange**. An agent is **the loop with autonomy** — the model keeps
choosing tools until the goal is met.

```
Tool use  →  "Call get_weather."  →  answer                        (one decision)

Agent     →  check CloudWatch → sees errors → query logs
             → finds a throttle → report                            (it chose the sequence)
```

Same mechanism, same `while` loop. The difference is **how much you let it decide.**
That's Week 7.

---

## 🔨 Build

Give the notes assistant hands.

- [ ] `search_notes(query, maxResults)` — reuse your Week 4 retrieval
- [ ] `read_file(path)` — return one note's contents
- [ ] `list_topics()` — the repo's directory structure
- [ ] Wire the loop above, with a turn cap and schema validation on every argument
- [ ] **Log every tool call and its arguments.** You cannot debug an agent you can't see

Then break it on purpose — this is where the learning is:

- [ ] Make one description deliberately vague. Watch it pick the wrong tool
- [ ] Ask something needing two tools in sequence. Watch it chain them
- [ ] Return `status: "error"` from a handler. **Watch it recover** — this is the moment tool
      use clicks
- [ ] Ask about something not in the notes. Does it invent a filename, or say it doesn't know?
      (If it invents — your escape hatch from note 03 is missing)

---

## ✍️ Write your note

`06-tool-use-notes.md`:
- Your tool definitions, and how you rewrote a description after it misfired
- The tool-call log from a two-tool question
- Input tokens on turn 1 vs. the final turn of an agentic exchange

---

## ✅ Checkpoint

1. What does the model actually do when it "uses a tool"?
2. What is `stopReason: "tool_use"` telling you?
3. Why is the tool description the most important string in your tool definition?
4. Why validate tool arguments when you defined a schema?
5. Why return errors as tool results rather than throwing?
6. Where does the current user's authorisation get enforced, and why not in the prompt?
7. Give a scenario where tool use is over-engineering.
8. In one sentence: tool use vs. agent.

---

*Next: `07-bedrock-agents.md` — Bedrock Agents, AgentCore, and multi-agent patterns. The
loop, managed for you.*
