# Ep 07 | Structured Output Handling

## Overview

This episode focuses on one of the most important ideas in agentic AI: making output predictable, enforceable, and safe.

The lecturer frames the problem in a direct way:

- prompts are requests, not contracts
- they are probabilistic by nature
- even a very good model can deviate from a rule or write a slightly different label than expected
- if you want deterministic behavior, you need structure

The whole lesson is really about moving from “Claude should do this” to “Claude must produce this exact shape.”

This matters because a lot of agent failures are not reasoning failures. They are shape failures.

A model may reason correctly but still output:

- a wrong severity label
- a missing field
- a fabricated fix
- a freeform paragraph instead of structured JSON

This episode is about fixing that by introducing schema-driven structured output and validation.

---

## Why structured output matters

The speaker starts with a concrete problem: the project’s review rules say severity must be one of a few valid values.

For example, the rules may say:

- blocking
- warning
- nit

But the model may return something like:

- critical
- issue
- major
- high

These are semantically close, but they are not the rule that the agent contract expects.

This is the key lesson:

- a markdown rule file is useful
- but it is still just guidance
- guidance is not a strict contract

That is why the episode introduces rigid structure instead of vague natural language alone.

A schema acts like a contract.

---

## The house-rules analogy

The lecturer uses an analogy to explain the difference between instructions and enforcement.

A CLAUDE.md file is like house rules:

- it tells the agent what to do
- it gives it constraints
- it improves behavior
- but it is still not a guarantee

The model can ignore or drift from it.

This is exactly why the episode moves to a stricter layer: output validation and schema enforcement.

The goal is no longer merely “think better” but “produce only valid objects.”

---

## Structured output as a contract

The speaker explains that the real idea is not “make the model write JSON nicely.”

The real idea is:

- define the schema for a review finding
- force the output to match that schema
- validate it before storing or publishing it
- reject invalid values immediately

This is a contract between the agent and the rest of the system.

The output is no longer just text. It is structured data.

That changes everything:

- validation becomes possible
- severity values become constrained
- missing fields become obvious
- the downstream system can trust the result

---

## The shape of a review finding

The lesson introduces a concrete schema for a single review finding.

A finding is expected to have fields like:

- file_path
- line
- severity
- message
- suggested_fix

The schema requires a real structure instead of a loose paragraph.

This is important because downstream logic wants to know exactly where the issue is and how serious it is.

The review system can then produce a final summary or publish a review comment from structured findings instead of free text passed around from agent to agent.

---

## Why “required but nullable” is a powerful pattern

One of the key design ideas in the episode is this:

- required fields should still be allowed to be null in a valid state
- the field is required, but the value may legitimately be “no fix known”

This is not a loophole.

It is a way to avoid hallucination.

For example, if the model is not confident about a suggested fix, it should return:

- suggested_fix: null

rather than inventing a fix that is not grounded in the code.

This is much better than making the field optional and hoping the model simply omits it.

The pattern is:

- required fields must exist
- but they can carry a valid null signal when needed

This keeps the contract honest while preventing fabricated answers.

---

## The schema-enforced severity enum

The teacher uses the review severity as a primary example.

Only specific values are accepted:

- blocking
- warning
- nit

Anything else is rejected.

This makes the output much more predictable and protects the rest of the agent’s logic from bad values.

The key point is that the system can no longer accept synonyms like:

- critical
- urgent
- issue
- major

unless they are explicitly allowed in the schema.

The model should not be allowed to invent a brand-new severity label.

---

## Validation vs generation

The episode makes a very important distinction:

- schemas help generate valid content
- validation catches invalid output
- neither of those is the same as asserting the claim is true

Schemas are about shape.

They do not guarantee that a finding is factually correct.

For example, a tool may validate that a finding contains:

- file path
- line number
- severity
- message

But it cannot confirm that the issue is genuinely real or that the source of truth is correct.

This is a crucial mental model.

Truth is separate from format.

---

## The role of validation functions

The instructor introduces a Python validation function that checks each finding before it is accepted.

This function verifies things like:

- file path is a non-empty string
- line is valid
- severity is in the allowed enum
- message is present
- required fields are not missing
- suggested_fix is either a string or null

If any check fails, the tool returns a structured error.

This is a deterministic guardrail.

It is not a vague prompt suggestion. It is actual enforcement.

---

## Returning an error from a tool

