# Ep 05 | Subagents & Multi-Agent Orchestration

## Overview

This episode introduces the next major architectural leap: moving beyond a single agent and building a team of specialized subagents coordinated by a main agent.

The speaker explains that the PR review agent we have been building so far is still a single-agent system. It works, but it has real limitations:

- one agent has to do too much
- tool context gets crowded
- system prompt complexity grows
- failures become single points of failure
- one model run can become overloaded with many responsibilities

The solution is to split the work into specialist subagents and give them focused tasks.

This episode is about that design pattern and why it matters in both real-world engineering and the certification exam.

---

## Why move from one agent to many?

The speaker frames this directly:

- a single agent can be powerful, but it becomes brittle when it is responsible for every task
- multi-agent systems allow better separation of concerns
- specialization reduces confusion and context overload
- subagents can work in parallel on independent tasks
- the coordinator can aggregate results into a final answer

This is a major improvement over the earlier single-agent review approach.

At the same time, the speaker is clear that there are tradeoffs. The coordinator becomes a central bottleneck if too many subagents are involved.

So the pattern is powerful, but not automatically “better” for every situation.

---

## What subagents buy you

The episode lists four core benefits of using subagents.

### 1. Context isolation

Subagents do not inherit the full context of the coordinator automatically.

This is crucial: they get their own focused prompt and their own reasoning context.

The coordinator only receives the subagent’s final summary, not all the raw intermediate work.

This preserves clarity and keeps the main agent from being overloaded with unnecessary details.

### 2. Parallelism

Independent tasks can run at the same time.

Instead of one agent doing security review, docstring review, and comment generation sequentially, multiple specialists can work in parallel on independent checks.

This reduces total turnaround time and makes the system more scalable.

### 3. Specialization

Each subagent can have a tailored prompt and a restricted tool set.

For example:

- one subagent checks docstrings
- another checks security risk in PR content
- another may prepare the final comment

This keeps each agent focused and avoids a giant, noisy system prompt.

### 4. Tool restriction

A subagent can be given only the tools it actually needs.

That means a docstring reviewer does not need comment-writing tools, and a security reviewer does not need every tool in the system.

This is a strong design principle: give the agent the narrowest privileges needed for the task.

---

## A real-world analogy

The lecturer describes a real-world review team analogy:

- one human checks documentation quality
- another checks security risks
- another checks product behavior or QA concerns

This is precisely the same idea as subagents in software systems.

A single human reviewer might be able to do all of that, but it is much slower, much noisier, and more likely to miss something.

So multi-agent orchestration is an analogy for team-based work rather than monolithic single-worker processing.

---

## The Agent tool

The key infrastructure concept in this episode is the built-in agent tool used for delegation.

The speaker notes that the tool used to be called `task`, but in the current SDK it is now called `agent`.

This matters because older tutorials or stale documentation may still mention the old name.

The official pattern is:

- the coordinator calls an agent tool
- that tool spawns a subagent
- the subagent runs with its own instructions and tools
- it returns a final summary to the coordinator

This is the mechanism through which orchestration happens.

---

## Important fact: subagent context is not inherited

The speaker repeatedly emphasizes one of the biggest mental model shifts:

- subagents do not automatically inherit the coordinator’s full conversation history
- they start from fresh context
- the coordinator must explicitly provide the task in the agent call

This is a key distinction between shared workspace and isolated context.

Two concepts are being contrasted:

- shared workspace: the same files, repo, or data source are available to all agents
- isolated context: each subagent’s conversation is separate and fresh

This is a very important architectural principle.

---

## Shared workspace, isolated context

This is one of the core ideas of the episode.

Even though multiple subagents may all read the same PR or same repo files, they do not share the same internal memory.

The coordinator’s instruction has to be explicit enough to tell the subagent exactly what to do.

This means the coordinator must pass the task, relevant repo information, and boundaries clearly.

Without that, a subagent has no context and will not know what to work on.

---

## Agent definition

To spawn a subagent, the SDK uses an agent definition object.

The speaker lists the most important agent definition fields:

- description
- prompt
- tools
- disallowed_tools
- model

The key idea is that the agent definition is the configuration for a specialized subagent.

### Description

This tells the coordinator when to use that specific subagent.

### Prompt

This becomes the subagent’s own system prompt or operating instruction.

