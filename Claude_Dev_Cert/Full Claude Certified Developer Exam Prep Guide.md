# Full Claude Certified Developer Exam Prep Guide

## Purpose of the exam

The Claude Certified Developer exam is about practical AI-assisted software development, not only basic knowledge of Claude.

The developer-level expectation is that you can build, integrate, evaluate, and ship production-grade Claude applications, agents, and workflows.

You are expected to make engineering decisions that keep systems:

- reliable
- affordable
- secure
- observable
- maintainable
- useful in real development workflows

The central mindset shift is this:

> Claude is not only a chatbot. It is a probabilistic software collaborator and, when given tools, an agent that can act on external systems.

---

## Exam structure and preparation

The transcript describes an exam with:

- 120 minutes
- a passing score represented as 720 in the course material
- a credential validity period of 12 months

The exact exam details can change, so verify current official information before scheduling the exam.

Anthropic preparation areas mentioned in the lesson include:

- foundations
- model selection
- production-grade prompting
- Claude Code
- MCP integration
- production engineering
- events or accelerators
- intellectual-property contribution

The course groups preparation into six practical areas:

1. Claude foundations and certification basics
2. Working effectively with Claude
3. Claude-assisted software development
4. Developer workflows and agentic usage
5. Enterprise scenarios
6. Production engineering, evaluation, and security

---

## Domain overview

The transcript highlights these approximate exam domains:

| Domain | Approximate emphasis |
|---|---:|
| Application and integration | 33% |
| Model selection and optimization | 16% |
| Agents and workflows | 14% |
| Prompt and context engineering | 11% |
| Tools and MCP | 10% |
| Security and safety | 8% |
| Claude Code | 3% |
| Evaluation, testing, and debugging | 3% |

The percentages are approximate study guidance. Always confirm the current official exam guide because the blueprint can change.

The most important study implication is that application and integration receives the largest emphasis. Learn how Claude fits into a real software system, not only how to write a prompt.

---

## 1. Claude foundations for developers

### Model tiers

The course uses three broad model tiers:

- Opus: strongest reasoning and highest cost or latency
- Sonnet: balanced general-purpose workhorse
- Haiku: fastest and most economical for lightweight reasoning

Choose the model based on the constraint.

Examples:

- real-time chat with subsecond latency: prefer a fast model such as Haiku
- ordinary application workflows: start with Sonnet
- complex planning or difficult reasoning: consider Opus
- simple routing, classification, and extraction: use Haiku when quality is sufficient

Do not automatically choose the largest model. Model selection is an engineering trade-off between quality, latency, cost, and reliability.

### Tokens

Tokens are the basic billing and context unit.

Input tokens include:

- system instructions
- conversation history
- user content
- tool definitions
- retrieved documents
- tool results

Output tokens include:

- generated text
- tool calls
- structured responses
- other generated content

Output tokens generally cost more than input tokens. Concise, constrained responses are therefore usually cheaper.

A rough rule of thumb is that 1,000 tokens may represent around 750 English words, but tokenization varies by language and content type.

### Context windows

A context window is the model's working budget for one request or session.

It is not durable memory.

A long-running agent must manage:

- history
- documents
- tool results
- system instructions
- retrieved memory
- context compaction

Do not pass the full history or every document into every step. Retrieve and include only what the current task requires.

### Sampling and nondeterminism

Claude outputs are probabilistic. The same prompt can produce different results across runs.

For stable output:

- lower randomness where the platform supports it
- provide an explicit output schema
- constrain the format
- use examples
- validate the result
- design deterministic checks around the model

Determinism comes from system design, not from relying on the model alone.

---

## SDK versus REST API

Prefer an official SDK when one is available and suitable because it commonly provides:

- typed request and response models
- retry behavior
- streaming support
- authentication helpers
- convenience abstractions
- integration with the platform's conventions

Use the raw REST or Messages API when:

- no official SDK is available for the environment
- you need low-level control
- you are implementing a custom abstraction
- you need to understand or customize the raw protocol

The raw API gives control, but you must implement more behavior yourself, such as retries, message sequencing, and tool-loop handling.

---

## 2. Production prompt engineering

A production prompt should communicate:

1. Role
2. Task
3. Context
4. Constraints
5. Expected output

A useful structure is:

- explain who or what the model is acting as
- state the task precisely
- provide only relevant context
- define constraints and boundaries
- specify the output shape
- include examples when the behavior benefits from them

### Explicit output schemas

If downstream code must parse the response, use a strict output format.

Prefer:

- JSON schema
- typed fields
- enumerated values
- required fields
- clear nullability rules
- no-extra-text instructions

