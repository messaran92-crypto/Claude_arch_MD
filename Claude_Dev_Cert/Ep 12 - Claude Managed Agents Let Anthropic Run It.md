# Ep 12 | Claude Managed Agents: Let Anthropic Run It

## Overview

This episode completes the main Claude Certified Developer syllabus.

Across the previous episodes, the course built a PR review agent from a raw Messages API loop through:

- the Claude Agent SDK
- custom tools
- MCP servers
- hooks and guardrails
- subagents
- memory and sessions
- structured output
- skills
- model selection
- cost management and error handling

The final production question is: who runs the agent?

Until now, the agent has run in a local Python process or in infrastructure managed by the developer. This episode introduces managed agents, where Anthropic manages both the agentic runtime and the sandbox environment.

The key shift is from:

> Build the agent and run the process yourself

To:

> Define the agent and let Anthropic run it in a managed environment

---

## The three runtime layers

The course has used a three-layer model throughout the series.

### Layer 1: Raw Messages API

You manage everything:

- the model calls
- the agent loop
- message history
- tools
- sandbox
- session state
- recovery
- observability

This gives maximum control, but also maximum responsibility.

### Layer 2: Claude Agent SDK

The SDK manages the agentic loop and provides useful abstractions for tools, sessions, model calls, and related agent behavior.

You still manage the runtime where the agent runs.

That runtime might be:

- your laptop
- your own server
- AWS
- your company infrastructure
- another deployment platform

### Layer 3: Managed Agents

Anthropic manages both sides:

- the agentic loop
- the sandbox or runtime environment

Your application defines and communicates with the agent, while Anthropic handles the long-running execution infrastructure.

This provides convenience, durability, recovery, and operational features, but with less infrastructure control and an additional runtime cost.

---

## Managed agents are still in beta

The transcript explains that managed agents are a relatively new platform capability.

The SDK uses a beta header, and the following can still change:

- API shapes
- event types
- pricing
- platform behavior

The practical lesson is to treat beta features carefully in production. They may already be used by real companies, but their contracts are not necessarily final.

---

## The four core primitives

Managed agents are built around four main concepts:

1. Agents
2. Environments
3. Sessions
4. Events

Understanding the difference between these primitives is central to the episode and to the certification exam.

---

## 1. Agents

An agent is a reusable, versioned configuration.

It contains the definition of how the agent should operate, including:

- model
- system prompt
- tools
- MCP servers
- skills
- subagents
- permissions
- other runtime configuration

An agent is similar to a recipe card: it describes what should be prepared, but it is not one specific execution of the task.

You create an agent once and then reuse it across many sessions.

### Agent versioning

Managed agents are versioned.

When you change the configuration, such as:

- modifying the system prompt
- adding a tool
- changing permissions
- updating an MCP configuration

a new immutable version is created.

A session is pinned to the agent version that it started with. This means a configuration update does not unexpectedly change an already-running session.

Versioning enables:

- safe iteration
- side-by-side testing
- A/B testing
- rollback when a new version regresses
- stable long-running sessions

This is an important difference from editing a local Python process while it is running.

---

## 2. Environments

An environment is where a session actually runs.

The environment includes the sandbox state and execution context needed by the agent.

The platform provides two broad options:

- cloud-managed environment
- self-hosted environment

With a cloud-managed environment, Anthropic manages the infrastructure.

A self-hosted environment gives you more control over where the sandbox operates, while the managed agent platform still provides the surrounding agent features.

The environment is like the kitchen in the course’s restaurant analogy. The agent is the recipe, while the environment is the place where the recipe is prepared.

---

## 3. Sessions

A session is a running instance of an agent performing a specific task.

Each session has its own:

- session ID
- transcript
- sandbox state
- outputs
- execution history

One agent definition can be used to start many sessions.

### Durable sessions

In a local SDK process, the session normally depends on your process and the recovery logic you build.

If the process crashes and you have not implemented session persistence or resume behavior, the active execution may be lost.

In managed agents, the session state is stored in Anthropic’s infrastructure.

This means the client application is mainly a communication layer. If the client disconnects or the process crashes, the session can still be recovered through its session ID.

The key mental model is:

- local SDK session: usually tied to your process unless you build persistence
- managed session: durable state maintained by the managed runtime

---

## 4. Events

Events are the communication mechanism between your application and a managed session.

In the Agent SDK, the application commonly calls `query` and receives messages through an asynchronous stream.

