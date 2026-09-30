# Ep 11 | Prompt Caching, Token Costs & Error Handling

## Overview

This episode is the production-readiness chapter of the course.

The lecturer makes a very important point: once your agent is working locally, the real challenge begins when it is deployed and starts costing money, making decisions at runtime, and failing in unpredictable ways.

The episode is built around three pillars:

1. Cost
2. Observability
3. Debugging and error handling

This is exactly where many teams get stuck. They can build a cool demo, but they cannot run it reliably in production.

The lesson fits right after model selection:

- Episode 10 taught how to choose the right model
- Episode 11 teaches how to run it responsibly
- the next episode moves toward Anthropic-managed runtime and dashboards

So this is the bridge between building an agent and shipping it.

---

## Why production cost matters

The lecture opens with a very realistic scenario:

- your PR review agent is working well
- it catches prompt injections
- it writes structured findings
- it looks polished
- but your billing dashboard tells you spending is much higher than expected

That is a common problem in real AI systems.

The point is simple:

A working agent is not necessarily an efficient agent.

And cost is not an afterthought. It is one of the core design constraints.

In LLM systems, the real cost unit is tokens.

---

## Tokens are the real billing unit

Claude does not process words the same way a normal program does. It processes tokens.

A token is a chunk of text. It can be a piece of a word or multiple words depending on the text.

Roughly, a token is around 3–5 characters on average in English, but it varies by language and content type.

So when you send a prompt, the model is charged by:

- input tokens
- output tokens

Input tokens are everything you send:

- system prompt
- conversation history
- tool definitions
- user message
- context from memory, sessions, or files

Output tokens are everything the model generates:

- final answer
- tool calls
- structured result payloads
- intermediate reasoning (depending on the API and setup)

This distinction matters because output tokens are usually much more expensive than input tokens.

In practice, output generation is the expensive part because it requires the model to produce sequential text. Reading input is cheaper than creating output.

That means:

- verbose agents cost more
- long free-form outputs cost more
- unnecessary steps burn output budget
- structured outputs are often cheaper and cleaner than long prose

This connects directly with earlier lessons on structured outputs and concise tool contracts.

---

## Model pricing and task matching

The lecture reminds the viewer of the earlier model framework:

- Haiku: cheapest and fastest, best for simple classification, routing, basic tasks
- Sonnet: the workhorse for balanced quality and cost
- Opus: strongest reasoning, highest cost, best for the hardest tasks

The practical guidance is:

- do not start with Opus unless you already know the task needs the strongest reasoning
- start with Sonnet if you are unsure
- use Haiku for subagents and cheaper tasks where deep reasoning is not essential

This is one of the most important design habits in production LLM work.

The same task should not always run on the most expensive model.

The right model depends on the job.

---

## The hidden cost multiplier: repeated context

This is where prompt caching becomes critical.

In an agentic loop, the same context repeats across many calls:

- system prompt
- tool schema
- instructions
- routing logic
- review constraints
- previous context fragments

A PR review agent may make many back-to-back model calls in a single run.

If each call resends the same fixed prefix, you are paying for the same input tokens over and over again.

That is exactly what prompt caching solves.

Anthropic can store repeated prompt prefixes server-side and reuse them across calls.

The result is major savings on repeated input tokens, often around 90% for cached input.

This is typically the biggest lever for reducing agent cost.

---

## Prompt caching explained simply

The lecturer shows the mental model very clearly:

Without caching, every API call resends the same prefix again and again.

With cacheing, you pay once for the repeated prefix, then reuse it.

This is especially useful for:

- system prompt blocks
- tool definitions
- large instruction sets
- reusable project rules
- standard schemas

Prompt caching is particularly effective in agentic systems because the same foundational context repeats across turn after turn.

That is why prompt caching is one of the most important cost optimization tools in production.

---

## How it works in the raw Messages API

The lecture walks through the raw API example.

In the raw Messages API, you can mark a block with a cache control flag, such as:

- cache_control: "ephemeral"

This tells the API: cache this content and reuse it on future calls when the prefix matches.