Free-form output may sound helpful but is harder to validate and more expensive to process.

### Few-shot examples

Provide one or more representative input and output examples when:

- the format is unusual
- category names must be exact
- the task has subtle edge cases
- the model needs to imitate a specific style

Examples should be correct, minimal, and representative.

### Delimit untrusted content

Use clear boundaries around user-provided text, documents, or tool output.

XML-like tags, JSON fields, or other explicit delimiters help separate:

- instructions
- reference material
- untrusted user content
- tool results

Delimiters do not make content automatically safe, but they make the intended authority boundaries clearer.

### Diagnose a weak prompt

When a prompt underperforms:

1. Read the actual output.
2. Identify the failure.
3. Determine whether the cause is missing context, ambiguity, weak examples, missing format, or excessive context.
4. Change one variable.
5. Run the prompt again.
6. Compare the result.

Do not change the prompt, model, temperature, tools, and output format simultaneously. You will not know what fixed or caused the behavior.

### Relevance beats volume

More context is not automatically better.

Include what the model needs and omit unrelated material. Extra context increases cost, consumes the context budget, and may distract the model from the actual task.

---

## 3. Claude across the software development lifecycle

The developer exam is not limited to code generation.

Claude can support the full developer workflow:

1. Understand an unfamiliar codebase
2. Generate code that fits the project
3. Debug failures
4. Design and generate tests
5. Refactor safely
6. Review and document changes

Human verification remains important at every stage.

---

## Understanding an unfamiliar codebase

A common scenario is joining a large, undocumented, or legacy codebase.

Do not immediately ask Claude to change code without first building a mental model.

Provide real files and ask Claude to:

- summarize the architecture
- identify entry points
- map request and data flows
- identify key modules
- list dependencies
- locate risky areas
- surface questions that require verification

Then compare the summary against the actual source code.

Claude can accelerate understanding, but its explanation is not proof that the code behaves that way.

### Greenfield versus brownfield

- Greenfield: building a new system from the beginning
- Brownfield: modifying an existing system with existing behavior and constraints

Brownfield work requires extra attention to compatibility, conventions, dependencies, and regression risk.

---

## Generating code that fits the project

Weak request:

> Write a function to upload a file.

Stronger request:

- provide the relevant files
- specify the language and version
- identify the framework and libraries
- describe existing conventions
- define error-handling requirements
- specify validation rules
- state security constraints
- define tests and expected behavior

Generated code can be plausible but still be wrong for the project if the request omits its actual constraints.

Treat Claude as a fast engineering teammate. Give it the same context and conventions that a human teammate would need.

---

## Debugging with Claude

For a difficult failure, provide evidence rather than saying only that something is broken.

Useful evidence includes:

- failing test
- stack trace
- relevant source code
- recent diff
- dependency changes
- environment details
- representative input
- logs
- timing or reproduction information

Ask Claude for:

- ranked hypotheses
- evidence supporting each hypothesis
- a minimal experiment to distinguish them
- a proposed fix after confirmation

A good result is a short list of testable hypotheses, not an overconfident single answer.

For nondeterministic or intermittent failures, compare runs and isolate changes systematically.

---

## Testing and test design

Claude can generate tests quickly, but more tests do not automatically mean better coverage.

Ask for coverage of:

- normal behavior
- boundary values
- invalid input
- empty input
- missing fields
- permission failures
- dependency failures
- timeout behavior
- concurrency or ordering concerns
- recovery paths

Prefer tests that verify behavior and contracts over tests that assert private implementation details.

A useful workflow is:

1. Ask for a test-case table.
2. Review equivalence classes and boundary values.
3. Identify missing error paths.
4. Generate the tests.
5. Run them against the real implementation.
6. Review failures and gaps.

---

## Safe refactoring

A large untested function is risky to refactor because behavior is not documented by tests.

The safe sequence is:

1. Understand the current behavior.
2. Generate characterization tests.
3. Run the tests and confirm they describe the current system.
4. Refactor in small steps.
5. Run the test suite after each step.
6. Review the final diff.

The goal of refactoring is to improve structure without changing observable behavior.

Characterization tests create a safety net before modularizing validation, business logic, I/O, or other mixed responsibilities.

---

## Review, documentation, and API work

Claude can help with:

- summarizing a diff
- identifying correctness risks
- spotting security concerns
- drafting a pull request description
- writing docstrings
- creating architecture notes from real code
- drafting handlers and endpoints
- defining request and response schemas

However, confident language is not verification.

Before publishing a review or documentation:

- check every claim against the source code
- confirm examples actually work
- verify API behavior
- run tests or validation tools
- keep a human reviewer involved for high-impact changes

