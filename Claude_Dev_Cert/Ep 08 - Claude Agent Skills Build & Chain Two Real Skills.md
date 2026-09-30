# Ep 08 | Claude Agent Skills: Build & Chain Two Real Skills

## Overview

This episode introduces a very important architectural idea: when a project grows, the agent’s permanent instructions start to bloat.

The lecturer points out a real pain that shows up in practical agent work:

- your CLAUDE.md keeps growing
- you keep appending “one more useful thing” to it
- the file becomes long and expensive to load every time
- the agent pays for the extra context even when that capability is not needed

This is the core problem that skills solve.

A skill is not a random extra file. It is a way to give the agent a capability without paying for that capability on every single run.

Instead of stuffing every occasional capability into one giant instructions file, you move it into a skill that loads only when needed.

---

## Why this matters in real projects

The speaker gives a concrete example: a PR review agent starts with a few essential rules, then gradually gains more features.

Examples of things people might add:

- a plain-English digest of a review
- a change log line from the review
- a summary of issues
- a release-note format
- a custom explanation mode

These features are useful, but not needed in every run.

If you add them all to the same CLAUDE.md file, then every agent session pays for them simply by reading the file.

That is the waste.

The core idea is:

- rules that are always true should live in CLAUDE.md
- occasional, contextual capabilities should live in a skill

This keeps the project fast and the instructions precise.

---

## The cost of context bloat

The lecturer makes a strong point: CLAUDE.md is always loaded.

That means:

- every session reads it
- every run pays for it in tokens
- the cost scales with the size of the file
- the agent also gets more context than it needs

The result is a lot of token overhead for features that are not always relevant.

This is why the course introduces skills as an alternative.

Skills allow on-demand capability loading instead of always-on context.

---

## What a skill actually is

The speaker is clear that the real answer is mechanical, not marketing-driven.

A skill is not a mysterious magical feature.

A skill is a folder on disk containing a skill.md file.

The file itself has two major parts:

1. YAML front matter
2. Markdown instructions

The front matter contains lightweight metadata like the skill’s name and description.

The markdown body contains the actual instructions that get loaded when the skill is chosen.

This is the essential structure to remember.

---

## Progressive disclosure

One of the most important concepts in the episode is progressive disclosure.

The speaker explains that Claude does not load every skill’s body into context at once.

Instead, it first sees a lightweight summary:

- skill name
- description

Then, when the situation matches, the full skill body is loaded.

This is what makes skills efficient.

The skill is like a business card first and a full manual only when needed.

This is the concept to remember from the exam and from the design perspective.

---

## Skill format and anatomy

The skill file uses a standard structure.

At the top, there is front matter with fields such as:

- name
- description
- maybe argument hints
- maybe allowed tools
- maybe context behavior options

Under that, the markdown instructions describe what the skill does and when to use it.

This is the exact file format the agent or CLI can discover and load.

The lecture underscores that this is portable and aligned with the Anthropic skill standard.

The same pattern is valuable both inside Claude Code and in the agent SDK workflow.

---

## Skills vs CLAUDE.md

This is one of the clearest conceptual distinctions in the episode.

CLAUDE.md is for universal truths.

It should contain only the instructions that are always relevant to the project.

Examples:

- repo rules
- review standards
- project conventions
- safety constraints
- core operational policies

A skill is for situational capability.

Examples:

- PR digest generation
- release note drafting
- exploratory reduction of a complex issue
- task-specific instructions that should not always be in context

The mental model is:

- CLAUDE.md = permanent rules of the house
- skill = optional manual for a specific task

---

## Skills vs subagents

The speaker also clarifies how skills differ from subagents.

A subagent is a separate agent instance created on demand.

It usually starts from a fresh context and needs explicit task handoff.

A skill is not a new agent.

It is a contextual instruction bundle that gets loaded into the current conversation when relevant.

The key differences are:

- CLAUDE.md: always loaded
- skill: loaded on demand
- subagent: separate agent with isolated context and delegation pattern

This is why they solve different problems.

---

## Skills vs tools

The lecturer also explains the difference between a skill and a tool.

A tool is an action.

For example:

- submit a finding
- publish a review
- inspect files
- run a command
- fetch metadata

A tool executes something and returns a result.

A skill is not execution. It is context improvement.

It changes the instructions and knowledge the model has before deciding what to do next.

This is the key separation:

- tools do work
- skills inform behavior

---

## The two real skills in this episode

The course builds two concrete skills for the PR review agent.

### 1. PR digest skill

This skill turns a completed review into a short, plain-English digest.

It is useful when someone asks:

- what is the outcome of this PR review?
- is this safe to merge?
- what happened in this review in simple terms?

The skill does not re-run the review. It summarizes the already-existing findings and final review output.

### 2. Change log entry skill

This skill takes the digest and turns it into a release-note style entry.

This is useful when someone wants a compact changelog line or a brief summary for release notes.

