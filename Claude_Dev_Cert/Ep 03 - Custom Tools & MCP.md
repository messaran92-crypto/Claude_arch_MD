# Ep 03 | Custom Tools & MCP

## Overview

This episode is about the moment when built-in tools stop being enough.

In the previous episodes, the agent used the raw Messages API and then the Claude Agent SDK. The basic workflow was already clear:

- send a prompt
- let Claude decide whether it needs tools
- use tools if required
- inject results back into the loop
- continue until done

But the speaker emphasizes that once your business logic becomes more specific, the built-in toolset is no longer enough. You need your own custom tool logic.

That is the theme of this episode: how to build a custom tool, expose it to the agent, and structure it so Claude can decide when to use it.

---

## Why custom tools are needed

Built-in tools are great for generic operations like:

- reading files
- writing files
- searching text
- shell commands
- simple repo navigation

But they are not enough for domain-specific tasks such as:

- checking whether functions have docstrings
- validating code quality heuristics
- checking PR patterns
- analyzing architecture conventions
- scanning custom business logic in a codebase

The speaker gives a concrete example: a general grep or read tool can find a function or symbol, but it cannot tell whether that function is missing documentation. That requires custom logic.

This is exactly where a custom tool becomes necessary.

---

## The core mental model

The most important concept in this episode is that custom tools are not just functions—they are structured capabilities that Claude can decide to call.

A custom tool includes four major parts:

1. name
2. description
3. input schema
4. handler function

The speaker stresses that the description is extremely important. Claude does not read your code directly; it reads the tool description to decide whether it should use it.

This is like writing a precise job description. A vague description leads to wrong tool selection. A clear description leads to the correct tool at the right time.

---

## The four parts of a custom tool

### 1. Name

This is the unique identifier Claude uses to call the tool.

It should be specific and consistent.

### 2. Description

This is the most important part.

The description tells Claude:

- what the tool does
- when to use it
- when not to use it
- what kind of result it returns

A good description is precise and practical. A weak description causes misrouting.

### 3. Input schema

This defines what arguments the tool accepts.

For example, a tool that checks a Python file for missing docstrings may accept a file path argument.

The schema is the contract that tells Claude what it must provide to the tool.

### 4. Handler

This is the actual Python function that executes the logic.

In the SDK, it is usually an async function that receives validated arguments and returns structured output.

---

## Python annotations and metadata

The speaker introduces the use of Python type hints combined with metadata through annotations.

This is not Anthropic-specific; it is standard Python behavior.

Example concept:

```python
from typing import Annotated

path: Annotated[str, "Path to the Python file to inspect"]
```

This pattern provides two benefits:

- Python knows the parameter type
- the SDK can use the metadata as readable input descriptions for Claude

This is important because many people simply paste the same code without understanding what these annotations are doing.

---

## The example tool: check docstring coverage

The episode uses a custom tool named something like `check_docstring_coverage`.

This tool checks a Python file to find:

- functions without docstrings
- classes without docstrings
- missing documentation at specific line numbers

The speaker explains why this cannot be done well with built-in tools alone.

`grep` or `read` can locate symbols, but they do not understand Python structure in the same way a custom parser can.

The custom tool uses Python introspection and parsing logic instead of text matching alone.

---

## Why the repo root matters

The speaker explains that custom tool logic often needs to resolve paths relative to the repository root.

This is a common source of confusion.

If the tool runs in a working directory that is not the project root, a file path like `episode_1/agent.py` may not resolve correctly unless the code explicitly handles it.

So the custom tool logic uses a repo-root resolution strategy so it can reliably locate files in a structured project folder.

This is a subtle but important detail in real agent code.

---

## The tool should fail gracefully

The speaker explicitly emphasizes that custom tool handlers should not crash with a raw Python traceback when something goes wrong.

Instead, the handler should return a structured error payload, such as:

- `is_error: true`
- a human-readable message
- the relevant context

This is important for agent orchestration, because if a tool fails silently or throws an unhandled exception, the outer agent cannot reason effectively about the failure.

The speaker ties this back to agent coordination and error propagation concepts discussed earlier in the architecture course.

---

