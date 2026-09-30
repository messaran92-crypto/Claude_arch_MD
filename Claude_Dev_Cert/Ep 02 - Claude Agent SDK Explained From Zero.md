# Ep 02 | Claude Agent SDK Explained From Zero

## Overview

This episode is the transition from the raw Messages API to the Claude Agent SDK. In the previous episode, we manually built the full agentic loop using raw API calls, handling:

- message appends
- stop reason checks
- tool call execution
- tool result injection
- loop repetition until the model is done

In this episode, the speaker explains that instead of writing all of that boilerplate by hand, we can use the SDK to manage the agent loop for us while we focus on business logic.

The core idea is simple:

- raw Messages API = maximum control, more code
- Claude Agent SDK = lower ceremony, faster agent development

This is the “buy the car instead of building the engine” metaphor used in the lesson.

---

## The big transition: from raw loop to SDK

In episode one, the agent used only one custom tool: `read_repo_file`.

That code did the following:

- sent a user prompt to Claude
- checked the response `stop_reason`
- if the stop reason was `tool_use`, executed a function
- appended the result back into the message history
- repeated until Claude answered with `end_turn`

This was a fully manual loop.

The speaker emphasizes that this is powerful, but also expensive in terms of maintenance and boilerplate. For many real-world cases, you do not want to maintain the loop itself. You want to focus on the agent’s high-level behavior and let the SDK handle the infrastructure.

---

## Why the SDK is useful

The Claude Agent SDK gives you:

- an internal agent loop
- built-in tool orchestration
- built-in state handling
- safety controls like `max_turns`
- token and cost tracking
- easier execution of multi-step reasoning workflows

This is especially helpful when you want to build a usable agent without rebuilding the session machinery every time.

---

## The “old” vs “new” SDK naming confusion

The speaker warns viewers not to get confused by old names and docs.

The course explains that:

- old package / older tutorials may refer to the “Claude Code SDK”
- newer versions are called the “Claude Agent SDK”
- some docs may still mention old classes like `ClaudeCodeOptions`
- the modern class is now `ClaudeAgentOptions`

This is an important practical note because old tutorials and examples can be stale and may not match the current API.

The speaker strongly suggests following the current syntax used in this course rather than outdated blog content.

---

## First test: basic query call

Before getting into the actual business problem, the speaker runs a quick sanity check with a very small query:

```python
from claude_agent_sdk import query

async def main():
    result = await query(
        "What is 2 + 2?"
    )
    print(result)
```

The result is simple, but it reveals an important lesson: if you do not specify the model, the SDK may default to a more expensive model.

In this example, the model defaults to Claude Opus 4.8, which is not always desirable for regular development or cost-sensitive work.

This is why the speaker insists that you should explicitly set the model instead of relying on defaults.

---

## Important setup requirements

The speaker reminds the audience that the SDK needs:

- Python 3.10 or above
- Node 18 or above for the JavaScript/TypeScript route
- a valid Anthropic API key in the environment
- the SDK installed in the active virtual environment

The installation command is shown as:

```bash
pip install claude-agent-sdk
```

The package includes the CLI tooling required by the SDK, so there is no separate install for the runtime in this setup.

---

## What the SDK gives you automatically

When you run a query, the SDK emits several message types internally. The speaker explains six major message types, but the most important ones are:

- system message
- assistant message
- user message
- result message
- stream event
- rate limit event

For most practical work, the main ones to care about are:

- system message: initialization and tool context
- assistant message: Claude’s actual reply or tool call
- user message: the input echo / tool result feed
- result message: final summary with cost, tokens, duration, and exit state

This is the SDK equivalent of the manual raw API lifecycle.

---

## The single biggest bug to avoid

The speaker calls out a very common mistake: indexing into content blocks by position.

For example, this is unsafe:

```python
response.content[0]
```

because the content array may contain:

- text blocks
- thinking blocks
- tool_use blocks
- tool_result blocks

The order is not guaranteed. The correct pattern is to inspect each block by type instead of assuming position.

Example:

```python
for block in message.content:
    if block.type == "text":
        print(block.text)
```

This is exactly the same discipline that mattered in the raw Messages API example from episode one.

---

## Claude Agent Options

The main configuration object in this SDK is `ClaudeAgentOptions`.

The speaker lists the six fields that matter most:

### 1. model

This is the most important field to set explicitly.

If you omit it, you may silently default to a very expensive model.

Example:

```python
model="claude-sonnet-4"
```

or whichever model you want to lock in.

### 2. allowed_tools

This is a pre-approval allowlist. It does not strictly restrict the toolset in a hard way.

