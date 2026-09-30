# Ep 06 | Agent Memory, Sessions, Resume & Forking

## Overview

This episode addresses one of the biggest problems in agentic systems: amnesia.

The speaker frames the issue very clearly:

- an agent appears smart and capable in a single run
- it reads the repo, reasons about code, and returns a useful review
- but when you run it again, it behaves as if it has never seen the repo before
- the agent starts from a blank slate every time

This is the core truth of most agent systems:

- no model run is automatically persistent
- a single process ends, and the context it built disappears
- subagents also start fresh unless the coordinator explicitly passes context

So the real question is not “can the agent reason?” but “how do we make it remember the right things without overloading the prompt?”

This episode introduces the four key memory layers that matter most in practice.

---

## Why memory is confusing

The speaker warns that the word “memory” is overloaded in AI work.

People often mix up different concepts and treat them as one thing, but they are not the same.

The episode introduces four distinct memory mechanisms:

1. project rules
2. tool-based memory or learned working memory
3. current session conversation history
4. compaction / summary memory

These are related, but they are not interchangeable.

This distinction matters a lot in real systems and also in the certification exam.

---

## The real-world analogy

The lecturer explains this with a construction analogy.

Imagine a contractor working on your house.

### 1. House rules

These are the instructions that are always true:

- take your shoes off
- do not touch the fuse box
- follow the safety rules

This is analogous to a project-level CLAUDE.md file.

These are rules that should be loaded every run.

### 2. Notebook / working memory

The contractor also keeps notes while working:

- upstairs sink is broken
- kitchen cabinet must stay open for the plumber
- oak was selected for the kitchen

This is like explicit memory captured in a file or tool system.

This is more durable than a single conversation, but it is not a permanent database by default.

### 3. Ongoing conversation

The contractor and the homeowner are in a live conversation about the project.

That is the current session: the active message history for the current run.

This is useful, but it is ephemeral.

### 4. Compacted summary

As the discussion becomes long, the system compresses it into a brief summary:

- kitchen agreed on oak
- quote pending
- hardware and finish decisions recorded

This is compaction.

It is useful for preserving intent, but it is a lossy summary, not a full record.

---

## Key principle: different memory types solve different problems

The speaker emphasizes a very important rule:

- put durable rules in CLAUDE.md
- keep structured memory in a file or database if it must survive runs
- use session history only for the current conversation
- use compaction only as a compression mechanism, not as a replacement for durable memory

A common mistake is to put everything in one place and assume the agent “remembers it.”

That creates two failure modes:

- a session-only rule disappears when the conversation ends
- a compacted summary loses important details that were never stored elsewhere

This is why context engineering is about choosing the right storage mechanism for the right purpose.

---

## CLAUDE.md: the project rules layer

The most underrated tool in this whole story is a plain markdown file named CLAUDE.md.

The speaker explains that it is not a README and not general documentation.

It should contain the facts that are always true and that Claude would otherwise get wrong without being told.

Typical examples:

- repo conventions
- review contract rules
- must-check file paths or severities
- important instructions for PR review
- compacting rules for long sessions

The agent SDK loads this file automatically when configured properly.

### Project-level vs global-level

The file can live at the project level or in the user-level home configuration.

The recommended default is the project-level file because it is scoped to the repo.

This keeps the instructions specific to a project instead of applying globally across all work.

### What belongs in CLAUDE.md

The plan is simple:

- put high-value rules here
- keep them short and specific
- avoid bloating it with raw history or everything the agent has ever done
- make it a compact source of truth for behavior

This is also the place where the curator can define rules that must survive across runs.

---

## The settings sources trap

This is one of the most important practical gotchas in the episode.

The speaker calls it the “settings sources trap.”

The idea is simple:

- the Agent SDK only reads CLAUDE.md if you let it
- there is a setting called `settingsSources`
- if you specify project scope, the SDK reads the project CLAUDE.md
- if you do not specify it, it may use a global user-level CLAUDE.md instead
- if you explicitly pass an empty array, it loads nothing at all