The pattern is chained:

- review findings are generated
- digest skill summarizes the review
- changelog skill turns that summary into a one-line release note

This is a perfect example of chaining skills in the same conversation.

---

## Why the PR digest skill is a good skill

The lecture emphasizes that the digest skill is a perfect fit for skill-based design because it is a contextual capability, not a universal rule.

Not every review needs a human-readable digest.

Only when someone asks for it does the agent need this capability.

That means it should not sit in CLAUDE.md for every run.

It should sit in a skill and only load when the request matches the skill description.

This is the exact cost-saving pattern the episode is explaining.

---

## Why the changelog skill is also a good skill

The changelog skill is even more obviously contextual.

It should not be loaded in every review session.

It only matters when someone asks for a release-note style summary.

That makes it ideal for the same skill pattern.

The chain is natural because it reuses the digest already in the session without requiring a new review pass.

This is a powerful example of skill chaining.

---

## Why context fork is not used here

This is a very important part of the lesson.

The speaker explains that these skills should not use context-forking behavior.

Why?

Because a skill like PR digest depends on the review already being in the conversation.

If you run the skill in an isolated forked context, it loses the information it needs.

This is the same lesson from subagent orchestration:

- isolated context means no inherited conversation
- if the skill needs the current review state, it must share the main conversation context

This is why they are designed to operate in the active conversation rather than in a clean-slate subagent context.

The lecture explicitly says:

- context fork is for tasks that should explore independently
- it is the wrong choice when a skill depends on prior results in the same conversation

---

## The skill metadata and file system layout

The project structure for this episode is straightforward.

The course introduces a hidden folder like:

- .claude/
- skills/
- skill-name/
- skill.md

The skill folder is discovered by the runtime and the skill’s metadata is used to decide when to load it.

The main thing to remember is that the skill includes:

- name
- description
- body instructions

In a real agent workflow, this is how you keep optional capabilities modular and low-cost.

---

## `settingsSources` and project-scoped loading

The lecture also reinforces the importance of project-scoped settings.

The agent should be told to read project-level configs, not just a global user config.

This matters because skills are part of the project’s local behavior.

If the project settings are not loaded correctly, the agent may ignore the project’s skill config or CLAUDE.md file entirely.

The speaker mentions a critical detail:

- if you leave settings empty, you may get no project config
- if you set project scope, the local project files are used

This ensures the agent discovers the project’s skills and project rules.

---

## Permission mode and headless execution

The agent config also includes a permission mode like “do not ask.”

This is important in headless or automation contexts.

If the system has no human to approve a prompt, then refusing to ask is preferable to hanging.

The lecture says that this is “headless hygiene.”

The goal is to avoid passive waiting for permission in an environment where there is no interactive user.

This is operationally important for real agent systems, especially in CI or scripted workflows.

---

## The gotcha about `allowed_tools`

The episode also mentions an important gotcha:

- the `allowed_tools` field in a skill definition is not the main enforcement mechanism in the Agent SDK workflow
- it is more relevant in the Claude Code CLI flow
- in the Python SDK workflow, global tool permissions still control the actual access

This is a very practical distinction.

The lecture warns not to confuse skill metadata with real access control.

The agent’s runtime tool permissions still matter most.

The metadata field is still useful for portability, but it is not the only or sometimes not the effective mechanism in this project setup.

---

## A common anti-pattern to avoid

The speaker warns against putting all optional behavior into CLAUDE.md.

That is the core anti-pattern the skill solves.

The temptation is to “just add one more paragraph” because it feels easy.

But this creates long-lived cost and noise.

The better design is:

- keep CLAUDE.md small and always relevant
- move task-specific capabilities into skills

This is a foundational abstraction in practical agent design.

---

## The chaining pattern in one sentence

The speaker’s practical demo makes this pattern very clear:

- submit findings
- publish the review
- ask for a digest skill
- ask for a changelog skill based on the digest

No manual piping of files is needed. The same conversation carries the intermediate output forward, and the skill system loads the right instruction bundle when needed.

That is exactly how skills become an elegant addition to the agent workflow rather than a noisy bloating layer.

---

## Final key takeaways for the exam

The episode leaves a few very memorable concepts:

- CLAUDE.md is for permanent project truth
- skills are for optional, on-demand capabilities
- skills use progressive disclosure
- the skill file is a folder with a skill.md file
- the skill name and description are what trigger usage
- skills are not subagents
- skills are not tools
- context fork is not the right choice when the skill depends on current conversation state
- tool permissions are still enforced at the runtime layer, not just by the skill metadata

This is a great example of context engineering in a practical system.

---

## Closing thought

The lesson closes by emphasizing that skills are a scaling tool for agent systems.

Once a project gets beyond a few rules and patterns, the system becomes much more maintainable if some capabilities are moved out of the always-loaded instructions and into on-demand skills.

This is not just a neat trick. It is one of the core ways to keep agent systems usable as they grow.