The lecture emphasizes that a custom tool can fail with an error response instead of “pretending” it succeeded.

When a finding is invalid:

- the tool returns an error result
- the SDK sees it as a failed tool call
- the model is forced to retry or repair the output

This is extremely useful because it turns validation into an active part of the loop.

It does not require the agent to “remember the rule.” It just reacts to tool failure.

In other words, the contract is enforced by the system rather than trust alone.

---

## Strict tool use in the raw Messages API

The speaker also explains the raw Messages API feature called strict tool use.

In the raw API, you can specify a tool schema and enable strict mode.

When strict mode is on, the API constrains generation so that the tool call matches the schema as closely as possible.

This is a powerful mechanism because it reduces malformed tool calls at the token level.

The speaker explains that this is a form of grammar-constrained generation.

The model is effectively prevented from generating tokens that violate the tool schema.

This is more reliable than trying to “prompt the model to be careful.”

---

## The big difference between raw API and Agent SDK

The lecturer points out a subtle but important distinction.

In the raw Messages API, you can often set strict mode directly on the tool definition.

In the Agent SDK, the interface may not expose the exact same convenience in the same way.

So the practical pattern is:

- define a strong custom JSON schema
- validate input in Python
- return structured error results for invalid output

This makes the Agent SDK pattern more explicit, even if it is not as magical as strict mode in the low-level API.

The point is the same: constrain the model by structure.

---

## One finding at a time

The speaker emphasizes the tool design pattern:

- submit exactly one structured review finding per tool call
- do not batch multiple findings in a single call
- call once per distinct issue

This matters because validation is much easier and more reliable when the unit is one result.

If a tool accepts a giant freeform blob with multiple issues at once, it becomes harder to enforce a schema and harder to validate the data correctly.

The pattern is intentionally narrow and precise.

---

## Publish review as a separate step

The episode introduces a second tool: publish review.

This tool is deliberately designed to be extremely narrow.

It takes no freeform text and no big review body.

Instead, it builds the final review comment only from the findings previously submitted through the structured tool.

That creates a clean pipeline:

1. subagent finds a bug
2. submit finding validates it
3. findings accumulate in a structured list
4. publish review generates the final review from the structured findings

This minimizes freeform drift.

The model is no longer writing a long natural-language review from scratch in one big step.

It is rendering from verified structured data.

---

## Structural design matters for subagents too

The lecture also reveals an important architectural improvement.

Earlier versions of the agent system had subagents return plain freeform text to the coordinator.

The coordinator then had to interpret it and translate it into real findings.

This introduces a lot of structure loss.

The new pattern is better:

- the subagent calls submit finding directly
- the finding is validated immediately
- the coordinator just aggregates the validated findings

This is more robust because the data is structured before it leaves the subagent.

The earlier freeform handoff was more brittle.

---

## Validation catches shape bugs, not truth bugs

The speaker makes this distinction very clear.

Schema validation and tool validation help with:

- wrong field names
- missing required fields
- bad enum values
- wrong types
- fabricated fixes that should have been null

They do not determine whether the issue is actually real.

A model may still produce a plausible but wrong finding.

That requires:

- repo inspection
- deeper reasoning
- human review
- stronger system design

In other words, structure gives you reliability at the interface boundary, not perfect fact-checking.

---

## Why this is important for agentic systems

The episode wraps up by showing why this pattern matters in the real world.

Without structure, agents often “look smart” but are fragile.

They can:

- mislabel severity
- invent fixes
- forget required fields
- return inconsistent output
- create review comments from raw drift-prone text

With structure:

- the system becomes more deterministic
- failures become easier to debug
- validation becomes part of the loop
- downstream logic is much safer

This is exactly the kind of engineering pattern interviewers and certification content emphasize.

---

## A critical design principle

The speaker leaves the audience with a memorable principle:

- prompts are not contracts
- schemas are contracts
- validation is enforcement

If you want an AI system to behave reliably, do not rely only on wording.

Build the system so that the invalid output cannot pass through.

That is the heart of structured output handling.

---

## Final takeaway

This lesson is about turning untrusted freeform generation into a controlled data pipeline.

The core ideas are:

- define a schema
- validate every finding
- reject invalid output early
- allow null when the fix is genuinely unknown
- require structured tool calls instead of freeform summaries
- separate shape validation from truth validation

This is a foundational concept for reliable agent systems and a very important part of the Claude Certified Developer track.