This means the behavior can silently change based on configuration.

That is dangerous because it looks like the agent is using your repo rules, when it is actually using a different source or nothing at all.

The speaker strongly warns developers to understand this because the failure mode is silent.

### Example mental model

- no setting sources configured: may load global defaults
- explicit project setting: loads repository rules
- empty array: load nothing

This is why engineers must be explicit about what memory sources are enabled.

---

## Auto-memory and its reality check

The episode also addresses a common confusion around “automatic memory”.

The speaker says they tested the behavior and found that the SDK does not reliably behave the way some documentation suggests.

In practice:

- the SDK stores session transcripts automatically
- those transcripts are saved to disk as JSONL session logs
- but they are not necessarily auto-loaded like a memory system in all cases

This is important because people assume the system will remember prior sessions automatically.

The honest answer is:

- sessions are recorded
- but you still need a deliberate pattern to resume, continue, or load the right state

This is one of the main reasons the episode focuses on session-based resumption rather than magic auto-memory.

---

## Sessions: the conversation layer

The session is the active transcript of the conversation.

In the Claude Agent SDK, every run records the full history:

- user messages
- assistant messages
- tool calls
- tool results
- intermediate reasoning traces
- final result content

This gets written to disk automatically as a session transcript.

That means you can inspect it later and resume from it.

The speaker shows that the SDK creates session files in a local project directory, and each run gets a unique session ID.

This is the key concept behind session memory.

### Why this matters

Without a session, each agent startup is effectively a fresh run.

With a session, you can continue the same discussion, re-run the same review, or branch a new conversation without losing the old one.

This is how an agent can avoid re-reading the entire repo every time if the human or system chooses to continue a prior session.

---

## Resume, continue, and fork

The episode introduces three operational concepts.

### 1. Continue conversation

This means continue the most recent active session.

The agent loads the most recent session ID and continues from there.

This is useful when you want to pick up where you left off.

### 2. Resume by ID

This explicitly resumes a known session ID.

That is the more precise version.

You can store the session ID and later say:

- resume this exact review thread
- resume this PR review
- resume this debugging session

This is what makes session memory actually usable in production workflows.

### 3. Fork session

Forking creates a new branch-like copy of the session.

This is extremely useful when you want to try a different direction without destroying the original conversation.

Examples:

- same PR, but try a stricter security review
- same debugging thread, but explore a different patch strategy
- same task, but test a new approach without losing the original chain of reasoning

The difference is that the original session remains intact while the fork explores a new path.

---

## Important detail: sessions persist conversation, not the repo state

The speaker makes one important distinction:

- a session is just the conversation history
- it does not automatically branch the filesystem state or repo state

So if you fork a session, the conversation copies, but the working tree is not magically isolated.

The real files in the repo are still the ones the tool is interacting with.

This is a crucial mental model:

- fork = new conversational branch
- repository edits are still real and shared unless the system is deliberately designed otherwise

This is why session history and file state should be treated as separate layers.

---

## Capturing the session ID

The speaker emphasizes that every run returns a session ID.

You should capture it even when the task fails.

Why?

- you may wish to resume and continue from the failure state
- some tasks fail before completion but still contain useful progress
- if you do not save the ID, you may lose the best continuation point

The model may return a result with a `session_id` or equivalent metadata.

That value is your anchor for future resumption.

This is a practical workflow for long-running agent tasks.

---

## Why sessions solve the “rerun everything” problem

This is the immediate practical benefit.

A fresh agent run will often reread the repo from scratch and ask the same questions again.

That is expensive and can be redundant.

With a session:

- the system can resume the exact prior thread
- the conversation does not start fresh
- the agent can continue from the last decisions instead of pretending they never happened

This is crucial when the agent is doing code review, debugging, or long-running technical analysis.

---

## Compaction: the memory pressure valve

The next major concept is compaction.