Managed agents use an event-based model:

- open a stream on a session
- send events into the session
- receive events from the session

Events can represent:

- user turns
- tool results
- status updates
- agent responses
- execution changes
- communication between agents

The underlying communication uses server-sent event-style streaming, but the main concept to remember is that events are the interface for interacting with a managed session.

---

## Creating agents through the platform

Managed agents can be configured through the Anthropic platform interface or through a configuration format such as YAML.

The interface exposes settings that are conceptually familiar from the Agent SDK:

- model selection
- system prompt
- built-in tools
- custom tools
- MCP servers
- skills
- subagents
- tool permissions

Tools may be configured with permission modes such as:

- always allow
- always ask
- always deny

The platform hides much of the implementation code, but it does not remove the need to understand agent design.

You still need to know:

- what the system prompt should require
- which tools should be available
- how a custom tool schema should be designed
- which tools need approval
- which credentials an MCP server requires
- how to prevent sensitive data from leaking

The SDK was not wasted effort. It taught you how to make the decisions that the managed interface asks you to configure.

---

## Managed agents and the Agent SDK are conceptually similar

The concepts transfer directly between the SDK and managed agents.

| Agent SDK concept | Managed agent equivalent |
|---|---|
| System prompt in agent options | System prompt in the agent configuration |
| Tool decorator or tool definition | Custom tool configuration |
| Allowed tools | Tool permission settings |
| Subagent definition | Linked agent used as a subagent |
| MCP configuration | MCP server attached to the agent |
| Local session files | Managed session transcript and audit trail |
| Custom recovery logic | Durable managed session behavior |
| Local scheduler or cron | Managed deployment schedule |

The abstraction changes, but the architecture does not.

Learning the lower-level SDK remains valuable because it explains what the managed configuration actually means.

---

## Hooks, subagents, MCP, and memory

The core concepts from earlier episodes still apply.

### Hooks

Hooks continue to provide deterministic enforcement and validation.

The main difference is that the enforcement logic may be configured or executed through the managed platform rather than only through local code.

### Subagents

A subagent is also an agent definition.

You create the subagent and then attach it to the coordinator agent.

### MCP servers

MCP integration is configured as part of the agent definition.

You still need to understand transport, authentication, permissions, and data handling.

### Memory

Managed agents can provide persistent memory stores.

A memory store gives the agent cross-session memory without requiring you to build all of the storage and retrieval logic yourself.

The name and description of a memory store can become part of the agent’s available context when the store is attached.

---

## Deployments, credentials, and schedules

A deployment binds an agent to an environment and its operational configuration.

This can include:

- environment selection
- credentials
- schedules
- execution settings

Managed agents provide credential vaults for storing secrets that the agent needs to access external systems.

These vaults must be treated carefully because credentials may be available to anyone with appropriate workspace or API-key access.

The platform can also provide scheduled deployments, reducing the need to build and operate your own cron jobs or task queues.

---

## Managed observability

In the local Agent SDK workflow, you may need to inspect JSONL session files or build your own dashboard.

Managed agents provide a console audit trail out of the box.

You can inspect a session by its ID and review:

- what the agent did
- which tools it called
- how the session progressed
- which subagents were invoked
- what outputs were produced
- where failures occurred

This is the managed equivalent of the session transcript and observability workflow discussed earlier in the course.

The important difference is that the platform stores and presents the audit trail for you.

---

## Cost model

With the Agent SDK, the main cost is API token usage.

Managed agents add a runtime charge on top of token costs.

The transcript describes an active-session charge of approximately $0.08 per hour.

The important detail is that the charge applies to active session runtime. An idle session does not continue consuming the same active runtime fee.

So the trade-off is:

- Agent SDK: lower infrastructure abstraction and token-based cost, but more operational work
- Managed agents: more convenience and durability, with additional runtime cost

Managed agents are not automatically cheaper. They are valuable when the operational benefits justify the additional cost.

---

## Dreaming agents

Dreaming is a background process that reviews prior sessions and memory stores between active sessions.

It can:

- identify patterns
- extract useful lessons
- curate memories
- improve future behavior over time

The analogy is that the agent processes experiences in the background between tasks.

The transcript describes dreaming as a research-preview feature requiring separate access.

It should therefore not be treated as a generally available production primitive without checking current platform access and documentation.

The key idea is that an agent can improve its persistent memory without needing all learning to happen during the active user interaction.

