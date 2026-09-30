# Ep 10 | Opus vs Sonnet vs Haiku, Thinking & Effort

## Overview

This episode is about model selection, cost control, and reasoning budget.

The lecturer starts by pointing out something subtle but very common in real agent projects:

- the team builds a serious agent with tools, memory, subagents, hooks, structured output, and MCP integrations
- but for a long time, they never stopped to ask the most basic question: which model should this agent use?

That is a huge oversight because model choice affects:

- output quality
- tool use quality
- cost
- latency
- reasoning depth
- reliability

This episode makes the point that a model is not just a brand name. It is a trade-off system with cost, speed, and capability all interacting.

---

## The hidden cost problem

The speaker is blunt: if you keep using the default model without thinking, you can end up with surprisingly large bills.

This is especially true in agentic systems because:

- context is large
- tool outputs are large
- sessions can accumulate a lot of tokens
- repeated runs and subagents multiply cost

The lesson is that model choice must be deliberate, not accidental.

A model is a cost vector, not just a capability label.

---

## The core mechanical fact: models predict tokens

The lecture explains the internals in a very practical way.

A language model is basically an autocomplete system on steroids.

At a high level, it does this repeatedly:

- read what came before
- predict the next chunk of text
- append it
- repeat

That chunk is a token.

A token is roughly a fraction of a word, often around three-quarters of a word on average.

This matters because cost is measured in tokens, both:

- input tokens
- output tokens

That is the billing model behind these systems.

Once you understand that, the economics of agent systems become much clearer.

---

## Why token counting matters so much

The speaker emphasizes that model cost is not about “number of requests.”

It is about how many tokens the system sends and receives.

That includes:

- system prompts
- the conversation history
- tool output
- file context
- messages from subagents
- completion output

So even a small design change can multiply cost dramatically.

This is the reason the course repeatedly emphasizes:

- keep prompts compact
- keep context focused
- choose the right model
- use the right amount of thinking
- avoid unnecessary tool output

---

## Context window is not memory

This is one of the most important misconceptions the lecture addresses.

The context window is like a desk or a visible working surface.

It is the amount of information the model can see at once.

It is not the same as durable memory.

When the session ends, that visible context effectively resets.

This is why earlier episodes on memory and session resume matter so much.

The model does not inherently remember everything across runs.

It only sees what you include in the current session and context window.

That is why session IDs, resume behavior, and memory files exist.

---

## Why identical prompts can still yield different responses

The speaker explains that models are probabilistic.

Even with the same prompt, they may not return the exact same answer every time because they sample from likely next tokens rather than always choosing the single deterministic output.

This is not a bug. It is part of the model behavior.

The controls that affect this include:

- temperature
- top-p
- top-k
- sampling choices

The lecture notes an important practical update:

- Anthropic's API no longer allows arbitrary temperature settings in the same way older tutorials often show
- the default behavior is usually the safe route
- extreme temperature tuning is not the recommended pattern anymore

The replacement is to steer behavior through prompting and structured output, not by fiddling with randomization knobs.

---

## The 3 tiers: Opus, Sonnet, Haiku

The course then introduces the three main Anthropic model families.

### Haiku

This is the lightweight, fast, low-cost model.

The speaker compares it to a scooter or similar small, efficient vehicle.

It is ideal for:

- well-scoped tasks
- quick classification
- formatting
- simple extraction
- repetitive mechanical jobs
- smaller subagents doing routine work

It is not the best fit for highly ambiguous or high-stakes reasoning.

### Sonnet

This is the middle ground and the default workhorse for many engineering tasks.

The speaker compares it to an everyday car.

It balances:

- capability
- speed
- cost

This is the model most people reach for when they want something useful without paying Opus prices.

It is very good for:

- coding tasks
- PR review
- multi-step tool-using agents
- moderate reasoning and judgment

### Opus

This is the most powerful and most expensive option.

The speaker compares it to a big heavy-duty vehicle or a workhorse used for the hardest jobs.

It is best for:

- hard reasoning
- ambiguous multi-step planning
- high-stakes tasks
- long or difficult debugging and design work

But it is expensive and sometimes slower.

---

## The trade-off triangle: capability, cost, and latency

The speaker explains a simple mental model: every model sits somewhere in a triangle of trade-offs.

The three corners are:

- capability
- cost
- latency

You cannot maximize all three at once.

- Haiku: cheap and fast, lower capability ceiling
- Sonnet: balanced middle ground
- Opus: strongest capability, higher cost and slower speed

This is why the right model depends on the task.

