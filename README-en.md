# Claude Code: Under the Hood — A Complete Step-by-Step Guide

<p align="center">
  <img src="https://img.shields.io/badge/lang-English-blue" alt="lang">
  <img src="https://img.shields.io/badge/level-beginner%20to%20advanced-green" alt="level">
  <img src="https://img.shields.io/badge/topic-agent%20architecture-orange" alt="topic">
  <img src="https://img.shields.io/badge/license-educational%20only-lightgrey" alt="license">
</p>

> **A deep dive into Claude Code's architecture «under the hood»** — how the agent loop works, tools, permissions, sub-agents, memory, MCP, and production-grade infrastructure. The guide follows a «simple to complex» principle: starting with a 15-line loop, then building up to a full production agent. Material is collected from publicly available sources and community discussions, intended solely for study and technical research.

> [!NOTE]
> This repository is **educational**. It explains the *patterns* of modern coding agents using Claude Code as an example. Internal implementation details (codenames, flags, exact file structure) may vary from version to version — what matters is not the specifics, but the principles that apply to any agent.

> [!TIP]
> **Popular resources on Machine Learning, AI, and data analysis.**
>
> 🧠 [**Machine Learning**](https://t.me/+-DTW9e5kyZ5jNGFi) — an original Telegram channel containing the entire foundation for working with AI models. Digests of the best projects, code breakdowns, LLM launch instructions, interview prep, and much more.
>
> 📚 [**Data Science**](https://t.me/+64ytM2_MgJozZTEy) — rare literature, articles, courses, and unique guides for ML specialists at any level. Read, grow, practice.
>
> 💼 [**Machine Interview**](https://t.me/+Gncl6NEJXOo4ODIy) — a database of 1900 machine learning interview questions. You'll easily land an offer by studying these popular questions.
>
> Telegram is better — [subscribe](https://t.me/+-DTW9e5kyZ5jNGFi).

---

## How to Read This Guide

This guide is designed for three types of readers — choose your route to avoid reading unnecessary material.

| You are… | Start with | Then | You can skip |
|---|---|---|---|
| 🟢 **Beginner** — just want to get started | sections 1–4 | 5, 16, 26 | 18–25 (for now) |
| 🟡 **Practitioner** — already using it | 10–12, 27–29 | 16, 19–22 | 6–7 (overview) |
| 🔴 **Architect** — building your own agent | 5–9 | 13, 18, 23–25 | nothing 🙂 |

You can also read in order — the material progresses «from simple to complex»: first a 15-line loop (section 5), then a full production wrapper (section 18). If you're in a hurry — keep section **34 «Cheat Sheet»** open as a reference.

---

## Table of Contents

**Part I. Fundamentals**
1. [Who Needs This and Why](#1-who-needs-this-and-why)
2. [What Is an Agent in Simple Terms](#2-what-is-an-agent-in-simple-terms)
3. [Getting Started: Installation in 5 Minutes](#3-getting-started-installation-in-5-minutes)
4. [First Session: 10 Commands Worth Knowing](#4-first-session-10-commands-worth-knowing)

**Part II. How It Works Inside**
5. [The Basic Agent Loop (Heart of Everything)](#5-the-basic-agent-loop-heart-of-everything)
6. [Architecture Overview](#6-architecture-overview)
7. [Lifecycle of a Single Request](#7-lifecycle-of-a-single-request)
8. [Tool System](#8-tool-system)
9. [Permissions System](#9-permissions-system)
10. [Slash Commands](#10-slash-commands)
11. [Hooks and settings.json](#11-hooks-and-settingsjson)
12. [MCP: Connecting External Tools](#12-mcp-connecting-external-tools)

**Part III. Advanced**
13. [Sub-Agents and Multi-Agent Systems](#13-sub-agents-and-multi-agent-systems)
14. [Context Management (Compact)](#14-context-management-compact)
15. [Memory and On-Demand Knowledge](#15-memory-and-on-demand-knowledge)
16. [Plan Mode](#16-plan-mode)
17. [Saving and Restoring Sessions](#17-saving-and-restoring-sessions)
18. [12 Production-Grade Mechanisms](#18-12-production-grade-mechanisms)
19. [Observability and Agent Tracing](#19-observability-and-agent-tracing)
20. [Cost and Token Optimization](#20-cost-and-token-optimization)
21. [Prompt Caching](#21-prompt-caching)
22. [Testing and Evaluating Agents (Evals)](#22-testing-and-evaluating-agents-evals)
23. [Security and Prompt Injection Defense](#23-security-and-prompt-injection-defense)
24. [Fault Tolerance: Retries, Timeouts, Idempotency](#24-fault-tolerance-retries-timeouts-idempotency)
25. [Streaming and Parallelism Under Load](#25-streaming-and-parallelism-under-load)

**Part IV. Practice**
26. [Workshop: Build a Mini-Agent Yourself](#26-workshop-build-a-mini-agent-yourself)
27. [Recipes: Real-World Use Cases](#27-recipes-real-world-use-cases)
28. [Practical Examples: Step-by-Step Breakdown](#28-practical-examples-step-by-step-breakdown)
29. [Best Practices and Anti-Patterns](#29-best-practices-and-anti-patterns)
30. [Troubleshooting](#30-troubleshooting)
31. [Glossary](#31-glossary)
32. [Frequently Asked Questions (FAQ)](#32-frequently-asked-questions-faq)
33. [Where to Go Next](#33-where-to-go-next)
34. [Cheat Sheet (Quick Reference)](#34-cheat-sheet-quick-reference)

---

## 1. Who Needs This and Why

This guide is for those who want to understand **not how to use Claude Code, but how it works inside**. If you are a developer and want to build your own coding agent, understand production agent patterns, or simply grasp what happens between your request and the model's response — you're in the right place.

The guide is useful for three types of readers. For the **beginner**, it provides a step-by-step entry: installation, first commands, mental model. For the **practitioner** — recipes, best practices, and breakdowns of common mistakes. For the **engineer-architect** — a detailed analysis of how a flat loop becomes a production system with permissions, sub-agents, and isolation.

We go from the simplest (a 15-line loop) to the complex (multi-agent commands, git worktree isolation), explaining each layer separately. Read sequentially or jump around using the table of contents.

---

## 2. What Is an Agent in Simple Terms

A regular chat with a model is «question → answer». The model can't *do* anything: it only generates text. An **agent** differs in one thing: it has **tools** and a **loop**.

Imagine an assistant that you've given not only the ability to speak but also hands: it can read files, run commands, search the internet. After each action, it looks at the result and decides what to do next — until the task is done. That's it. Agent = model + tools + a loop that runs while there's work to do.

```text
CHAT:                          AGENT:
question -> answer             question -> [think -> act -> result]* -> answer
(one step)                     (many steps, until the task is solved)
```

Everything else in this guide is about how to make this simple loop **reliable, secure, and scalable**.

### Agent vs Chat vs Autocomplete — When to Use What

Three tools solve different problems. Confusing them is a common source of frustration («the agent is too slow», «autocomplete is too dumb»).

| | Autocomplete (Copilot) | Chat with Model | Agent (Claude Code) |
|---|---|---|---|
| **What it does** | completes a line/block | answers with text | acts in a loop until result |
| **Sees the project** | current file + a little | what you paste | reads files on its own |
| **Runs code** | no | no | yes (tests, builds, git) |
| **Iterates** | no | you do it manually | itself: edit → test → edit |
| **Best use** | fast code typing | question-explanation | multi-step tasks |
| **Cost/speed** | instant, cheap | fast | slower, more expensive |

Practical rule: **autocomplete** — when you're writing code yourself and know what to do; **chat** — when you need to understand or ask something; **agent** — when a task requires multiple steps with feedback («fix this test», «refactor this module», «debug this bug»). Don't use an agent for a one-liner — it's like calling a tow truck to move your car a meter.

---

## 3. Getting Started: Installation in 5 Minutes

**Step 1. Check your environment.** You need Node.js version 18 or higher:

```bash
node --version   # expected v18.x or higher
```

If Node isn't installed — download from [nodejs.org](https://nodejs.org) or install via a version manager (nvm, fnm).

**Step 2. Install Claude Code globally:**

```bash
npm install -g @anthropic-ai/claude-code
```

**Step 3. Navigate to your project folder and launch:**

```bash
cd your-project
claude
```

**Step 4. Authenticate.** On first launch, a browser will open for you to log into your Anthropic account. The token is saved locally — no need to re-login.

**Step 5. Ask your first question,** for example: «explain the structure of this project». Done — you're inside the agent loop.

> [!TIP]
> Run `claude` from the project root — this way the agent immediately sees the context (files, git, `CLAUDE.md`). Launching from an empty folder is less useful.

### Quick Check That Everything Works

| Command | What it does |
|---|---|
| `claude --version` | shows version |
| `claude "hello"` | one-shot query without entering REPL |
| `claude` | interactive mode (REPL) |
| `claude --help` | list of all flags |

---

## 4. First Session: 10 Commands Worth Knowing

Inside interactive mode (REPL), slash commands are useful. Here's the minimal set for beginners:

| Command | Purpose |
|---|---|
| `/help` | list all available commands |
| `/clear` | clear context and start fresh |
| `/init` | generate `CLAUDE.md` for the current project |
| `/model` | view/switch model |
| `/memory` | open/edit memory files |
| `/compact` | manually compress context while preserving essence |
| `/review` | request a review of changes |
| `/resume` | return to a previous session |
| `/cost` | show tokens spent and cost |
| `/exit` | exit the session |

> [!TIP]
> Start any new project with `/init` — Claude will scan the repository and create a `CLAUDE.md` describing the stack, build commands, and structure. This dramatically improves the quality of subsequent responses.

---

## 5. The Basic Agent Loop (Heart of Everything)

At its core, Claude Code is built on a very simple idea. Everything else is built on top of it.

```text
MAIN LOOP
=========

User --> messages[] --> Claude API --> response
                                    |
                        stop_reason == "tool_use"?
                             /          \
                           yes           no
                            |             |
                    execute tool        return text
                    add tool_result
                    loop back -----------> messages[]
```

Let's break down what happens step by step:

1. The **user** sends a request — it goes into the `messages[]` array (the entire conversation history).
2. The array is sent to the **Claude API** along with the list of available tools.
3. The model responds. A key field in the response — `stop_reason`.
4. If `stop_reason == "tool_use"` — the model wants to **invoke a tool**. We execute it, place the result back into `messages[]` as `tool_result` and **return to step 2**.
5. If not — it's the final text response, the loop is done.

That's the whole «magical» agent. Claude Code wraps this loop in production-grade infrastructure: permissions, streaming, parallelism, context compression, sub-agents, persistence, and MCP. Throughout the guide, we'll break down each layer of this infrastructure.

> [!IMPORTANT]
> Remember the main idea: **the loop never changes**, no matter how many capabilities we add. You can have 3 tools or 300 — the structure remains the same. This is what makes the architecture extensible.

---

## 6. Architecture Overview

```text
ENTRY LAYER
  cli --> main --> REPL (interactive mode)
              --> QueryEngine (headless / SDK)
       |
       v
QUERY ENGINE (QueryEngine)
  submitMessage(prompt) --> message stream
       ├── assemble system prompt (tools + CLAUDE.md)
       ├── process /commands
       ├── main agent loop
       │     ├── parallel tool execution
       │     ├── auto context compression
       │     └── tool orchestration
       └── stream result to consumer
       |
       ├──────────────┬──────────────┐
       v              v              v
  TOOLS           SERVICES        STATE
  40+ tools       API client      permissions, history,
  Bash/Read/Edit  compact/mcp     agents, modes
  Glob/Grep       telemetry
  WebFetch/Agent  plugins
```

The architecture is best read top-to-bottom as a «layered cake»:

- **Entry layer** decides what mode we're in: interactive REPL (human at a terminal) or headless/SDK (agent inside a script or CI).
- **QueryEngine** — the conductor. It assembles the system prompt, processes commands, runs the main loop, and streams the response.
- **Three supporting layers**: *tools* (what the agent can do), *services* (API, compression, MCP, telemetry), and *state* (permissions, file history, active agents, modes).

---

## 7. Lifecycle of a Single Request

What exactly happens between pressing Enter and seeing the response:

```text
USER INPUT (prompt / slash command)
        |
        v
  parse /commands, assemble UserMessage
        |
        v
  assemble system prompt (tools -> sections, CLAUDE.md memory)
        |
        v
  write transcript to disk (JSONL)
        |
        v
  ┌── normalize messages for API (compress if needed)
  │       |
  │       v
  │   Claude API (streaming) — POST with tools and system prompt
  │       |
  │       ├── text block --> deliver to consumer
  │       └── tool_use block?
  │              |
  │              v
  │        permission check (hooks + rules + user prompt)
  │              ├── DENY --> tool_result(error), continue loop
  │              └── ALLOW --> execute tool --> add tool_result
  └────────── return to API call
        |
        v  (stop_reason != "tool_use")
  final message — text, cost, session id
```

Note two important details. First, the **transcript is written to disk before the API call** — if the process crashes, the session can be restored. Second, a **permission denial does not break the loop**: the agent gets `tool_result` with an error and continues working, potentially trying a different approach.

---

## 8. Tool System

Every tool implements a single interface. Adding a tool = adding one handler, while the loop itself doesn't change.

```text
TOOL LIFECYCLE
  validateInput()      — reject bad arguments upfront
  checkPermissions()   — tool-specific permission checks
  call()               — execute and return result

CAPABILITIES
  isEnabled()          — feature flag check
  isConcurrencySafe()  — safe to run in parallel?
  isReadOnly()         — any side effects?
  isDestructive()      — irreversible operations?

RENDERING (React/Ink) — how to show input/output/progress in terminal
AI-SIDE — prompt() and description() describe the tool for the model
```

It's worth highlighting the `isConcurrencySafe()` flag. Read-only tools (e.g., search and file reading) can be run **in parallel** — this dramatically speeds things up. Tools that modify files or run commands, however, execute **sequentially** to avoid interfering with each other.

### Tool Categories

| Category | Tools | Purpose |
|---|---|---|
| **Files** | FileRead, FileEdit, FileWrite, NotebookEdit | reading and editing files |
| **Search** | Glob, Grep, ToolSearch | codebase navigation |
| **Execution** | Bash, PowerShell | running terminal commands |
| **Web** | WebFetch, WebSearch | fetching data from the internet |
| **Agents/Tasks** | Agent, TaskCreate/Update/List, SendMessage | delegation and coordination |
| **Planning** | EnterPlanMode, ExitPlanMode, TodoWrite | structuring work |
| **MCP** | MCPTool, ListMcpResources, ReadMcpResource | external integrations |
| **System** | Config, Skill, ScheduleCron | settings and extensions |

> [!TIP]
> Read-only tools (Glob, Grep, Read) are safe and usually execute without confirmation. Be careful with Bash, Write, and Edit — these are the ones to keep under permission control (see next section).

---

## 9. Permissions System

Permissions are the agent's main security mechanism. Before a tool executes, the request passes through several «gates»:

```text
TOOL INVOCATION REQUEST
        |
        v
  validateInput()  — reject invalid input before any checks
        |
        v
  PreToolUse hooks  — user commands from settings.json
                      can: approve / deny / modify input
        |
        v
  Permission rules  — alwaysAllow / alwaysDeny / alwaysAsk
        |
    no match?
        |
        v
  Interactive prompt — Allow Once / Allow Always / Deny
        |
        v
  checkPermissions() — tool logic (e.g., path sandbox)
        |
    APPROVED --> call()
```

### Permission Modes

Claude Code supports several modes that set the overall behavior:

- **default** — asks permission for potentially dangerous actions. The safe default choice.
- **plan** — agent only plans and reads, but **changes nothing**. Ideal for exploring unfamiliar code.
- **acceptEdits** — automatically accepts file edits, but still asks about commands.
- **bypassPermissions** — no questions asked (dangerous, only for isolated environments like containers).

> [!WARNING]
> `bypassPermissions` mode gives the agent full freedom. Use it **only** in a sandbox/container where there's nothing to break. Never run it on a production machine with access to live systems.

### Rules in settings.json

Rules let you describe once what is allowed without question and what is always forbidden:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run test:*)",
      "Read(src/**)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Read(.env)"
    ]
  }
}
```

This way you allow tests and reading source code, but permanently forbid destructive commands and reading secrets.

---

## 10. Slash Commands

Slash commands (`/command`) are a quick way to manage your session. There are about 80 of them; here are the categories and the most useful ones:

| Category | Commands | What they do |
|---|---|---|
| **Session** | `/clear`, `/compact`, `/resume`, `/cost` | context and history management |
| **Project** | `/init`, `/memory`, `/review` | project memory and review |
| **Model** | `/model`, `/config` | model selection and settings |
| **Planning** | `/plan` | enter plan mode |
| **Agents** | `/agents` | manage sub-agents |
| **MCP** | `/mcp` | MCP server status |
| **Authentication** | `/login`, `/logout` | sign in and out |

### Custom Commands

You can create **custom commands** — they're just markdown files in `.claude/commands/`. For example, `.claude/commands/test.md`:

```markdown
Run all tests, find the failing ones, and suggest fixes.
Response format: first list the failing tests, then for each — cause and fix.
```

Now in a session, `/test` will execute this scenario. This is how you codify recurring team tasks.

---

## 11. Hooks and settings.json

**Hooks** are your own shell commands that Claude Code runs at specific moments in the lifecycle. They let you inject your own logic without touching the agent's code.

| Hook | When it fires | Typical use |
|---|---|---|
| `PreToolUse` | before tool invocation | block dangerous command, log |
| `PostToolUse` | after tool invocation | auto-formatting, linting |
| `UserPromptSubmit` | when prompt is submitted | context injection, audit |
| `Stop` | when response finishes | notifications, reports |

Example: automatically run a formatter after every file edit. In `settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "prettier --write \"$CLAUDE_FILE_PATHS\"" }
        ]
      }
    ]
  }
}
```

Now any file the agent modifies automatically goes through Prettier — without a single reminder.

### Where Settings Live

Settings are read with priority (lower overrides higher):

```text
~/.claude/settings.json           — global, for all projects
<project>/.claude/settings.json   — project settings (in git, for the team)
<project>/.claude/settings.local.json — local, yours only (in .gitignore)
```

> [!TIP]
> Put team-wide rules in `.claude/settings.json` and commit to the repository — so the whole team gets the same agent behavior. Personal preferences — in `settings.local.json`.

---

## 12. MCP: Connecting External Tools

**MCP (Model Context Protocol)** — an open protocol that lets you connect external data sources and tools to the agent: databases, task trackers, APIs, file systems. It's a way to extend the agent's capabilities without modifying the agent itself.

```text
MCP ARCHITECTURE
  MCPConnectionManager
    ├── Server discovery (from settings.json)
    │     ├── stdio  — spawn child process
    │     ├── sse    — HTTP EventSource
    │     ├── http   — Streamable HTTP
    │     └── ws     — WebSocket
    │
    ├── Client lifecycle
    │     ├── connect -> initialize -> list tools
    │     ├── calls via MCPTool wrapper
    │     └── reconnection with backoff
    │
    └── Tool registration
          ├── naming: mcp__<server>__<tool>
          ├── schema pulled from server dynamically
          └── permissions go through the same Permissions system
```

### Connection Example

To give the agent access to the filesystem via an official MCP server, add to config:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
    }
  }
}
```

After restarting, the server's tools will appear under names like `mcp__filesystem__read_file`. Check connection status with `/mcp`.

> [!NOTE]
> MCP tools go through the **same permission checks** as built-in ones. An external server cannot bypass your permission system — it simply adds new tools to the common pool.

---

## 13. Sub-Agents and Multi-Agent Systems

When a task is large, it pays to split it up and delegate. Each sub-agent gets a **fresh context**, so the main conversation stays clean while the sub-task is handled in isolation.

```text
MAIN AGENT
     |
  ┌──┴────────────┬───────────────┐
  v               v               v
FORK-AGENT    REMOTE         IN-PROCESS
child          AGENT          partner
process        via bridge     same process
fresh msgs[]   isolated       shared state

LAUNCH MODES:
  default   — in-process, shared conversation
  fork      — child process, fresh messages[], shared file cache
  worktree  — isolated git worktree + fork
  remote    — bridge to remote Claude Code / container

COMMUNICATION:
  SendMessageTool   — messages between agents
  TaskCreate/Update — shared task board
  TeamCreate/Delete — team lifecycle management
```

**Why is this needed?** Imagine the task «refactor the auth module and update tests». The main agent can assign one sub-agent to explore the code, another to write tests, then assemble the results itself. Each works in its own context and doesn't «pollute» the shared conversation with unnecessary details.

### Custom Sub-Agents

You can define a specialized agent in `.claude/agents/`. For example, `reviewer.md`:

```markdown
---
name: reviewer
description: Strict code reviewer. Looks for bugs, security issues, and style problems.
tools: Read, Grep, Glob
---

You are an experienced reviewer. Check code for bugs, vulnerabilities, and style violations.
Don't suggest changes without justification. Be specific and concise.
```

Note: this agent is granted **only read-only tools** — it can analyze but not change code. This is a safe pattern for reviews.

---

## 14. Context Management (Compact)

The context window is not infinite. When tokens run out — old messages are compressed to free up space without losing the essence.

```text
CONTEXT WINDOW BUDGET
┌─────────────────────────────────────────────┐
│ System prompt (tools, permissions, CLAUDE.md) │
├─────────────────────────────────────────────┤
│ Conversation history                         │
│   [compressed summary of old messages]       │
│   --- compression boundary ---               │
│   [recent messages — full detail]            │
├─────────────────────────────────────────────┤
│ Current turn (request + response)            │
└─────────────────────────────────────────────┘

THREE COMPRESSION STRATEGIES:
  autoCompact      — when token threshold is exceeded
                     summarizes old messages with a separate API call
  snipCompact      — removes «dead» messages and stale markers
  contextCollapse  — restructures context for efficiency

COMPRESSION FLOW:
  messages[] --> take messages after compression boundary
        |
        v
  old messages --> Claude API (summarization) --> compressed summary
        |
        v
  [summary] + [compression boundary] + [recent messages]
```

Compression happens automatically, but you can trigger it manually with `/compact` — e.g., before changing topics, to «clean up» context and save tokens.

> [!TIP]
> If responses start «drifting» and the agent forgets the beginning of the session — that's a signal of context overflow. Do `/compact`, and for a completely new task — `/clear`.

---

## 15. Memory and On-Demand Knowledge

Claude Code doesn't load all knowledge into the system prompt (that's expensive and bloats context). Instead, it loads it **lazily**, when needed.

- **CLAUDE.md** — memory files, read on demand for each directory. Put project rules, code style, build commands, and important notes here.
- **SkillTool** — skills are injected via `tool_result`, not in the system prompt, saving context.
- **Skills/notes directory** — a knowledge store available to the agent on demand.

### CLAUDE.md Hierarchy

Memory files are layered from general to specific:

```text
~/.claude/CLAUDE.md              — personal rules for all projects
<project>/CLAUDE.md              — project rules (in git, for the team)
<project>/<subdir>/CLAUDE.md     — rules for a specific subfolder
```

### A Good CLAUDE.md

```markdown
# Project: My App

## Stack
- Backend: Node.js + Fastify
- Frontend: React + Vite
- DB: PostgreSQL

## Commands
- \`npm run dev\` — start in development
- \`npm test\` — tests (required after changes)
- \`npm run lint\` — style check

## Rules
- Use TypeScript strictly, no \`any\`.
- Cover all new endpoints with tests.
- Don't touch files in \`legacy/\` without explicit request.
```

> [!TIP]
> Keep `CLAUDE.md` short and specific. It's not project documentation, but a «cheat sheet for the agent»: commands, conventions, prohibitions. Long files eat context and degrade quality.

---

## 16. Plan Mode

Plan mode is when the agent **thinks and drafts a plan first, then acts**. In this mode, it reads code and reasons, but **changes nothing** until you approve the plan.

```text
NORMAL MODE:   request -> immediate edits -> result
PLAN MODE:     request -> investigation -> PLAN -> [your approval] -> edits
```

How to enable: the `/plan` command or launch with a flag. Benefits:

- **Safety** — the agent breaks nothing until you agree.
- **Transparency** — you see the intent before the actions.
- **Quality** — «an agent without a plan drifts»; an explicit plan noticeably increases the share of successfully solved tasks.

> [!TIP]
> For unfamiliar or critical code, always start with plan mode. Read the plan, correct if the agent misunderstood the task, and only then allow execution.

---

## 17. Saving and Restoring Sessions

Every session is written to disk as a journal — this allows recovery after restart or crash.

```text
SESSION STORAGE
  ~/.claude/projects/<hash>/sessions/
    └── <session-id>.jsonl   — append-only journal
         ├── {"type":"user",...}
         ├── {"type":"assistant",...}
         └── {"type":"system","subtype":"compact_boundary",...}

RECOVERY:
  --continue        — last session in the current folder
  --resume <id>     — specific session
  --fork-session    — new id, copy of history
```

The write strategy is designed for reliability: user messages are written **synchronously** (so they're definitely not lost on crash), while assistant responses are «fire and forget» with order preservation. JSONL format (one object per line) is easy to read and parse.

**Practice:** you interrupted work — come back with `claude --continue`. Need a specific old session — `claude --resume <id>` or interactively via `/resume`.

---

## 18. 12 Production-Grade Mechanisms

This is the core of the guide. Each subsequent mechanism builds on the previous one — this is how a flat loop becomes a production agent.

| # | Mechanism | One-sentence essence |
|---|---|---|
| 1 | **Loop** | «one loop and Bash is enough»: while-true calls API, checks stop_reason, executes tools |
| 2 | **Tool dispatch** | adding a tool = adding one handler; the loop doesn't change |
| 3 | **Planning** | an agent without a plan drifts; first a list of steps, then execution |
| 4 | **Sub-agents** | large tasks are split; each sub-agent has a fresh context |
| 5 | **On-demand knowledge** | load knowledge when needed, via tool_result |
| 6 | **Context compression** | context overflows — free up space (three strategies) |
| 7 | **Persistent tasks** | big goals → small tasks → to disk, with dependencies and statuses |
| 8 | **Background tasks** | slow operations in background, agent continues thinking |
| 9 | **Agent commands** | too big for one — delegate to partners with mailboxes |
| 10 | **Command protocols** | a unified request-response pattern governs all communication |
| 11 | **Autonomous agents** | partners scan and pick up tasks on their own, without manual assignment |
| 12 | **Worktree isolation** | each works in its own directory (git worktree), linked by id |

The main idea of this table: **an agent's complexity isn't in the loop, but in the infrastructure around it**. The loop remains the same since the very first section. Everything that makes an agent «production-grade» is 11 layers on top of one simple loop.

---

## 19. Observability and Agent Tracing

An agent without observability is a black box: unclear why it made a decision, where it spent tokens, and at which step it broke. A production agent logs **every turn of the loop** in a structured way.

```
WHAT TO TRACE ON EACH LOOP TURN
─────────────────────────────────
API request       → model, context size (tokens in), latency
model response    → stop_reason, tokens out, turn cost
tool invocation   → name, arguments (no secrets!), duration, success/error
permission check  → decision (allow/deny), which rule fired
context compress  → before/after (tokens), which strategy

SPAN HIERARCHY (OpenTelemetry-compatible)
session
 └── turn (one user turn)
      └── api_call (loop iteration)
           ├── tool_use: bash
           └── tool_use: read_file
```

Three observability levels worth setting up immediately: **logs** (structured JSONL per event), **metrics** (tokens/cost/latency/tool error rate), and **traces** (end-to-end path of one request through all turns and tools).

> [!TIP]
> Log `session_id` and `turn_id` in every entry. Then any incident («agent deleted the wrong file») can be reconstructed from the journal in seconds: you'll see the exact tool, arguments, and the permission rule that allowed it.

---

## 20. Cost and Token Optimization

Tokens are money and latency. In a long agent session, a significant chunk of cost comes from **repeatedly sending the history** on every loop turn. Understanding where tokens go saves multiples.

```
WHERE TOKENS GO (typical session)
────────────────────────────────────
system prompt + tool descriptions  ~ fixed overhead EVERY turn
conversation history                grows linearly with steps
tool output (bash, files)           often the MAIN eater
model responses                     usually small

RULE: input-tokens >> output-tokens. Optimize INPUT.
```

| Technique | Effect |
|---|---|
| Prompt caching for system prompt | don't pay for the static part repeatedly |
| Truncating tool output | don't stuff thousands of log lines into context — only tail/grep |
| Timely `/compact` | linear history growth → flat summary |
| Narrow `Grep`/`Glob` instead of reading everything | read only relevant lines |
| Cheaper model for sub-agents | routine work (sorting, extraction) — on a simpler model |

> [!TIP]
> The `/cost` command shows a breakdown per session. If cost grows faster than task complexity — almost always the culprit is bloated tool output that made it into history. Filter output *before* it returns to `messages[]`.

---

## 21. Prompt Caching

Claude Code's system prompt is huge: descriptions of 40+ tools, permissions, CLAUDE.md. Sending it anew on every turn is wasteful. **Prompt caching** lets you «warm up» a stable prefix once and reuse it.

```
WITHOUT CACHE:  [SYSTEM+TOOLS][history][turn]  ← pay for everything every turn
WITH CACHE:     [====cache-hit====][history][turn]      ← pay only for what's new

CACHE BOUNDARIES (cache breakpoints), from stable to volatile:
1. system prompt + tool definitions   (rarely changes)  ← cache
2. CLAUDE.md / project context         (stable)          ← cache
3. early conversation history          (grows)           ← cache
4. recent messages                     (every turn)      — no cache
```

Key principle: caching works by **prefix**. Keep everything stable at the beginning, everything volatile at the end. One change at the beginning invalidates the entire cache below.

> [!IMPORTANT]
> Order matters most. If you insert «current date» or a random id at the very beginning of the system prompt — caching will never work. Keep dynamic content closer to the end of the context.

---

## 22. Testing and Evaluating Agents (Evals)

Regular unit tests check deterministic code. An agent is **non-deterministic**: the same task can be solved in different ways. So it's evaluated not by «exact output» but by **goal achievement** on a set of scenarios (eval set).

```
AGENT TESTING PYRAMID
───────────────────────────
        ┌───────────────┐
        │  End-to-End    │  «fix bug X in repo» → test green?
        │   scenarios    │  (few, expensive, most valuable)
        ├───────────────┤
        │  Tools         │  each tool: valid input → expected effect
        │  (determin.)   │  (many, cheap, fast)
        ├───────────────┤
        │ Permission     │  does the deny rule really block rm -rf?
        │ checks         │
        └───────────────┘
```

How to evaluate a probabilistic result: define a **verifiable success criterion** (tests pass, file contains required content, command returned 0), run each scenario **multiple times** and look at the success rate (pass@k), and hand off complex cases to **LLM-as-judge** with a clear rubric.

| What to test | How |
|---|---|
| Individual tools | regular unit tests (deterministic) |
| Permission gates | allow/deny scenarios with blocking verification |
| Agent behavior | eval task set + automatic result verification |
| Regressions | run eval set on every prompt/model release |

> [!WARNING]
> Don't evaluate the agent «by eye» after manually checking a couple of tasks. If you change the system prompt or model — run the entire eval set. Improvement on one example often breaks five others.

---

## 23. Security and Prompt Injection Defense

As soon as an agent reads **external data** (web pages, files, command output, tickets), the main threat appears — **prompt injection**: external text contains hidden instructions the agent may take as user commands.

```
CODING AGENT THREAT MODEL
──────────────────────────
1. Prompt injection   malicious text in a file/web → «run rm -rf», «leak .env»
2. Exfiltration       secrets in context → sending out via bash/web
3. Dangerous commands destructive (rm, drop table, git push --force)
4. Perimeter breach   access to paths outside the project, to production

TRUST BOUNDARY
[ user instructions ] = trusted
[ tool/file/web output ] = DATA, not instructions
```

Practical defense is layered: **permissions and sandbox** (deny for `rm -rf`, reading `.env`, writing outside the project), **isolation** of secrets from agent context, **human-in-the-loop** on irreversible actions, and treating any external text as *data*, not commands.

| Layer | What it does |
|---|---|
| Permission rules (`deny`) | hard block of dangerous operations, whatever the model «asks» |
| Path sandbox | `checkPermissions()` doesn't allow going outside the project |
| Secret isolation | keys never enter `messages[]` at all |
| Human-in-the-loop | confirmation on destructive and network operations |

> [!WARNING]
> Never give the agent both access to secrets and the ability to send data out without confirmation. This is the classic exfiltration channel: injection in a read file → read `.env` → `curl` to an external server.

---

## 24. Fault Tolerance: Retries, Timeouts, Idempotency

The real world is unreliable: API responds 429/500, network drops, a command hangs. A production agent shouldn't crash on the first error — it **recovers**.

```
EXPONENTIAL BACKOFF WITH JITTER
─────────────────────────────────
attempt 1 → error 429 → wait ~1s
attempt 2 → error 429 → wait ~2s   (+ random jitter)
attempt 3 → error 500 → wait ~4s
attempt 4 → success ✓
                    (after N attempts — gracefully give up)

WHAT TO RETRY AND WHAT NOT
429 / 500 / 503 / timeout  → retry (transient)
400 / 401 / 403            → DON'T retry (request/permission error)
```

Basic resilience techniques: **retry with backoff** only on transient errors, **timeouts** on every tool call (a hung `bash` shouldn't hang the session), **idempotency** of repeatable actions, and **graceful degradation** — on tool failure, return `tool_result` with an error so the agent can try another path instead of crashing.

> [!TIP]
> A tool failure is not the end of the loop, but a signal to the model. Return a clear error in `tool_result` («command exceeded 30s timeout») — the agent will often figure out a workaround on its own.

---

## 25. Streaming and Parallelism Under Load

Agent responsiveness rests on two things: **streaming** (user sees the response as it's generated, not after 30 seconds of silence) and **parallel execution** of safe tools.

```
PARALLEL TOOL EXECUTION
────────────────────────────────────
model returned 3 calls in one turn:
  read_file(a)  ┐
  read_file(b)  ├─ isConcurrencySafe? → yes → run TOGETHER
  grep(x)       ┘
                     vs
  write_file(c) → modifies state → strictly SEQUENTIAL

STREAMING FLOW
API (SSE) → text deltas → immediately to terminal
          → tool_use block assembled fully → then executed
```

The safety rule for parallelism is that same `isConcurrencySafe()` flag from the tools section: **read-only** operations (reading, searching) fly as a batch and dramatically speed up codebase exploration, while anything that **modifies** files or state is serialized to avoid races.

> [!TIP]
> The cheapest speed boost — parallelize reads and searches. When the agent explores an unfamiliar project, a dozen parallel `Read`/`Grep` calls turn minutes of waiting into seconds.

---

## 26. Workshop: Build a Mini-Agent Yourself

To really feel the loop, let's build it in ~20 lines of pseudo-Python. This is the core that scales up to all of Claude Code.

```python
messages = [{"role": "user", "content": prompt}]

while True:
    response = claude_api(messages, tools=TOOLS)
    messages.append(response.message)

    if response.stop_reason != "tool_use":
        print(response.text)   # final answer
        break

    for call in response.tool_calls:
        if not check_permission(call):             # permission gate
            result = {"error": "denied"}
        else:
            result = TOOLS[call.name](call.input)  # execute tool
        messages.append({"role": "tool", "content": result})
    # loop repeats — model sees results and decides what's next
```

### Adding Tools (2 are enough to start)

```python
import subprocess

def tool_read_file(args):
    with open(args["path"]) as f:
        return {"content": f.read()}

def tool_bash(args):
    out = subprocess.run(args["cmd"], shell=True, capture_output=True, text=True)
    return {"stdout": out.stdout, "stderr": out.stderr, "code": out.returncode}

TOOLS = {"read_file": tool_read_file, "bash": tool_bash}
```

With just these two tools (reading a file + running a command), the agent can explore a project, run tests, and fix bugs. **This is exactly how any coding agent works at its core.**

### What to Add Next, Layer by Layer

1. **Permission check** before invocation (we already sketched the check function).
2. **Planning** — a todo list the agent manages itself.
3. **History compression** when context overflows.
4. **JSONL logging** for recovery after crashes.
5. **Sub-agents** for large subtasks with clean context.

Going through these five steps, you'll retrace the path from a learning script to Claude Code-level architecture.

### From Pseudocode to a Working Agent (Real API)

Now let's build the same loop, but with the real Anthropic SDK and proper tool schemas. This is a fully working skeleton — add your API key and run it.

```python
import subprocess, json
from anthropic import Anthropic

client = Anthropic()  # key is taken from ANTHROPIC_API_KEY

# 1) Describe tools for the model (JSON Schema)
TOOL_SCHEMAS = [
    {
        "name": "read_file",
        "description": "Read a text file at path and return its contents.",
        "input_schema": {
            "type": "object",
            "properties": {"path": {"type": "string"}},
            "required": ["path"],
        },
    },
    {
        "name": "bash",
        "description": "Execute a shell command and return stdout/stderr/exit code.",
        "input_schema": {
            "type": "object",
            "properties": {"cmd": {"type": "string"}},
            "required": ["cmd"],
        },
    },
]

# 2) Tool implementations
def read_file(path):
    with open(path, encoding="utf-8") as f:
        return f.read()[:10_000]  # truncate to avoid bloating context

def bash(cmd):
    r = subprocess.run(cmd, shell=True, capture_output=True, text=True, timeout=30)
    tail = lambda s: s[-2000:]   # return only the tail of long output
    return json.dumps({"stdout": tail(r.stdout), "stderr": tail(r.stderr), "code": r.returncode})

IMPL = {"read_file": lambda a: read_file(a["path"]),
        "bash":      lambda a: bash(a["cmd"])}

# 3) Permission gate: dangerous — block, rest — ask
DENY = ("rm -rf", "git push --force", ":(){", "mkfs", "dd if=")
def check_permission(name, args):
    if name == "bash":
        cmd = args.get("cmd", "")
        if any(bad in cmd for bad in DENY):
            return False
        return input(f"Execute `{cmd}`? [y/N] ").strip().lower() == "y"
    return True  # reading is safe

# 4) Main agent loop
def run(prompt):
    messages = [{"role": "user", "content": prompt}]
    while True:
        resp = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2048,
            tools=TOOL_SCHEMAS,
            messages=messages,
        )
        messages.append({"role": "assistant", "content": resp.content})

        if resp.stop_reason != "tool_use":
            print(next(b.text for b in resp.content if b.type == "text"))
            return

        results = []
        for block in resp.content:
            if block.type != "tool_use":
                continue
            if not check_permission(block.name, block.input):
                out, err = "Denied by user", True
            else:
                try:
                    out, err = IMPL[block.name](block.input), False
                except Exception as e:
                    out, err = f"Tool error: {e}", True
            results.append({
                "type": "tool_result",
                "tool_use_id": block.id,
                "content": str(out),
                "is_error": err,
            })
        messages.append({"role": "user", "content": results})

if __name__ == "__main__":
    run("Find all .py files in src/, count lines, and show the largest")
```

> [!IMPORTANT]
> Note four production details that weren't in the pseudocode: **timeout** on `bash`, **output truncation** (`tail`) to save tokens, the **`is_error`** flag in `tool_result` (the agent understands the tool failed and tries differently), and a **block-list** of dangerous commands that triggers before the user prompt.

### What One Run Looks Like (Trace)

Request: «Find all .py files in src/, count lines, and show the largest». Here's what happens turn by turn:

```text
TURN 1
  -> API: messages=[user], tools=[read_file, bash]
  <- stop_reason=tool_use  ->  bash("find src -name '*.py'")
  [permissions] command is safe, user: y
  -> tool_result: "src/app.py\nsrc/db.py\nsrc/utils.py"

TURN 2
  <- stop_reason=tool_use  ->  bash("wc -l src/app.py src/db.py src/utils.py")
  [permissions] y
  -> tool_result: "120 src/app.py\n64 src/db.py\n38 src/utils.py"

TURN 3
  <- stop_reason=end_turn (tools no longer needed)
  <- text: "Found 3 files. Largest — src/app.py (120 lines)."
```

Three turns, two tool invocations, one text response. The model itself decided: first *find* the files, then *count* the lines, then *draw a conclusion*. We didn't program this sequence — it's a consequence of the loop.

---

## 27. Recipes: Real-World Use Cases

### Recipe 1. Understanding an Unfamiliar Project

```text
/init                          # generate CLAUDE.md
"Describe the project architecture and entry points"
"Where are HTTP requests handled?"
```

### Recipe 2. Fixing a Failing Test

```text
"Run the tests, find the failing one, explain the cause, and suggest a fix"
# review the plan, approve the edit
```

### Recipe 3. Safe Refactoring

```text
/plan                          # enable plan mode
"Refactor the auth module: split into layers, preserve behavior"
# read the plan -> approve -> agent edits -> runs tests
```

### Recipe 4. Pre-Commit Review

```text
/review
"Check my uncommitted changes for bugs and style issues"
```

### Recipe 5. CI Automation (Headless)

```bash
claude -p "Update CHANGELOG from commits since last tag" --output-format json
```

### Recipe 6. Understanding an Unfamiliar Bug from a Stack Trace

```text
"Here's a stack trace from production (pasted below). Find the cause in the code,
 explain the call chain, and suggest a minimal fix. Don't change anything
 until I confirm."
# agent: Grep by exception name -> Read offending files -> hypothesis -> fix
```

The key here is the phrase «don't change anything until I confirm». It keeps the agent in analysis mode, and you get the breakdown before edits.

### Recipe 7. Mass Renaming Across the Codebase

```text
/plan
"Rename the function `getUser` to `fetchUser` throughout the project:
 update the declaration, all calls, and tests. Don't touch strings in comments
 or logs unless it's an identifier."
# plan -> approval -> Grep finds all occurrences -> Edit each -> run tests
```

> [!TIP]
> For risky mass edits, always use the «`/plan` + frequent commit» combo. Do a `git commit` before starting — if the result isn't satisfactory, `git reset --hard` rolls everything back with one command.

### Recipe 8. Writing Tests for an Existing Module

```text
"Cover the module `src/pricing.py` with tests. First read it,
 list edge cases (zeros, negatives, empty lists),
 then write pytest tests and run them."
```

The agent first *enumerates* edge cases (you can adjust the list), and only then writes code — this way tests come out meaningful, not mechanical.

### Recipe 9. Log Duty (Headless via Cron)

```bash
# Every morning at 9:00 — a summary of errors for the past day
claude -p "Read logs/app.log, group errors from the last 24h \
  by type, highlight top-3 by frequency, and assess severity. Format: markdown." \
  --output-format json | ./post-to-slack.sh
```

### Recipe 10. Dependency Migration with Verification

```text
/plan
"Upgrade library X from version 1.x to 2.x. Read the CHANGELOG for breaking
 changes, find affected places in code, update them, and run tests.
 If tests fail — fix until they're green."
# agent works in a loop: edit -> test -> analyze failure -> edit ...
```

This demonstrates the agent's key advantage over regular autocomplete: the **feedback loop**. It doesn't just write code — it runs tests, sees the result, and iterates until success.

---

## 28. Practical Examples: Step-by-Step Breakdown

Five end-to-end examples where we don't just give a command, but build a real artifact: a custom slash command, a hook, a sub-agent, an MCP integration, and a full debugging session. Each example can be replicated on your own machine.

### Example 1. Custom Slash Command for Code Review

Goal: instead of a long prompt, just type `/review-pr`. Create `.claude/commands/review-pr.md`:

```markdown
Review changes in the current branch relative to main.

Steps:
1. Run `git diff main...HEAD` and read the changes.
2. For each file, evaluate: bugs, resource leaks, edge cases, style.
3. Check whether new code paths are covered by tests.

Response format:
- **Blockers** (must fix) — list with file and line.
- **Notes** (should fix) — list.
- **Good** — what's done right.

The argument $ARGUMENTS is an optional review focus (e.g., "security").
```

Now in a session:

```text
/review-pr security
# agent substitutes "security" into $ARGUMENTS and reviews with that focus
```

> [!TIP]
> The `$ARGUMENTS` placeholder substitutes everything you typed after the command name. This way one command covers both a general review and a narrow focus — without duplicating files.

### Example 2. Hook That Prevents Committing Secrets

Goal: before any file write, check if a secret is about to be written. This is a protective `PreToolUse` hook. In `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          { "type": "command", "command": "python .claude/hooks/no_secrets.py" }
        ]
      }
    ]
  }
}
```

The script `.claude/hooks/no_secrets.py` reads hook data from stdin and returns a decision:

```python
import sys, json, re

data = json.load(sys.stdin)          # what the agent is about to write
content = json.dumps(data.get("tool_input", {}))

PATTERNS = [
    r"sk-[A-Za-z0-9]{20,}",           # API keys like sk-...
    r"AKIA[0-9A-Z]{16}",              # AWS access key
    r"-----BEGIN (RSA )?PRIVATE KEY",  # private keys
]

for p in PATTERNS:
    if re.search(p, content):
        # non-zero exit + message to stderr = BLOCK the operation
        print(f"Blocked: looks like a secret ({p})", file=sys.stderr)
        sys.exit(2)

sys.exit(0)  # clean — allow the write
```

Now, even if the agent accidentally tries to write a key into a file, the hook intercepts the operation *before* the write and returns an error message to the agent — which will try another path.

> [!WARNING]
> Exit code `2` in a `PreToolUse` hook means «block and notify the model». Code `0` means «all clear, proceed». This is a simple and reliable contract for your own security checks.

### Example 3. Sub-Agent Tester with Isolation

Goal: extract test writing and running into a separate agent with its own clean context. File `.claude/agents/tester.md`:

```markdown
---
name: tester
description: Writes and runs tests. Call when coverage or verification is needed.
tools: Read, Grep, Glob, Bash
---

You are a test engineer. Your task is to get tests to green.

Algorithm:
1. Read the target module and understand its contract.
2. List test scenarios: happy-path, boundaries, errors.
3. Write tests in the project's style (look at existing tests).
4. Run them. If red — fix the test or flag a bug in the code.
5. Return a brief report: how many tests, what's covered, what's left.

Don't refactor production code without an explicit reason — only tests.
```

The main agent calls `tester` with one task; it works in its own context and returns a report — the main conversation isn't polluted with hundreds of lines of pytest output.

```text
"Delegate to the tester sub-agent: cover module src/cart.py"
# main agent -> Agent(tester, "cover src/cart.py") -> receives report
```

### Example 4. Connecting a Real Database via MCP

Goal: let the agent read schemas and make safe queries to Postgres. Use a ready-made MCP server. In `.claude/settings.json`:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y", "@modelcontextprotocol/server-postgres",
        "postgresql://readonly_user@localhost:5432/mydb"
      ]
    }
  }
}
```

After restarting, check the connection and work:

```text
/mcp                      # verify postgres: connected
"Show the schema of the orders table and write SQL: top-5 customers by total for 2024"
# tool mcp__postgres__query executes the query and returns the result
```

> [!IMPORTANT]
> Connect databases under a `readonly` user. MCP tools go through the same permission system, but the principle of least privilege on the database side is your last line of defense: even an erroneous `DELETE` simply won't execute.

### Example 5. Full Debugging Session: From Red Test to Fix

Let's bring it all together. A test is failing, we investigate from scratch.

```text
$ claude

> Test test_discount_applies_once is failing. Investigate and fix.

[agent] /plan
  Grep "test_discount_applies_once" -> tests/test_cart.py:42
  Read tests/test_cart.py (scenario: discount shouldn't apply twice)
  Read src/cart.py -> apply_discount() function
  Hypothesis: discount_applied flag is reset on every add_item()

PLAN:
  1. Move flag reset from add_item() to checkout()
  2. Run full cart test suite
  [awaiting approval]

> yes, proceed

[agent] Edit src/cart.py (moves flag reset)
        Bash "pytest tests/test_cart.py -q"
        -> 14 passed
        Done: the cause was premature flag reset in add_item().
         All 14 cart tests green.
```

Let's unpack what worked here from the guide: `/plan` (section 16) kept the agent from blind edits, `Grep`/`Read` (section 8) localized the problem, the permission gate (section 9) gave you a control point before `Edit`, and the `edit -> test -> result` loop (basic loop, section 5) drove the task to green. One example — nearly the entire architecture in action.

> [!TIP]
> Notice the «first localize, then fix» pattern. A good debugging prompt almost always asks the agent to *first find and explain the cause*, and only then fix. This dramatically reduces the share of «random edits».

---

## 29. Best Practices and Anti-Patterns

### ✅ Do This

- **Start with `/init`** — give the agent a `CLAUDE.md` with project context.
- **Use plan mode** for unfamiliar and critical code.
- **Keep tasks narrow** — «fix this test» is better than «fix all bugs».
- **Commit often** — so it's easy to roll back an unsuccessful agent edit.
- **Set up hooks** — auto-formatting and linting after edits save time.
- **Explicitly forbid dangerous actions** — `deny` rules for `rm -rf`, reading `.env`, etc.

### ❌ Avoid This

- **`bypassPermissions` on a work machine** — only in an isolated sandbox.
- **A huge `CLAUDE.md`** — it eats context; keep it short.
- **One infinite session** — periodically do `/clear` or `/compact`.
- **Blindly approving everything** — read what the agent is about to run.
- **Secrets in prompts and files** — don't give the agent access to keys and passwords.

---

## 30. Troubleshooting

| Symptom | Likely cause | What to do |
|---|---|---|
| Agent «forgets» the start of the session | context overflow | `/compact` or `/clear` |
| Responses became slow/expensive | bloated history | `/compact`, start a new session |
| Constantly asks for permission | no rules in settings | add `allow` rules |
| MCP tool not visible | server didn't start | check `/mcp`, restart |
| Editing the wrong files | no project context | run `/init`, clarify the task |
| `command not found: claude` | not in PATH | reinstall globally via npm |
| Agent does unnecessary things | task is too broad | narrow the phrasing, enable plan mode |

> [!TIP]
> The `/cost` command shows how many tokens and money were spent on the session. If the number grows too fast — almost always the culprit is context overflow.

---

## 31. Glossary

| Term | Meaning |
|---|---|
| **Agent** | model + tools + loop that runs until the task is solved |
| **Agent loop** | while-loop: API call → check stop_reason → execute tools → repeat |
| **Tool** | a function the model can call (read file, bash, search, etc.) |
| **tool_use / tool_result** | the model's request to invoke a tool and the result of its execution |
| **stop_reason** | API response field: why the model stopped (needs a tool or is done) |
| **Permissions** | the «gate» system deciding whether a tool can execute |
| **Hook** | your shell command, launched at a point in the agent's lifecycle |
| **MCP** | Model Context Protocol — protocol for connecting external tools |
| **Sub-agent** | child agent with fresh context for a separate sub-task |
| **Compact** | compression/summarization of old history to save context |
| **CLAUDE.md** | project memory file: rules, commands, conventions |
| **Plan mode** | «plan first, then act» mode, no changes until approval |
| **Worktree** | separate git working directory for isolating parallel work |
| **REPL** | interactive mode (Read-Eval-Print Loop) in the terminal |
| **Headless** | non-interactive launch (in scripts, CI) via the `-p` flag |

---

## 32. Frequently Asked Questions (FAQ)

**Do I need a paid plan?** Claude Code requires access to an Anthropic API/account. Check current conditions on the Anthropic website.

**Can I use it offline?** No — the model works via the Anthropic API. Only sessions and settings are stored locally.

**Where is my session data stored?** Locally, in `~/.claude/projects/`. These are regular JSONL files — you can read and parse them.

**How do I set project rules?** Create a `CLAUDE.md` file in the repository root (or run `/init`).

**Is it safe to give the agent terminal access?** Yes, if you use the permission system: keep `default` mode, add `deny` rules for dangerous commands, and use `bypassPermissions` only in a sandbox.

**How is a sub-agent different from a new session?** A sub-agent launches *inside* the current task with a clean context and returns a result to the main agent without polluting the main conversation.

**What if the agent goes off track?** Interrupt it, clarify the task, `/clear` if needed. Frequent commits make it easy to roll back unsuccessful edits.

**How do I connect my database or API?** Via an MCP server — see section 12.

---

## 33. Where to Go Next

- **Build your own mini-agent** from section 26 — the best way to understand how everything works.
- **Set up `CLAUDE.md` and hooks** for your project — you'll feel the quality difference.
- **Explore MCP** — connect a real external tool.
- **Experiment with sub-agents** — define a reviewer and a tester.
- **Read the official Anthropic documentation** for up-to-date details on the specific version.

> Liked the guide? Give the repo a ⭐ and share with colleagues.

---

## 34. Cheat Sheet (Quick Reference)

Everything essential in one place — keep it open alongside.

### Which Mode/Approach for Which Task

```text
                        WHAT KIND OF TASK?
                              |
      ┌───────────────────────┼───────────────────────┐
      v                       v                       v
 single-step?            multi-step?             unfamiliar/
 (ask, understand)       (fix, build)            critical code?
      |                       |                       |
      v                       v                       v
   regular chat           regular agent            /plan first
   or /ask               (default perms)          (read-only,
                              |                    plan -> approve)
                   need external data?
                    (DB, API, tracker)
                              |
                              v
                        connect MCP (section 12)

  Dangerous/irreversible? -> add deny rule to settings.json
  Slow operation?         -> run in background / delegate to sub-agent
  Context «drifting»?     -> /compact, for new topic -> /clear
```

### Installation and Launch

| Command | What it does |
|---|---|
| `npm install -g @anthropic-ai/claude-code` | installation |
| `claude` | interactive mode (REPL) in current folder |
| `claude "question"` | one-shot query |
| `claude -p "..." --output-format json` | headless (scripts, CI) |
| `claude --continue` | continue last session |
| `claude --resume <id>` | return to a specific session |

### Slash Commands (in REPL)

| Command | Purpose |
|---|---|
| `/init` | generate `CLAUDE.md` for the project |
| `/plan` | plan mode (plan first, then act) |
| `/clear` · `/compact` | clear · compress context |
| `/review` | review changes |
| `/model` · `/config` | model selection · settings |
| `/agents` · `/mcp` | sub-agents · MCP server status |
| `/memory` · `/cost` | memory files · tokens and cost |
| `/resume` · `/exit` | restore session · exit |

### Permission Modes

| Mode | Behavior | When |
|---|---|---|
| `default` | asks about dangerous things | default, safe |
| `plan` | read-only + plan | exploring unfamiliar code |
| `acceptEdits` | edits without asking, commands with prompt | trusted flow |
| `bypassPermissions` | no questions | ⚠️ only in sandbox/container |

### Configuration Files

| Path | What it is |
|---|---|
| `~/.claude/settings.json` | global settings (all projects) |
| `<project>/.claude/settings.json` | project settings (in git, for team) |
| `<project>/.claude/settings.local.json` | personal (in `.gitignore`) |
| `CLAUDE.md` | project memory: stack, commands, rules |
| `.claude/commands/*.md` | custom slash commands |
| `.claude/agents/*.md` | custom sub-agents |
| `.claude/hooks/*` | hook scripts |

### Permission Rules in settings.json (Example)

```json
{
  "permissions": {
    "allow": ["Bash(npm run test:*)", "Read(src/**)"],
    "deny":  ["Bash(rm -rf *)", "Read(.env)", "Bash(git push --force*)"]
  }
}
```

### Hooks — Lifecycle Points

| Hook | When | Example |
|---|---|---|
| `PreToolUse` | before tool invocation | block dangerous, check secrets |
| `PostToolUse` | after invocation | auto-format, linting |
| `UserPromptSubmit` | when prompt is submitted | context injection, audit |
| `Stop` | when response finishes | notifications, reports |

> [!TIP]
> If you remember only one thing: **`/init` at project start, `/plan` for risky work, `/compact` when things «drift», and `deny` rules for dangerous actions.** These four habits cover 90% of daily work.

---

## License

The materials in this repository are intended solely for technical research and education. All intellectual property rights belong to the original company. If a rights violation is discovered, please contact the repository owner for removal.