Anthropic does not cache random middle blocks. It works on the prefix pattern.

This means the best caching candidates are usually:

- system prompts
- tool definitions
- large instruction blocks
- schema definitions near the front of the request

The lecture notes a few practical rules:

- caching is best when the repeated content is near the beginning of the request
- there is usually a minimum block size for effective caching
- the cache is usually warm for a short time window during active use
- repeated calls within a few minutes benefit the most

This is exactly why many agent loops benefit so much from prompt caching.

---

## Why it matters for agent architecture

The lesson connects caching to the earlier design decisions.

If your system does things like:

- subagents delegating work
- repeated tool descriptions
- review instructions repeated across loops
- the same coordinator prompt reused for every step

then prompt caching becomes a huge cost reduction tool.

The speaker specifically notes that the coordinator and tool schema are ideal candidates for caching.

This is one of those concepts that is easy to ignore in a demo but extremely important in production.

---

## Observability: opening the black box

The second pillar is observability.

This is one of the biggest differences between traditional software and agentic software.

In regular software, you can usually trace a function call directly.

In agentic systems, the model decides what to do at runtime.

So you need to be able to answer questions like:

- what tool was called?
- with what arguments?
- what result came back?
- what did the model decide next?
- where did the workflow break?
- did the system silently degrade?

This is observability.

It means making the black box inspectable.

---

## Session transcripts are the flight recorder

The lecture returns to the project’s earlier session architecture.

The agent SDK writes session transcripts to disk, and these are extremely valuable.

You can inspect:

- each turn
- tool usage
- model responses
- arguments passed
- timing
- errors
- context transitions

This is the closest thing to a flight recorder for an LLM agent.

Without this, debugging becomes guesswork.

That is why earlier episodes on memory, sessions, and resumable state matter more than they seem to at first glance.

The transcript is not just a log. It is the ground truth of the runtime.

---

## What to log in an agent system

The lecturer gives the effective production checklist.

You want to capture:

- tool name
- tool arguments
- execution result
- timing
- step number
- token usage per request
- errors
- session identifier
- compaction events
- fallback behavior

In short: if an agent can do something surprising, you want a record of it.

This matters because agent failures are often not obvious in the final output.

A bad run may appear complete while still being wrong in a subtle way.

---

## Why debugging agents is harder than debugging normal code

The lecture explains the core difficulty:

### 1. Nondeterminism

The same prompt can produce different tool sequences on different runs.

This is not a bug in the system; it is a property of probabilistic agents.

### 2. Compound failures

The root cause may appear three or four steps earlier than the visible symptom.

A bad tool result may lead to a wrong decision later, and the data chain becomes hard to reconstruct.

### 3. Context degradation

The model may lose important information because memory is compressed or summarized.

This is especially relevant in long-running agents with context windows and session compaction.

### 4. External dependencies

MCP servers, APIs, and tools can fail transiently or return incomplete data.

A system is only as reliable as its weakest dependency.

This is why debugging agents often feels like forensic work.

---

## The practical debugging playbook

The lecturer gives a good production mindset for troubleshooting.

### Strategy 1: read the transcript

Do not guess.

Read the actual session log and reconstruct the chain of actions.

This often reveals where the model began to drift or where tool output was wrong.

### Strategy 2: binary search the tool chain

If there are many steps, check the middle of the flow first.

Once you locate the first wrong output, you narrow the problem quickly.

This is similar to debugging a distributed system by isolating the first bad value.

### Strategy 3: check compaction history

If a context loss or summary happened before the failure, this is a strong clue.

Compaction is not just a memory optimization; it can be a correctness risk if important information was dropped.

### Strategy 4: treat the run as evidence

The logs are not optional. They are the evidence.

---

## Error handling patterns that actually work

Now the third pillar: debugging and error handling.

The lecturer stresses that this is a different category from ordinary software error handling.

There are two dangerous anti-patterns:

### Anti-pattern 1: silently swallowing errors

A tool fails, but the agent continues as if nothing happened.

The result looks valid, but a critical check was skipped.