The speaker explains that the conversation window is finite.

Even if the model context is large, it is not infinite.

As the conversation grows, the quality and clarity degrade. The model has more context to reason over, and the relevant details become harder to preserve.

So the SDK compacts older conversation sections into a shorter summary.

That summary is designed to preserve what matters for the next step.

### Why compaction is not durable memory

This is a crucial distinction.

Compaction is a compression strategy.

It is not a durable memory store.

It optimizes for “what do I need for the next action?” not for “what exact facts must never be lost?”

This means compaction may remove the raw conversation details that were important earlier.

That is why the lecture strongly pushes the idea that if an instruction or fact is truly important, it should live in:

- CLAUDE.md
- an explicit memory file
- a database
- a structured notes store

Otherwise it may vanish during compaction.

---

## Compaction and durable state

The lectures also explain that compaction and memory should be designed together.

The summary should preserve only the parts that matter for future steps.

For example, in a PR-review workflow, the important durable facts may include:

- PR number
- confirmed findings
- file paths and line numbers
- whether a comment has already been posted
- severity classification
- whether a review is blocked

These facts are not optional noise. They are critical state.

If they are not stored as durable state, they can disappear into compaction.

That is why the recommended design is to keep important operational facts in a structured memory layer instead of relying on summary alone.

---

## Pre-compact hooks and observability

The session also introduces a pre-compaction hook.

This is not the same as a pre-tool-use security hook.

The speaker explains that a pre-compact hook is mainly for observability.

It fires before the SDK compacts the conversation and lets you log what is about to be summarized.

This is useful for debugging context health, but it is not a substitute for proper memory design.

The speaker warns that it is too late to save meaningful state if the compaction hook fires after the important facts have already been lost.

This is a very important design principle:

- enforce protection early
- store durable facts before the summary stage
- use compaction as a safety valve, not as the primary memory system

---

## The memory hierarchy in one sentence

The whole episode can be summarized as a hierarchy:

- CLAUDE.md = always-true rules
- explicit memory files / tools = durable working memory
- sessions = current conversational transcript
- compaction = lossy, short-lived summarization layer

That is the basic architecture behind reliable agent memory.

---

## Why this matters for the exam

The speaker emphasizes that this topic sits under the broader domain of context engineering.

Even if it is not the largest single weight in the exam, it is conceptually central.

The exam often expects you to understand:

- what memory means in an agent
- why sessions are not the same as durable memory
- why CLAUDE.md is important
- why compaction is not a substitute for a data store
- how resume and fork differ
- how working-directory scope affects session reuse

This is exactly the kind of topic that gets tested indirectly in real-world architecture questions.

---

## Practical workflow for PR review agents

The lecture closes by tying it all together in a practical pipeline.

A real review agent can use:

- CLAUDE.md for project review rules
- a session index for PR-to-session mapping
- the session ID to resume or continue previous reviews
- a forked session if investigating a different interpretation
- compaction only to keep the conversation manageable

This means the agent does not have to re-read the whole repo from scratch every time if it can resume the right review state.

The result is a much more efficient and realistic agent workflow.

---

## Final takeaways

The speaker leaves the audience with a few high-value final principles:

- agents do not memorize unless you deliberately design for memory
- sessions are not durable memory
- CLAUDE.md is the rules layer
- compaction is a pressure valve, not a strategy
- resume and fork are how you navigate long-lived agent workflows
- sessions are keyed by working directory, so wrong CWD means a fresh start
- if a fact is important, store it somewhere durable

This episode is about making the agent remember intentionally rather than accidentally.

And that distinction is what separates a toy demo from a real, maintainable agent workflow.

---

## Closing thought

The speaker ends by saying the next episode will cover structured output, which also matters heavily in real agent systems.

But before that, the main lesson is clear:

memory in agents is not one feature. It is a stack of mechanisms, each with different responsibilities.

Understanding that stack is the difference between a fragile agent and a useful one.