The lecture makes a very important summary statement:

- there is no universally best model
- there is only the best model for the specific task

---

## Matching model choice to job type

The speaker gives simple decision rules.

### Use Haiku when:

- the task is narrow and mechanical
- the requirements are clear
- the work is repetitive and high-volume
- the job is mostly structured extraction or classification
- the model is just doing a routine subagent task

### Use Sonnet when:

- you need balanced reasoning
- you want something reliable for typical coding or review work
- the task is more than trivial but not extremely open-ended
- you want a default production workhorse

### Use Opus when:

- the task is highly ambiguous
- the risk is high if the answer is wrong
- the job needs hard reasoning and deep planning
- you need the strongest model for a difficult architectural or debugging challenge

This is the practical framework the lecturer wants you to keep in your head.

---

## Thinking and effort: the paid scratchpad

The speaker moves from model selection to the second major concept: reasoning effort.

Thinking is the model’s internal scratchpad.

It is not “the answer itself” but the model’s private reasoning process before it writes the final output.

This is important because thinking tokens are still tokens, and they cost money and time.

So thinking is not free.

It is a paid reasoning budget.

This is why the speaker calls it a “paid scratchpad.”

---

## When thinking is worth it

Thinking is worth it when:

- the task is genuinely hard
- there is ambiguity
- the job requires multi-step reasoning
- wrong output is expensive
- there are multiple valid approaches and the system needs judgment

Examples include:

- architecture decisions
- exploring a complex bug
- evaluating subtle trade-offs
- cross-repository reasoning
- planning a fix with uncertain constraints

These are cases where deeper reasoning can materially improve quality.

---

## When thinking is wasted

Thinking is wasted when:

- the task is routine and deterministic
- the output is mechanical and structured
- the task is mostly extraction or formatting
- the job is already tightly specified
- the model does not gain much from extra deliberation

Examples:

- simple classification
- extracting fields from a well-defined schema
- formatting a summary
- repeating a straightforward transformation

In these cases, extra thinking mostly burns budget without helping much.

---

## Effort controls vs reasoning mode

The transcript distinguishes between a few concepts:

- model choice
- thinking budget / effort
- adaptive or explicit reasoning selection

In the raw Messages API, you can often configure the effort level directly.

In the higher-level SDK, the pattern is more like choosing a model and letting the system handle the rest unless you are explicitly controlling lower-level behavior.

The practical message is that thinking is a cost knob and should be tuned to the task, not applied blindly.

---

## Why subagents often use cheaper models

The lecture connects this back to the earlier subagent architecture.

The coordinator may use a more capable model, but subagents doing narrow, repetitive tasks can often use cheaper models like Haiku.

This is a powerful optimization:

- the coordinator handles orchestration and high-level judgment
- specialized subagents do narrow, well-scoped tasks on cheaper models

This keeps overall cost low while preserving model quality where it matters.

This is exactly the kind of architectural optimization the course has been teaching throughout.

---

## Why pinning model names matters

The speaker also emphasizes an operational discipline: pin the exact model string.

Why?

- if you use a vague alias like “Haiku,” the underlying model version can change
- model behavior can shift slightly between releases
- prompts and performance can regress unexpectedly
- you lose reproducibility and compatibility

The better pattern is:

- pin the explicit model identifier
- test it under the intended workload
- upgrade intentionally when you want to change the version

This is especially important in production systems and in certification-level understanding because behavior drift is a real software risk.

---

## The practical quick decision framework

The lecture ends with a simple framework the audience can reuse.

### Use Haiku for:

- narrow, mechanical tasks
- high volume operations
- structured extraction
- repetitive sub-work
- cheap filtering or classification tasks

### Use Sonnet for:

- normal coding and review work
- most everyday agent operations
- tool-using tasks with moderate complexity
- balanced production defaults

### Use Opus for:

- hard reasoning
- high stakes decision making
- ambiguous or open-ended tasks
- situations where you are willing to pay for the best reasoning

And for thinking budget:

- apply more reasoning only when the task is genuinely difficult
- otherwise, keep it lean and cheap

---

## Final takeaways

This lesson is really about making model choice a deliberate engineering decision.

The key ideas are:

- cost is measured in tokens
- context window is not memory
- model choice is a trade-off between capability, cost, and latency
- Haiku is fine for lightweight work, Sonnet for most work, Opus for hard reasoning
- thinking is a paid reasoning budget and should be used selectively
- pin the exact model version if you want stable behavior
- the best model is the one that fits the task, not the most expensive one

That is the heart of practical model design in a Claude-based workflow.