---

## Human-in-the-loop judgment

Use Claude freely for tasks where mistakes are easy to inspect and reverse:

- explaining unfamiliar code
- drafting boilerplate
- generating test ideas
- enumerating edge cases
- proposing refactors
- summarizing changes

Keep tighter human control for:

- security-critical logic
- financial calculations
- exact numerical rules
- irreversible actions
- production data changes
- external claims that must be factually verified
- credential and permission decisions

The correct mindset is trust but verify, with stricter verification as impact increases.

---

## 4. From a single call to an agent loop

A single model call answers one turn.

An agent runs a loop:

1. Reason about the current state.
2. Choose an action.
3. Call a tool or produce a result.
4. Observe the tool result.
5. Continue until the task is complete.

Agents are appropriate for multi-step tasks that:

- interact with external systems
- require intermediate decisions
- need multiple tool calls
- must adapt based on returned data

Each loop iteration can involve another model call, so loops affect both cost and latency.

### Healthy versus runaway loops

A healthy loop completes in a small number of purposeful steps.

A runaway loop often indicates:

- broken tool behavior
- unclear stopping conditions
- missing success criteria
- invalid or incomplete tool results
- a coordinator that cannot recognize completion

Set boundaries with:

- maximum turns
- timeouts
- retry limits
- clear completion criteria
- structured tool results
- cancellation behavior

---

## Tool-use mechanics

The standard tool-use loop is:

1. Claude decides that a tool is needed.
2. Claude emits the tool name and structured arguments.
3. Your application intercepts the request and executes the real function or API call.
4. Your application returns the tool result to Claude as a new message.
5. Claude observes the result and either calls another tool or produces the final answer.

The model does not execute your function by itself. Your code or runtime executes the function.

Tools should have:

- clear names
- precise descriptions
- strict input schemas
- validation
- authorization checks
- structured success and error results

---

## MCP integration

The Model Context Protocol is a standard way to expose tools, data, and prompts to Claude applications.

An MCP server can front:

- internal APIs
- databases
- GitHub
- Slack
- testing systems
- enterprise services
- browser or automation capabilities

MCP creates a reusable boundary between the model and enterprise systems.

The model requests a capability through a tool, while the MCP server handles the underlying integration.

This enables teams to build an integration once and reuse it across applications and agents.

### MCP security

Never place API keys or secrets in prompts.

A safer pattern is:

1. Store credentials in a server-side secret manager or environment configuration.
2. Keep the secret inside the MCP server or integration layer.
3. Let Claude request a capability such as `get_invoice`.
4. Have the MCP server authenticate when calling the real billing API.
5. Return only the required result to the model.

The secret should not enter the model context, prompt, logs, or traces.

---

## Context, memory, and scope

### In-context memory

Information included in the current request or conversation is immediately available, but it competes for the fixed context budget.

As history grows, the system may become:

- more expensive
- slower
- harder to control
- vulnerable to context drift

### External memory

Long-term state can be stored in:

- databases
- vector stores
- files
- session stores
- managed memory services

Retrieve only the information relevant to the current step.

### Scope discipline

Give each agent or subtask only the context it needs.

Narrow context is generally:

- cheaper
- faster
- easier to reason about
- less likely to leak unrelated information
- more reliable

This is especially important in multi-agent systems.

---

## Planning, decomposition, and approval

For large or risky tasks:

1. Ask Claude to produce a plan.
2. Review and approve the plan.
3. Decompose the work into small steps.
4. Make each step testable and reviewable.
5. Keep a human in the loop for irreversible or high-impact actions.

Fixing a flawed plan is cheaper than fixing a wrong implementation after many files have changed.

Subagents can help with decomposition when tasks have clear boundaries and specialized responsibilities.

---

## Claude Code, project rules, and skills

Claude Code is an agentic coding tool that can read, write, and run code under a permission model.

Important concepts include:

- permission approval before risky actions
- durable project instructions
- reusable skills
- user-level and project-level configuration
- tool and MCP access

A project rules file gives the agent durable context about:

- architecture
- coding conventions
- testing commands
- security rules
- deployment constraints

A skill packages a repeatable workflow or capability so it can be reused by a team.

The practical progression is:

- use project rules for durable guidance
- use skills for reusable workflows
- use MCP for external capabilities
- use permissions and human approval for risky actions

---

## 5. Production evaluation

An evaluation, or eval, is a repeatable test of Claude behavior against a representative dataset.

A production evaluation loop is:

1. Define what success means.
2. Build a labeled or representative test set.
3. Run the current system over the set.
4. Score the results.
5. Compare against a baseline.
6. Change the system.
7. Rerun the evaluation.
8. Investigate regressions before shipping.

