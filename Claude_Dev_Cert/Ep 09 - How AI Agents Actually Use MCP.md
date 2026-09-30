# Ep 09 | How AI Agents Actually Use MCP

## Overview

This episode is about the Model Context Protocol, or MCP, and the core idea is that it is not a magical AI concept by itself.

At the mechanical level, MCP is a standard way for an AI system to discover and call tools that live outside the model’s own process.

The speaker explains that by the time this lesson arrives, the course has already built a bunch of tools and agents, but they were all being used in a local, in-process way.

The missing piece was this:

- how do you connect your agent to external systems?
- how do you expose tools from another service?
- how do you use a real GitHub MCP server?
- how do you build a standalone MCP server yourself?

That is the focus of this episode.

---

## The big idea: MCP is just tool access over a standard protocol

The speaker repeatedly makes the point that MCP is really about one thing:

- standardizing how an AI agent communicates with tools or data sources

In other words, MCP is not a model feature. It is an interoperability layer.

It lets the agent say:

- here is the tool I need
- here is the input shape
- here is the result format

without needing custom ad hoc wiring for every tool provider.

This is why MCP matters so much in real AI agent systems.

---

## The course recap and why this episode matters

The episode frames the series as a build-along progression:

- Episode 1: raw Messages API loop
- Episode 2: Agent SDK
- Episode 3: custom tools
- Episode 4: hooks and guardrails
- Episode 5: subagents
- Episode 6: memory and sessions
- Episode 7: structured output
- Episode 8: skills

The course has been building a PR review agent step by step.

But until now, it has mostly used mock data and local tool definitions.

This episode reveals the next layer: let the agent talk to real systems such as GitHub and let the agent use a standalone MCP server.

This is the bridge from a demo agent to a real integration-capable agent.

---

## Three different “tool worlds” in the course

The speaker explains that the course has already touched multiple ways tools can be exposed.

### 1. In-process custom tools

This is the pattern used with the SDK, where tools live directly in the same Python process as the agent.

This is simple and effective for local code and demos.

### 2. Standalone MCP server

This is a separate process or service that exposes tools via the MCP protocol.

It can be shared, reused, and connected to many clients.

### 3. Remote MCP server

This is a server that is already running somewhere else, often over HTTP or a remote endpoint.

The agent just connects to it and calls the tools it exposes.

The lesson points out that the “custom tools” built earlier were already essentially MCP-ish under the hood, even if they were never described that way at the time.

This is a useful insight: many local tool systems are really just local versions of the same underlying idea.

---

## The four MCP transport patterns

The speaker enumerates the major ways MCP servers can be exposed.

### 1. STDIO

A server starts as a local process and communicates over standard input/output.

This is very common for command-line tools and local agents.

### 2. HTTP transport

The server listens on a URL and communicates over HTTP.

This is useful when you want a remote service or a shared server.

### 3. SSE transport

This is a streaming variant of HTTP used in some older patterns.

The lecture notes that this is older and less preferred in current designs, but it still matters historically.

### 4. SDK/native in-process pattern

This is the “run the server in the same app process” approach.

That is the pattern used earlier in the course when build tools were embedded directly into the agent runtime.

This is a useful practical taxonomy for the exam and for real implementation choices.

---

## GitHub MCP server: real integration example

A large chunk of the lesson focuses on GitHub’s official MCP server.

The speaker explains that GitHub exposes a real MCP server for repository workflows such as:

- reading pull requests
- listing issues
- reviewing repository state
- accessing repository metadata
- interacting with GitHub APIs through a standard tool interface

This is exactly the kind of real-world capability agents need.

The point is that you do not always need to build everything yourself.

Sometimes the correct path is to connect to an existing MCP server and scope it to the specific tools you need.

---

## Why this is more than “just use GitHub API”

The lecturer stresses that GitHub MCP is an abstraction layer over the GitHub API using the MCP standard instead of direct custom plumbing.

That means:

- the agent sees a normalized set of tools
- tool schemas are standardized
- tool use is easier to compose across different clients
- tool access sits behind a standard protocol rather than a custom one-off integration

This is the real value of MCP. It becomes a portable integration surface.

---

## `.mcp.json` configuration

The practical side of the episode is showing how the MCP server is configured in JSON.

The configuration file declares the server details, such as:

- server name
- transport type
- URL or command
- headers or environment data for auth

This file is where you tell the agent what MCP servers are available.

The lecture makes a point that this is configuration, not Python logic.

It is not the server implementation itself. It is just how the client locates and authenticates the server.

---

## Personal access tokens and least privilege

