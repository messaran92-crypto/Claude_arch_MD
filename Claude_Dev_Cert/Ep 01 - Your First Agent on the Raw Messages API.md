# Ep 01 | Your First Agent on the Raw Messages API

## Overview

This episode introduces the raw Messages API and builds a very first tool-calling agent in a simple, hands-on way. The goal is to understand the agentic loop rather than hide it behind an SDK.

The course is framed as a practical developer path for Claude-based agents, and this episode focuses on the foundational mechanics:

- creating a client
- sending a message with tool definitions
- letting Claude decide which tool to call
- executing the tool in your app
- returning the tool result to Claude
- continuing the loop until stop_reason says the task is complete

This is the raw API version of the workflow, before using a higher-level SDK abstraction.

---

## Why use the raw Messages API?

The speaker emphasizes that raw Messages API is useful when you want:

- complete control over the loop
- a simple use case to learn from
- a deeper understanding of how agent behavior works under the hood

This is a perfect starting point if your goal is to build real agent logic and understand how tools, messages, and stop reasons interact.

---

## Project structure and learning approach

The course is structured episode by episode with project folders and code snippets. Each episode includes:

- explained code
- an incremental build
- a practical example you can run step by step
- a deeper explanation of why each part exists

The idea is not to blindly ask Claude to create an entire agent and trust it. Instead, you should understand every step in the loop.

---

## Setup requirements

Before coding, you need:

- Anthropic SDK installed
- a valid Anthropic API key
- the key exported in your environment

Typical setup:

```bash
pip install anthropic
export ANTHROPIC_API_KEY="your_key_here"
```

On macOS/Linux, you may also add the variable to a shell profile like `.zshrc` or `.bashrc` so it persists.

The client is initialized as:

```python
from anthropic import Anthropic
client = Anthropic()
```

This client reads the `ANTHROPIC_API_KEY` environment variable automatically.

A quick sanity check is to run a small script and confirm the API key is being picked up correctly.

---

## The first tool: read repo file

The first capability added to the agent is a simple tool called `read_repo_file`.

Its purpose is to read the contents of a file from a repository given a relative path.

Example pseudo-structure:

```python
REPO_FILES = {
    "README.md": "# Demo Repo\n...",
    "src/utils.py": "def helper():\n    return 'hi'",
}


def read_repo_file(path: str):
    if path in REPO_FILES:
        return REPO_FILES[path]
    return {"error": f"File not found: {path}"}
```

This is intentionally kept simple so the focus stays on the agent loop and not on repository logic.

---

## Claude does not call Python functions directly

This is one of the most important ideas in the raw API workflow.

Claude never sees your Python function definition directly. It only sees a JSON tool schema like this:

```python
tools = [
    {
        "name": "read_repo_file",
        "description": (
            "Read the full text contents of a single file from the demo repository "
            "given its path relative to the repo root. Use this whenever you need to see "
            "a file's contents before answering a question about it. Do not guess at "
            "file contents you have not read."
        ),
        "input_schema": {
            "type": "object",
            "properties": {
                "path": {"type": "string"}
            },
            "required": ["path"]
        }
    }
]
```

The key fields are:

- `name`: the tool name Claude must call
- `description`: explains when to use it
- `input_schema`: defines the expected arguments

This is where a lot of agent performance comes from. If the description is weak or ambiguous, Claude may choose the wrong tool or hallucinate answers.

---

## A first Messages API call

The first message to Claude looks like this:

```python
response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    tools=tools,
    messages=[
        {
            "role": "user",
            "content": "What does src/utils.py do?"
        }
    ]
)
```

The important concept: the model does not answer directly if it needs information. Instead, it responds with a tool call request.

The result often contains:

- stop_reason = `tool_use`
- content block with type `tool_use`
- a tool name like `read_repo_file`
- input arguments like the file path

This is exactly where the agentic loop begins.

---

## The first tool-use response

Claude's response is not the final answer. It is a request to perform an action.

For example, Claude may say:

- “I need to read the file to answer this accurately.”
- then emit a `tool_use` block for `read_repo_file`

At this point, your app must execute the actual Python function corresponding to that tool.