Do not judge a model or prompt from one impressive example.

Measure aggregate quality across:

- normal cases
- edge cases
- adversarial cases
- security cases
- failure cases
- representative production inputs

Run evals when you change:

- prompt
- model
- tool schema
- retrieval logic
- memory behavior
- MCP integration
- orchestration

### Tracing regressions

A prompt improvement may improve one category while silently harming another.

Compare old and new versions over the same evaluation set.

When scores drop:

- identify the failed examples
- look for a common pattern
- trace the change that caused the regression
- fix or explicitly accept the trade-off
- rerun before deployment

The strongest evidence that a prompt change is safe is its evaluation score compared with a baseline over a representative set.

---

## Cost, latency, and reliability

### Cost controls

Use:

- the smallest model that meets the quality requirement
- concise prompts and outputs
- prompt caching for repeated static context
- batching for non-urgent bulk work
- context trimming
- structured output
- bounded agent loops

### Latency controls

Use:

- faster models for time-sensitive tasks
- fewer model calls
- parallel tool or subagent execution where safe
- smaller context
- caching
- appropriate timeouts

### Reliability controls

Use:

- retries with backoff for transient failures
- timeout handling
- fallbacks
- structured errors
- graceful degradation
- idempotent operations where possible
- session and tool observability
- bounded retries and loops

Reliability is not achieved by choosing a larger model alone.

---

## Security and safety

### Prompt injection

Prompt injection occurs when untrusted content tries to override the system’s instructions or manipulate the agent into unsafe behavior.

Potential sources include:

- user text
- documents
- web pages
- repository files
- tool results
- external API responses

Defenses include:

- delimit untrusted content
- clearly separate instructions from data
- constrain the output format
- limit tool authority
- validate tool arguments
- require approval for risky actions
- keep secrets outside the model context
- never assume model instructions alone are sufficient

### Secret management

API keys and credentials belong in:

- server-side secret managers
- protected environment variables
- credential vaults
- secure deployment configuration

They do not belong in:

- prompts
- source code
- logs
- traces
- tool descriptions
- user-visible output

### Tool authority

Grant the least privilege required.

A tool that can read data should not automatically be allowed to delete data.

A tool that can propose an action should not automatically execute it.

For irreversible actions, require explicit human approval.

---

## Accelerators and intellectual property

An accelerator turns a one-off working solution into a reusable asset.

A useful accelerator packages:

- prompts
- tool definitions
- MCP configurations
- project rules
- deployment instructions
- evaluation suite
- example inputs and outputs
- security guidance
- configuration points

The goal is that another team can deploy and understand the asset without reverse-engineering the original implementation.

Include the evaluation suite so future changes can be checked against the same quality baseline.

Contributing reusable assets back to shared infrastructure compounds value across teams and engagements.

---

## High-value scenario patterns

### Scenario: unfamiliar legacy codebase

Problem:

- large undocumented monolith
- unclear module boundaries
- unknown request flow
- unknown risky areas

Best approach:

- provide real files
- ask Claude for a component and request-flow map
- ask for risks and open questions
- verify the summary against code
- make a small, reviewable change

### Scenario: production incident

Problem:

- 500 errors increased after deployment
- logs are spread across services
- a dependency is flaky

Best approach:

- provide error samples, logs, and the deployment diff
- ask for ranked hypotheses
- ask for a minimal experiment for each hypothesis
- confirm the root cause with a real run
- apply a focused fix and evaluate regression risk

### Scenario: billing test coverage

Problem:

- tests cover only the happy path
- release freeze is approaching

Best approach:

- ask for a test-case table
- identify equivalence classes and boundaries
- cover invalid input and failure paths
- review the table
- generate tests
- run the suite and inspect coverage gaps

### Scenario: vague export feature

Problem:

> Let users export their data.

Missing details may include:

- format
- volume
- permissions
- asynchronous behavior
- retention
- failure handling

Best approach:

- ask clarifying questions
- draft an implementation plan
- obtain approval
- scaffold the code
- add tests
- document behavior

### Scenario: risky refactor

Problem:

- 400-line function mixes validation, business logic, and I/O
- no tests protect behavior

Best approach:

- characterize current behavior with tests
- refactor incrementally
- run tests after each step
- review the diff

---

## Common exam traps

Avoid these weak approaches:

- choosing the biggest model by default
- using vague prompts
- requesting free-form output when code must parse it
- passing the full history and every document into every step
- allowing unbounded agent loops
- auto-executing irreversible actions
- putting credentials in prompts
- trusting generated code without running it
- trusting documentation without checking the source
- changing many variables at once while debugging
- silently swallowing tool errors
- treating one successful example as an evaluation
- confusing an agent that builds software with an agent being built