## Optional parameters and private methods

The tool includes logic for optional parameters, such as whether to include private methods or functions.

The speaker explains that optional parameters are often represented in code like this:

```python
include_private = args.get("include_private", False)
```

This avoids making every argument mandatory.

The example tool defaults to skipping private methods unless explicitly requested.

This is a nice practical pattern for custom tools: use optional filters to keep behavior flexible without forcing Claude to always pass every argument.

---

## Packaging as an MCP tool

Once the custom tool is written, it must be exposed through an MCP-style server.

The speaker explains that an MCP server is basically a bundle of tools that Claude can discover.

When you register a custom tool, the SDK is effectively telling the agent:

- this is a valid tool
- here is its description
- here are the arguments it expects
- this is the function to call when selected

The code pattern for tool registration is discussed in the lesson. A tool inside a server is represented by a structured name pattern, typically similar to:

```python
server_name__tool_name
```

This is how the client identifies which sub-tool belongs to which MCP server during discovery.

---

## Why this matters for the agent

The agent is not just a model call. It is a runtime that binds together:

- prompt
- system instructions
- tool registry
- current working directory
- permission policy
- tool descriptions

Custom tools plug into that runtime so the agent can do work beyond generic text operations.

This is what turns a simple LLM into a task-capable agent.

---

## Agent setup with custom tools

The agent code is then updated so that it imports the custom tool server and attaches it to the runtime configuration.

The speaker shows how to configure:

- the model
- the MCP server(s)
- allowed tools
- working directory
- system prompt
- max turn limit

This is the agent-side configuration that allows the custom tool to be used automatically when needed.

---

## Allowed tools and permissions

The episode revisits the concept of `allowed_tools`.

The speaker reminds the audience that this is not the same as “restrict Claude to only this tool.”

It simply means:

- do not ask for confirmation before using this tool
- pre-approve it for this run

This is useful when you want a tool to be used automatically without blocking the workflow.

The important distinction is: allowed tools are about permission flow, not total restriction.

---

## Why tool descriptions matter so much

The speaker comes back to one of the central themes of the course:

> if Claude picks the wrong tool, the agent fails even if the tool itself works.

This is why the tool description must be carefully written.

A good description should say:

- what the tool is for
- when to use it
- when not to use it
- what it returns
- any constraints or assumptions

This is a real-world example of the “prompting” problem showing up at the tool layer.

---

## The real output from the demo

The speaker runs the agent against the code from earlier episodes and gets a useful review.

The tool checks docstring coverage on previous files and reports issues such as:

- a function missing a docstring
- a main execution loop missing documentation
- some missing coverage that should be fixed before merging

The result is not necessarily a blocker, but it demonstrates the usefulness of custom tools in a realistic code-review workflow.

This is a clear demonstration of how a custom tool can contribute actual task value beyond built-in read/search tools.

---

## Key takeaways from this episode

### 1. Built-ins are not enough for domain logic

Use custom tools when the task requires semantics that general-purpose file tools cannot provide.

### 2. Tool descriptions are critical

Claude decides whether to call a tool based largely on the description.

### 3. Custom tools should be structured and explicit

Name, description, schema, and handler are the essential parts.

### 4. Errors should be structured, not raw crash traces

A tool should return a proper error payload rather than crashing the whole flow.

### 5. MCP is how tools are exposed to the agent

You do not just define a Python function—you register it in a tool runtime that the agent can discover and use.

### 6. allowed_tools simplifies permission flow

It pre-approves a tool without making it the only tool available.

---

## Final summary

Episode three introduces custom tools and MCP integration as the next major step in building a practical agent.

The move from built-in tools to custom tools is the point where the agent becomes specialized to the developer workflow rather than merely generic chat behavior.

The real lesson is that a tool is not just a Python function. It is an interface contract between the agent and the business logic.

When the description is clear, the schema is precise, and the handler is structured, Claude can use the tool intelligently and reliably.

---

## Key message from the episode

The best way to think about this lesson is:

> Built-in tools solve generic tasks; custom tools solve your real workflow.

And when you combine those with MCP, you are creating the actual foundation of a capable domain-specific agent.