The GitHub MCP example also reinforces a security principle that the course already emphasized earlier:

- use the least privilege you need
- never grant broad delete or admin permissions if the agent only needs read access
- keep secrets out of the repository

The speaker says the right pattern is to use a fine-grained personal access token with a narrow scope.

This matters because an agent with excessive permissions can do dangerous things if it gets misled or compromised.

This is consistent with earlier lessons around hooks, guardrails, and permission boundaries.

---

## The variable-expansion gotcha

This is one of the practical “real world” gotchas that the lecture explicitly calls out.

The speaker explains that the docs may suggest using environment variable interpolation in the MCP config, such as a placeholder value injected from a local environment variable.

But sometimes in real setups, that substitution can fail silently for HTTP header config, especially at the time of recording.

This means the literal string can be sent instead of the actual secret value.

This is the exact kind of subtle issue that can make an MCP server seem “connected” when it is not actually authorized.

The lesson deliberately frames this as a real-world debugging issue rather than a theory problem.

The practical lesson is:

- keep secrets in environment variables
- verify actual runtime behavior
- do not assume config substitution works without testing

---

## Why the GitHub tool list should be scoped

The speaker emphasizes one of the most important engineering principles in agent systems:

- do not expose every tool to the model by default
- only expose the minimum tools needed for the job

This is especially important with GitHub MCP because the server can expose many different tool categories.

The agent should not be given a giant, unrestricted tool set when it only needs a narrow subset like:

- list pull requests
- read pull request details
- inspect repository content

This keeps the system safer, cheaper, and easier to reason about.

---

## Standalone MCP server example

The second half of the episode turns to building an actual standalone MCP server.

The speaker creates a simple Python MCP server that exposes one tool like:

- count TODOs in a repo

This is a great example because it demonstrates that a standalone MCP server is not complicated in principle.

It exposes methods, uses the protocol, and returns structured tool metadata.

The important conceptual point is:

- the tool implementation is just Python code
- the server wrapper sits outside the agent and exposes it over MCP
- the agent discovers it through the config and uses it as a tool

This is a key understanding for the exam and for practice.

---

## Why the server implementation is not the same as the client config

The lecture repeatedly separates two concerns:

1. server implementation
2. client configuration

The server implementation is the Python or service code that defines the tool.

The MCP client configuration is the metadata telling the agent how to connect to the server.

These are different layers.

Understanding that separation is important because it explains how local tools, remote tools, and shared servers can all participate in the same model ecosystem.

---

## The protocol is JSON-based under the hood

Even though the concept sounds advanced, the lecture makes the mechanical side approachable.

The underlying MCP handshake is JSON-based and uses the standard tool metadata format.

In a simplified view, the client asks the server:

- what tools do you expose?
- what are the schemas?

The server responds with tool definitions and descriptions.

Then the agent calls the tool and receives structured results.

This is the real protocol flow behind the abstract “MCP” label.

---

## Large tool results and output truncation

The episode also includes a practical note: some MCP tool results can be very large.

If a result is too large, the SDK may not stuff all of it into the model context directly.

Instead, it may save the result to a file and pass only a short pointer or summary to the model.

This is important because it is a real operational behavior of MCP integrations.

The lecture notes that tool results can exceed token thresholds, and then the system falls back to file-based handling instead of dumping all content directly into context.

This explains why you sometimes see a short “output saved to a file” response instead of the full raw payload.

---

## The practical takeaway: MCP is a standard integration layer

The lecture closes by reinforcing that MCP is not a special-case feature of one framework.

It is a standard interface for tool interoperability between AI clients and tools or external systems.

In the real world, that means:

- use existing MCP servers when they fit
- build a private MCP server when you need custom functionality
- keep the tool list narrow
- store secrets securely
- verify configuration behavior in practice

That is the practical pattern the course is teaching.

---

## Final key takeaways

The speaker leaves the audience with several high-value points:

- MCP is a standard for tool integration
- tools can be local, remote, or standalone
- GitHub MCP is a real example of external tool access
- use a narrow tool list and least privilege
- `.mcp.json` is where client config lives
- standalone servers can be built with Python and exposed via MCP
- protocol mechanics are JSON-based and tool-discovery driven
- large outputs may be saved to file instead of passed directly into context

This episode is the point where the agent stops being “just a local prompt loop” and becomes something closer to a real system that can interface with external tools and repositories.

---

## Closing thought

The episode ends with a clear message: if you understand MCP, you understand how AI agents can plug into real-world systems instead of only talking to a local codebase and a few built-in tools.

That is the practical bridge from toy demos to agentic software that can actually operate in the real world.