### Tools

This restricts the tools the subagent can access.

### Disallowed tools

This prevents usage even if a broader tool set is present elsewhere.

### Model

This lets you choose a different model for different subagents if needed.

This is useful when some tasks are simple and some are complex.

---

## The coordinator pattern

The episode then moves into the concrete setup for a coordinator plus two specialist subagents.

The example is a PR review pipeline with:

- a docstring reviewer
- a security reviewer

The coordinator receives a single prompt like:

- review this PR
- delegate both checks in parallel
- combine the results into one final review

The point is that the coordinator does not do the actual review work itself. It delegates to specialists.

This is exactly the kind of orchestration pattern that is valuable in agentic systems.

---

## Parallelism is not a config flag

This is a tricky point in the lecture.

There is no explicit switch that says “run exactly in parallel.”

Parallel behavior emerges from the way the model chooses to call multiple subagents in the same decision cycle.

The coordinator prompt must make the parallel work obvious and independent.

For example, if the reviewer says:

- check docstrings
- check security in parallel
- do not wait for each other

the model is more likely to fan out into multiple subagent tool calls in the same turn.

This is more a prompt and task design issue than a simple configuration option.

---

## The architecture tradeoff

The speaker is honest that this approach has a downside: the coordinator becomes the bottleneck.

Why?

- it has to gather results from all subagents
- it has to reconcile conflicting findings
- it has to maintain overall context and final quality

This means more agents is not “free scaling.”

Instead, it gives better specialization, but can increase coordination load.

This is the tradeoff to be aware of in real world multi-agent design.

---

## The hub-and-spoke model

The episode frames subagents as a hub-and-spoke architecture:

- the coordinator is the hub
- subagents are the spokes
- communication flows through the coordinator

Subagents do not talk to each other directly.

This is useful for control and auditability, but it means the central coordinator has to reconcile everything.

That is a major architectural design decision.

---

## Tool restriction and least privilege

The speaker reminds viewers that subagents should receive exactly the tools they need.

This is not just a cleanliness issue; it is a security and reliability matter.

For example:

- the docstring agent only needs the docstring checker tool
- the security agent only needs the PR metadata tool
- the comment poster should not be available to the docstring checker

This is closer to the principle of least privilege and keeps the system safer and more controlled.

---

## The output of the demo

The speaker runs the coordinator and the subagents in the demo.

The final result combines findings from both specialized reviewers:

- docstring review result
- security review result
- combined recommendation

This demonstrates the value of multi-agent orchestration.

The final review is not just a single model answer; it is the product of multiple specialized checks.

---

## Certification relevance

The episode explicitly associates this topic with the certification syllabus.

The speaker says this topic is a major part of the developer certification and may cover a substantial percentage of the exam.

The key things to remember are:

- subagent architecture
- coordinator pattern
- agent tool naming
- isolated context vs shared workspace
- parallelism
- tool restriction
- hub-and-spoke tradeoffs

These become core exam concepts.

---

## Important naming update

The lecture also calls out the naming change in the SDK:

- older docs may say `task`
- newer SDKs use `agent`

This is a naming update, not a different concept.

The speaker warns viewers not to get confused by stale material.

This is especially important because older tutorials or custom notes may mislead you during exam prep.

---

## Final takeaways

### 1. Subagents are specialized workers

They are not just “another conversation.” They are distinct agent instances with focused roles.

### 2. The coordinator is the orchestrator

It delegates, aggregates, and resolves results.

### 3. Context is isolated, workspace is shared

This distinction is critical.

### 4. Tool restriction matters

A narrow tool set makes the subsystem safer and more reliable.

### 5. Parallelism is emergent

It depends on how you structure the delegated task and prompt the model.

### 6. The coordinator can become the bottleneck

More agents are not automatically better. The architecture must be intentionally designed.

---

## Closing summary

This episode shows the shift from single-agent workflows to multi-agent orchestration.

The key message is that a good agent system is often not one huge agent with everything attached. It is a coordinator plus specialists with narrow responsibilities.

That structure makes the system more scalable, easier to reason about, and better aligned with real-world software review and orchestration patterns.

---

## One-sentence takeaway

The most important idea in the episode is:

> A good multi-agent system gives each specialist a narrow job, a fresh context, and the right tools, then lets a coordinator combine the results.
