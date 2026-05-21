---
title: Agents and concurrent execution in Claude Code
category: software
date: 2026-05-21
tags: [agents, concurrency, claude-code, ai-tooling]
---

# Agents and concurrent execution in Claude Code

## TL;DR

An agent (subagent) is a separate Claude instance with its own isolated context window. Concurrency isn't a setting — it's just multiple agent tool calls issued in a single response turn. Independent tasks run in parallel; tasks with dependencies must run sequentially.

## The explanation

### What a subagent is

When Claude Code delegates work, it spawns a **subagent**: a separate Claude instance with its own context window, its own tools, and a task to complete. The subagent doesn't see your conversation history, doesn't know what other agents are doing, and can't share context with the main agent or with each other. Everything it needs must be in the task description given to it.

Subagents are defined as Markdown files with YAML frontmatter in `.claude/agents/` (project-level) or `~/.claude/agents/` (personal). They specify:

- A **name** and **description** — Claude uses the description to pick the right specialist when delegating
- A **system prompt** — the body of the file; instructions the subagent always starts with
- Optional **tool restrictions** — e.g. read-only, no shell access
- An optional **model** — route cheap tasks to a smaller model to save cost

Claude Code ships with three built-in subagents:

| Name | Model | Tools | Use for |
|---|---|---|---|
| `Explore` | Haiku (fast) | Read-only | Searching/locating code |
| `Plan` | — | Read-only | Architecture and planning |
| `General-purpose` | — | Full access | Complex multi-step tasks |

### Subagents vs. skills

These are different things and are easy to confuse.

- **Skills** are invoked explicitly — either by typing `/skill-name` or by Claude calling the `Skill` tool when a trigger condition is met. Some skills have auto-trigger rules (e.g. "invoke `tm:coding-standards` any time the user asks me to write code"). They do not run unless called.
- **Subagents** are workers that Claude dispatches to do a task. Claude picks the subagent based on the description field in the subagent definition. They're about *doing*, not *instructing*.

A skill can orchestrate subagents — the skill defines the workflow, the subagents do the parallel work.

### How concurrency actually works

Concurrency is not a special mode. It's a natural consequence of how Claude makes tool calls: **multiple tool calls in a single response turn run in parallel when they're independent.**

The trigger is your prompt. Compare:

```
# Forces sequential — "then" implies ordering
Research the auth module, then research the payments module, then the DB module.
```

```
# Allows concurrent — no implied ordering
Research the auth, payments, and DB modules using separate subagents.
```

The second version lets Claude dispatch all three simultaneously because none depends on another's result.

### The dependency rule

Draw the dependency graph. If Task B needs Task A's output, they must run sequentially. If there's no arrow between two tasks, they can run in parallel.

```
# Must be sequential — fix depends on knowing what failed
Run the test suite, then fix the failing tests.

# Can be parallel — independent investigations
Review this PR for security issues, performance bottlenecks, and test coverage.
```

Also watch for **file conflicts**: two agents writing to the same file means the last writer wins with no merge. If you're parallelising implementation work, partition which files each agent owns.

### Foreground vs. background

**Foreground (default):**
- The main conversation pauses and waits for the subagent to finish
- Permission prompts from the subagent surface to you interactively
- Use when you need the result before anything else can proceed

**Background:**
- The subagent runs while the main conversation continues
- Permission prompts are auto-denied — there's no one to ask
- Use when the work is independent and non-blocking
- You can set `background: true` in the subagent's frontmatter to always run it this way

If a background agent keeps silently failing, re-run it as foreground to see what permissions it's asking for.

### Writing good delegation prompts

Because subagents are context-isolated, **the task description is the only channel of communication into the subagent.** It has no access to your conversation history, the files you've already read, or what other agents are doing.

Bad delegation:
```
Use the codebase-explorer to look into what we were discussing.
```

Good delegation:
```
Use the codebase-explorer to research how the checkout service handles payment 
retries. Look specifically in src/checkout/payments/ and report the retry logic, 
timeout values, and error handling patterns.
```

If you need specifics in the report, name them explicitly — otherwise you'll get a distilled summary that may omit the detail you needed.

### Key gotchas

**Results are summaries** — the agent's full reasoning and all the files it read don't come back to the main conversation. You get a distilled report. Name what you need in the output or you won't get it.

**No nesting** — subagents cannot spawn their own subagents. The main agent orchestrates everything flat.

**Trust but verify** — subagents report text; they can miss things or be wrong. For consequential work, verify their output rather than acting on the summary alone.

**Stale references** — if an agent was told about a file path or function that has since moved or been renamed, it won't know. Verify named resources still exist before acting on them.

### Agent teams (experimental)

Agent teams (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) add peer-to-peer communication between agents:

- A **lead** session coordinates work
- **Teammates** work independently and can message each other directly
- There's a shared task list teammates claim from

The key difference from subagents: teammates can challenge each other's findings, debate hypotheses, and coordinate without routing everything through the lead. Much more powerful for work that benefits from multiple perspectives actively comparing notes — but significantly more expensive since each teammate has its own full context window.

For most use cases, start with subagents. Only reach for agent teams when agents genuinely need to talk to each other.

### Checklist before parallelising

1. Are the tasks independent? → parallelize
2. Do they write the same files? → if yes, sequence them or partition file ownership
3. Does each task description include everything the agent needs? → no assumed shared context
4. Do I need the result before proceeding? → foreground if yes, background if no
5. What should appear in the report? → be explicit or you'll get a vague summary

## When does this matter?

- **Parallelising research** — investigating multiple modules, codebases, or questions simultaneously instead of waiting for each to finish before starting the next.
- **Parallel code review** — dispatching separate agents for security, performance, and test coverage in a single pass.
- **Background work** — running a slow task (e.g. a full test suite scan) while continuing work on something else.
- **Debugging slow or failed agent runs** — understanding whether the bottleneck is a dependency (must be sequential) or a permissions issue (switch from background to foreground).
- **Understanding why a skill didn't run** — skills require explicit invocation; subagents are selected automatically by Claude based on the description field.

## See also

- [Claude Code subagent documentation](https://docs.anthropic.com/en/docs/claude-code/sub-agents)
- [Pub/sub](software/pub-sub.md) — the same independence vs. dependency thinking applies to event-driven systems