Prefer these stronger approaches:

- match the model to the constraint
- specify role, context, constraints, and output
- use structured schemas and examples
- retrieve only relevant context
- bound loops and retries
- require approval for high-impact operations
- use server-side secret management
- test and review generated work
- compare against a baseline
- change one variable and retest
- return structured errors and partial results

---

## Practice questions and answers

### 1. A real-time chat feature needs subsecond latency. Which model strategy is most appropriate?

Use a fast, economical model such as Haiku if it meets the quality requirement. Latency is the binding constraint.

### 2. What does a fixed context window imply for a long-running chat agent?

The system must manage history and leave enough context headroom for the current task and response.

### 3. An extraction pipeline returns slightly different JSON keys on each run. What is the best first fix?

Lower randomness if appropriate and constrain the response with an explicit schema and allowed field names.

### 4. When is the raw API preferable to an official SDK?

When no suitable SDK exists or when low-level protocol control is required.

### 5. What should happen when downstream code parses Claude output?

Use an explicit output schema with no extra text and validate the response before processing it.

### 6. Why delimit user-provided content?

To clearly separate untrusted data from system instructions and reduce ambiguity around instruction authority.

### 7. A simple lookup causes an agent to loop fourteen times. What is the likely issue?

A broken tool, invalid result, unclear completion condition, or missing stop condition.

### 8. Who executes a tool function in the standard tool-use loop?

The application or runtime intercepts the model’s tool request and executes the real function.

### 9. Where should an MCP password be stored?

In server-side environment configuration or a secret manager. It should not enter the model context.

### 10. An agent is about to delete a production record. What is essential?

Explicit human approval and a permission boundary around the irreversible action.

### 11. How should a large untested function be refactored safely?

Generate characterization tests first, then refactor in small steps while keeping the tests green.

### 12. What is the strongest evidence that a prompt change is safe to ship?

The new version performs acceptably against a representative evaluation set compared with the old baseline.

### 13. What is the most effective cost lever for repeated static context?

Prompt caching.

### 14. What is the primary defense against prompt injection in user content?

Treat the content as untrusted, delimit it, constrain the agent’s authority, validate tool actions, and require approval for risky operations.

### 15. What is an accelerator?

A reusable, documented, configurable package that includes the implementation assets and evaluation material needed for safe reuse.

---

## Final readiness checklist

Before the exam, make sure you can explain:

### Foundations

- tokens and token costs
- input versus output tokens
- context windows
- memory versus context
- sampling and nondeterminism
- model quality, cost, and latency trade-offs
- temperature or other supported randomness controls

### Application development

- official SDK versus raw API
- prompt structure
- few-shot examples
- output schemas
- structured tool results
- prompt diagnosis
- code generation with project constraints
- debugging with evidence
- test-case design
- characterization tests
- safe refactoring
- code review and documentation verification

### Agents and workflows

- single call versus agent loop
- reason, act, observe, repeat
- tool-use sequence
- loop boundaries
- retries and timeouts
- decomposition and subagents
- human approval for irreversible actions
- graceful degradation

### MCP and integrations

- what MCP provides
- MCP server boundaries
- tool and data exposure
- credential isolation
- server-side authentication
- least privilege
- context and memory scope

### Claude Code and reusable workflows

- permission model
- project rules
- user and project context
- skills
- MCP configuration
- repeatable developer workflows

### Production engineering

- evaluation sets
- success metrics
- baselines
- regression tracing
- cost controls
- prompt caching
- latency controls
- reliability patterns
- observability

### Security

- prompt injection
- untrusted tool output
- secret management
- permission scoping
- approval gates
- irreversible actions

### Exam strategy

- read the entire scenario before choosing an answer
- identify the constraint first
- prefer the option with verification and least privilege
- eliminate answers that are unbounded, silent, vague, or overpowered
- for multi-select questions, evaluate each option independently
- choose the smallest model and smallest authority that satisfy the requirement
- look for the answer that preserves human oversight where risk is high

---

## The developer mindset

The strongest developer answer is usually not the most impressive-looking one.

It is the answer that:

- uses the right amount of context
- chooses the right model
- constrains the output
- verifies behavior
- protects credentials
- limits authority
- measures quality
- handles failure explicitly
- keeps humans involved when consequences are high

Claude can accelerate every stage of development, but engineering judgment remains your responsibility.

The exam is ultimately testing whether you can turn a capable model into a reliable software system.