---

## Outcomes and independent grading

Outcomes provide an evaluation loop for agent work.

The workflow is:

1. Define a rubric or success criteria.
2. Let the main agent perform the task.
3. Use a separate grader model and context to evaluate the result.
4. Return feedback to the main agent.
5. Allow the agent to make another pass.

The grader uses an independent context window instead of simply asking the same agent to judge its own work.

This reduces the bias of self-review.

It is especially useful when the agent:

- writes code and needs an independent review
- generates documents that need quality checks
- performs a task with measurable acceptance criteria
- must satisfy a structured rubric

The transcript identifies outcomes as a public-beta feature.

---

## Managed multi-agent orchestration

The course previously implemented a coordinator and subagents in a local process.

That works well for smaller workflows, but local orchestration becomes harder when:

- there are many subagents
- each subagent needs a separate sandbox
- tasks should run in parallel
- the coordinator context becomes too large
- one Python process becomes a bottleneck

Managed orchestration gives subagents independent execution contexts and session threads.

Each subagent can have its own:

- context
- sandbox
- session thread
- execution history

The subagents can run in parallel on Anthropic’s infrastructure, and the coordinator does not need to carry every subagent’s full context.

This is the managed version of the hub-and-spoke pattern discussed earlier in the series.

The transcript identifies managed multi-agent orchestration as a public-beta capability.

---

## When to choose managed agents

Managed agents are a strong fit when you need:

- durable long-running sessions
- tasks that run for minutes or hours
- crash recovery without building it yourself
- scheduled deployments
- built-in observability and audit trails
- managed sandbox execution
- persistent memory stores
- large-scale parallel orchestration
- customer-facing reliability

They are especially useful when operational convenience and durability matter more than minimizing every runtime cost.

---

## When to choose the Agent SDK instead

Use the Agent SDK when you need:

- deployment on your own infrastructure
- integration with an existing CI/CD system
- complete control over the runtime
- custom compliance requirements
- zero-data-retention requirements
- stronger control over data location
- custom networking or infrastructure behavior
- lower-level control over session and recovery behavior

Managed agents are stateful by design. They store session history, sandbox state, and outputs on the managed service.

Therefore, they may not be appropriate when organizational policy requires strict control over where data is stored or how long it is retained.

---

## Exam decision framework

A simple way to remember the choice is:

### Managed agents

Choose managed agents for:

- durability
- scale
- managed recovery
- managed environments
- built-in observability
- scheduled execution

### Agent SDK

Choose the Agent SDK for:

- control
- portability
- custom infrastructure
- compliance
- custom integration
- self-managed runtime behavior

The choice is not about which technology is universally better.

It is about which operational responsibilities you want Anthropic to own and which responsibilities your team must retain.

---

## Certification syllabus map

The episode closes by connecting the series to the certification domains.

The full course now covers:

- application and integration
- model selection and optimization
- agents and workflows
- tools and MCP
- prompt and context engineering
- security and safety
- evaluation, debugging, and testing
- managed agent runtime concepts

The exact weighting may change as the certification evolves, but the technical progression is consistent: start from the raw API, understand every important decision, and then move up to managed abstractions.

---

## Key takeaways

1. Managed agents let Anthropic run both the agentic loop and the sandbox environment.
2. The four core primitives are agents, environments, sessions, and events.
3. Agents are reusable and versioned; sessions are task-specific executions.
4. Sessions are durable and can survive client-process failures.
5. Events are the communication mechanism for managed sessions.
6. The Agent SDK remains valuable because it teaches the decisions hidden by the managed interface.
7. Managed platforms provide audit trails, credential vaults, schedules, and memory stores.
8. Dreaming is a research-preview feature for background memory improvement.
9. Outcomes use an independent grader to evaluate agent output against a rubric.
10. Managed multi-agent orchestration gives subagents independent contexts and sandboxes.
11. Choose managed agents for durability and scale; choose the Agent SDK for control and compliance.

---

## Final perspective

This episode marks the transition from building an agent to operating an agent as a service.

The course began with manually sending messages to a model. It ends with a platform where agents can be versioned, deployed, observed, resumed, scheduled, evaluated, and orchestrated.

The most important lesson is that managed abstractions do not replace foundational understanding.

They package the same decisions behind a more convenient interface.

If you understand the raw API and the Agent SDK first, managed agents become much easier to reason about because you know what the platform is managing on your behalf.