That is the critical handoff:

1. Claude decides which tool to use
2. your code executes it
3. your code sends the result back to Claude

---

## Why the agentic loop matters

The raw Messages API is stateless from the model’s point of view. Claude does not remember previous conversation history unless you send it again in the next request.

This is the key mental model:

> Claude remembers nothing; you remember everything.

So every time you continue the loop, you append the new messages to the `messages` array and send the whole history back.

That means the conversation is built like this:

```python
messages = [
    {"role": "user", "content": "What does src/utils.py do?"}
]
```

After Claude requests a tool, you do:

```python
assistant_message = response
```

Then you execute the tool and append the result as a new message in the proper order.

---

## Correct message ordering is essential

The order of messages is non-negotiable.

The right flow is:

1. user message
2. assistant message with tool call request
3. tool result message
4. next assistant response

If you append the tool result before the assistant request, or otherwise break the sequence, the API may reject the request or produce invalid behavior.

This is a common bug in agent code.

The tool result message must match the earlier tool_use block ID, and it must be layered in the right order.

---

## The loop structure

The core loop is simple:

```python
while True:
    response = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        tools=tools,
        messages=messages
    )

    stop_reason = response.stop_reason

    if stop_reason == "tool_use":
        # extract tool call from response
        # execute matching tool
        # append assistant message + tool result to messages
        continue

    if stop_reason == "end_turn":
        print(response.content)
        break
```

This is the heart of the agentic loop.

---

## stop_reason is the real control signal

The most reliable way to control the loop is using `stop_reason`, not guessing based on the content or trying to parse freeform text.

Typical values include:

- `tool_use`: Claude wants to use a tool
- `end_turn`: the task is done and Claude is returning a final answer
- `max_tokens`: the model hit the token limit
- `refusal`: Claude refused the request

The loop should act based on these structured values.

---

## Example of the tool result handoff

Once Claude requests a tool, your code extracts the tool call:

```python
for block in response.content:
    if block.type == "tool_use":
        tool_name = block.name
        tool_args = block.input
```

Then you run the real Python function:

```python
if tool_name == "read_repo_file":
    result = read_repo_file(tool_args["path"])
```

Finally, you append the tool result back to the conversation:

```python
messages.append({
    "role": "assistant",
    "content": response.content
})

messages.append({
    "role": "user",
    "content": [
        {
            "type": "tool_result",
            "tool_use_id": block.id,
            "content": str(result)
        }
    ]
})
```

This is how the tool output is fed back to Claude, allowing it to reason about the result and answer correctly.

---

## What happens after the tool result is returned?

Once the tool result is appended, you send the full conversation back to Claude again.

At this point, Claude can now reason using the actual file contents and produce a final answer.

For example, if the user asked “What does `src/utils.py` do?”, Claude may inspect the file, determine its purpose, and produce a concise explanation.

The final response is typically an `end_turn` with a normal assistant answer.

---

## Important takeaways from this episode

### 1. The raw Messages API is the lowest-level, most controllable approach

It helps you learn the exact mechanics of agentic behavior.

### 2. Tool definitions must be explicit and well written

The `name`, `description`, and `input_schema` directly shape how Claude chooses tools.

### 3. Claude is stateless

You must send the full message history every time.

### 4. stop_reason drives the loop

Do not guess whether the model is done. Let the structured response tell you.

### 5. Message ordering matters

Tool responses must come after the tool call they answer.

### 6. This is not chat memory — it is state management

You are manually managing the conversation state in your own application.

---

## Final summary

This episode is the first real step into building a tool-calling agent with Claude using the raw Messages API. The core idea is simple but powerful:

- define a tool
- send a user message with the tool list
- let Claude decide to call the tool
- execute the tool in code
- send the result back to Claude
- repeat until the model decides it is done

That loop forms the foundation of much more advanced agent systems, including repo review agents, PR analysis agents, and multi-step autonomous workflows.

This is where real agent understanding begins.

---

## Key message from the episode

The real lesson is not merely “how to call Claude.” It is:

> You are designing an agent loop, not just prompting a model.

And that loop is built from messages, tool definitions, structured outputs, and a careful understanding of stop reasons.