This is incredibly dangerous in security, quality, and review workflows.

### Anti-pattern 2: crashing on the first failure

You treat one tool failure as a fatal system failure and stop the whole run.

That can discard valid results from other subagents or partial work that could still be useful.

The lesson’s answer is not “be lazy” or “fail fast.”

The correct pattern is:

- return structured errors
- include a clear reason
- provide a retry signal when appropriate
- allow graceful degradation
- keep partial progress visible

---

## The structured error pattern

The course has already introduced structured outputs and error result shapes.

This episode reinforces them by making the point that tools should not fail in vague ways.

A tool should return something like:

- success
- error
- retryable flag
- message
- partial payload if available

This gives the agent enough information to decide what to do.

For example:

- retry transient timeout
- skip an optional tool
- fall back to a simpler path
- notify a coordinator that a section is missing

This makes the system robust instead of brittle.

---

## Graceful degradation

This is one of the most important principles in real production systems.

If one subagent fails, the system should still give useful output rather than giving nothing.

A coordinator can do something like:

- keep results from successful subagents
- mark which part failed
- present a partial review
- flag the missing coverage explicitly

That is much better than silent failure or total collapse.

The lecturer’s example is a review system where one subagent times out.

The correct behavior is not to pretend the review is complete.

It is to keep the valid findings, flag the gap, and let the human or the system decide next steps.

---

## Retrying wisely

The lesson also emphasizes retry patterns.

Not every error is retryable.

A good retry strategy should distinguish between:

- transient network failures
- timeout issues
- temporary external dependency issues
- permanent validation failures
- user-input problems

Retry with backoff when the error is likely temporary.

Do not endlessly retry forever.

A retry loop with cap, delay, and logging is much healthier than a blind infinite loop.

---

## Hooks as deterministic safety nets

The course keeps reinforcing that deterministic logic is valuable in a probabilistic system.

Hooks are one of the best examples of this.

A hook can:

- validate policy conditions
- block unsafe actions
- log events
- capture context drift
- enforce invariants before or after a turn

This is important because the model is not a perfect decision engine. It is a probabilistic system.

Deterministic guardrails are a needed complement.

---

## The exam-style mental model

The speaker uses a simple test case to reinforce the principle.

A coordinator fans out to several subagents.

One subagent returns a timeout or structured failure.

The right behavior is to keep the valid results, mark the failed portion, and continue gracefully.

The wrong behavior is to:

- silently ignore the failed subagent
- pretend the full analysis was successful
- crash the whole run
- retry forever without bounds

This is exactly the kind of judgment the course is preparing students to make.

---

## The big takeaway

This episode is a reminder that building agents is not only about model quality or tool design.

The production story includes:

- cost management
- observability
- error handling
- graceful recovery
- prompt efficiency
- structured failures

The lesson closes with a few clear rules:

### 1. Output tokens are expensive

Keep outputs concise and intentional.

### 2. Prompt caching is one of the biggest cost wins

Repeated system prompts and tool blocks should be cached whenever possible.

### 3. Choose the model by the task

Do not default to the most expensive model.

### 4. Read the transcript, not your assumptions

Logs are your evidence.

### 5. Never silently swallow failures

If something fails, the system should know and respond appropriately.

### 6. Use partial results when full success is impossible

Graceful degradation is better than total collapse.

---

## Why this matters for the series

This episode is the turning point where the course stops focusing mainly on “can the agent do the thing?” and starts focusing on “can it be run responsibly and reliably?”

That is what makes the system production-grade.

It is the move from demo to real agent operations.

The next episode then moves into managed Anthropic runtime and deployment patterns, where these concepts become even more important.

---

## Summary

Episode 11 connects all the earlier lessons into one operational reality.

It reminds the student that:

- tokens are the real cost unit
- prompt caching can drastically reduce repeated spend
- observability is mandatory for agent systems
- debugging requires transcripts and structured evidence
- error handling must be graceful, explicit, and recoverable

This is the part of the course where the student learns that an agent is not just a chatbot with tools. It is a running system with economics, operational complexity, and failure modes.