It means: “do not interrupt me for permission when these tools are used.”

This is not the same as “Claude can use only these tools.”

### 3. cwd

This is the current working directory. It tells the agent where it should operate from when it reads files or runs commands.

### 4. system_prompt

This is the instruction that shapes the agent’s persona and constraints.

Example:

```python
system_prompt="You are a senior Python reviewer. Be concise."
```

### 5. permission_mode

This controls how tool access is handled.

Examples include:

- prompt for permission
- auto-approve selected tools
- full auto-approve mode, depending on setup

### 6. max_turns

This is a safety valve. It limits how many turns the agent can run before stopping.

The speaker stresses that this is not the primary stop condition. The real stopping signal is still the model’s structured output and the agent loop semantics, not just a hard-coded turn count.

---

## Important truth about allowed tools

This is treated as one of the biggest conceptual traps in the lesson.

The speaker explains:

- `allowed_tools` does not mean “Claude can only use these tools”
- it means “these tools are pre-approved and won’t trigger a permission prompt”
- the rest of the tools still exist in the environment
- permission mode still decides how the rest behave

This distinction matters because many people misunderstand the tool gate.

Example:

- `allowed_tools=["read"]`
- Claude can still technically use other tools if permission mode allows it

If you truly want hard restrictions, you need stronger controls than `allowed_tools` alone.

---

## Built-in tools in the SDK

The speaker highlights that the Claude Agent SDK ships with a broad built-in toolset.

Examples include:

- read
- write
- edit
- bash
- grep
- glob
- web search / fetch
- human-in-the-loop prompts

This means you often do not need to hand-roll basic file or shell functionality unless your use case needs something unique.

The main reason to build custom tools is when the capability is domain-specific, such as:

- PR summarization
- repo validation
- business logic checks
- custom API integrations

---

## A real example: same task, much less code

The episode then rebuilds the same kind of task from episode one, but with the SDK.

The goal is to ask the agent: “What does this file do?”

Instead of manually managing message history, the code is much simpler:

```python
from claude_agent_sdk import query
from claude_agent_sdk.options import ClaudeAgentOptions

async def main():
    result = await query(
        prompt="What does this file do?",
        options=ClaudeAgentOptions(
            model="claude-sonnet-4",
            allowed_tools=["read"],
            cwd=".",
            system_prompt="You are a senior Python reviewer. Be concise.",
            max_turns=5,
        )
    )

    for message in result.messages:
        if message.type == "assistant":
            for block in message.content:
                if block.type == "text":
                    print(block.text)
```

The point is not the exact syntax, but the dramatic reduction in complexity.

The SDK handles the agentic loop and tool orchestration for you.

---

## What the output looks like

The speaker runs the query and gets a final result summarizing the code from the earlier episode.

The output includes:

- final answer text
- number of turns
- cost information
- whether there was an error
- token usage summary

This reinforces that the SDK is not just a wrapper around a model call. It is a full orchestration layer.

---

## The practical lesson

The speaker’s key message is:

> The SDK removes the need to maintain the agent loop manually.

This matters because the raw API is good for learning, but the SDK is good for shipping.

In other words:

- raw API = understand the machine
- SDK = use the machine efficiently

The speaker is encouraging a layered learning path:

1. build a manual loop once
2. understand stop reasons and tool flow
3. switch to SDK for production-grade development

---

## Final takeaways

### 1. The raw loop was the foundation

Without understanding the manual loop, the SDK can look like magic.

### 2. The SDK is a real agent runtime

It manages the tool loop, message flow, and turn system for you.

### 3. You still need to understand the concepts

Even with the SDK, the same ideas remain:

- tools
- stop reasons
- message ordering
- state management
- safe usage patterns

### 4. Default model choice matters

Setting the model explicitly prevents accidental spending on expensive models.

### 5. `allowed_tools` is not the same as total restriction

It is a convenience feature for approval flow, not a hard security boundary by itself.

### 6. Built-in tools save time

Use the SDK’s standard tools whenever possible and build custom tools only for business-specific capabilities.

---

## Closing summary

Episode two introduces the Claude Agent SDK as the abstraction layer that replaces the manually implemented agentic loop from episode one.

The key message is that the raw Messages API teaches the mechanics, while the SDK handles the orchestration.

Once you understand how the loop works, the SDK becomes a huge productivity win.

The next step is to move beyond built-in tools and start adding custom business tools that are specific to your project.

---

## Key phrase from the episode

The best summary of the lesson is this:

> In episode one, we built the engine by hand. In episode two, we learned to drive the car.

This is the essence of the shift from low-level orchestration to high-level agent usage.
