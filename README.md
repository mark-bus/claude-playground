# Claude Code — Complete Beginner's Guide

> *"There is no one correct way to use Claude Code."*

Claude Code is Anthropic's **agentic coding assistant** that runs directly in your terminal. Unlike a simple chatbot, it can read your files, run commands, edit code, manage git, and even orchestrate multiple AI sub-agents — all from your command line.

- 📖 **Official Docs:** [code.claude.com](https://code.claude.com/docs/en/overview)
- 💻 **Install (recommended):** `curl -fsSL https://claude.ai/install.sh | bash`
- 💻 **Install (legacy npm, deprecated):** `npm install -g @anthropic-ai/claude-code`
- 🔑 **Requires:** A Claude.ai Pro/Max subscription **or** an Anthropic API key ([console.anthropic.com](https://console.anthropic.com))

---

## Table of Contents

1. [Basic Setup & Commands](#i-basic-setup--commands)
2. [MCP & Agent Teams](#ii-mcp--agent-teams)
3. [Skills, Hooks & Plugins](#iii-commands-skills-hooks--plugins)
4. [Demo Workflows](#iv-demo-workflows)
5. [Bonus Tips](#v-bonus-tips)

---

## I. Basic Setup & Commands

### Modifying Claude Models (Effort Setup)

Before you start, you can control **which Claude model** is used and **how hard it thinks**. This is the foundation of tuning performance vs. cost.

- Use `/config` to open the settings menu inside Claude Code.
- The model can be changed from the default (e.g., switching between Sonnet, Opus, Haiku).
- **Higher effort = slower + more tokens consumed = better results for complex tasks.**

📖 See: [Model selection docs](https://code.claude.com/docs/en/overview)

---

### `/init` — Project Context Files

When you run `/init` in a project folder, Claude Code creates a `CLAUDE.md` file. You can also manually create an `AGENTS.md` for multi-agent setups.

| File | Created by | Purpose |
|---|---|---|
| `CLAUDE.md` | `/init` | Tells Claude about your project: tech stack, conventions, special instructions |
| `AGENTS.md` | You (manual) | Defines how sub-agents should behave in multi-agent setups |

**Why this matters for beginners:** Without these files, Claude starts every conversation "blind" about your project. With them, it instantly understands your codebase style, tools, and rules.

> **Tip:** Put things in `CLAUDE.md` like: *"Always use TypeScript. Never use `var`. Tests go in `/tests`."*

📖 See: [CLAUDE.md docs](https://code.claude.com/docs/en/memory)

---

### `.claudeignore` — Hiding Files from Claude

Works exactly like `.gitignore` — list files and folders you **don't** want Claude to read. Place it in the root of your project.

```
# Dependencies — never useful for Claude to read
node_modules/
.pnp/
vendor/

# Secrets & credentials — never let Claude see these
.env
.env.local
.env.production
secrets/
*.pem
*.key

# Build output — Claude doesn't need compiled files
dist/
build/
.next/
out/
*.min.js

# Logs & caches — noise that wastes context
*.log
.cache/
coverage/

# Large data files — too big, not useful
*.csv
*.parquet
fixtures/large/
```

**Why you'd use it:** Two main reasons — **focus** and **safety**.

Focus: Claude reads files to understand context. If it reads 40,000 lines of minified vendor code or auto-generated types, it wastes your entire context window on noise. `.claudeignore` keeps Claude reading only what matters.

Safety: Files like `.env`, private keys, or internal config files should never be sent to an AI. `.claudeignore` is your hard guarantee they won't be, regardless of what `@` references or file scans Claude runs.

> **Tip:** `.claudeignore` and `.gitignore` are separate files. Claude does not automatically respect `.gitignore` — you need to add rules explicitly. A quick shortcut is to copy your `.gitignore` as a starting point and add anything else sensitive on top.

---

### Managing the Context Window

The **context window** is like Claude's short-term memory — it holds the conversation history and all code it has read. Managing it well is crucial.

#### Key facts:
- **Auto-compact triggers at ~80%** of the context window — Claude will summarise old conversation automatically.
- **A fuller context = worse performance.** Claude gets confused or "hallucinates" (makes things up) when context is bloated.
- **Clear context between unrelated tasks** to avoid cross-contamination.

#### Why hallucinations happen with large context:
When the context window is nearly full, Claude has to "compress" older information. Important details can get lost or confused, leading to incorrect code or fabricated file names.

#### The two commands for context management:

**`/compact`** — Summarises old messages while keeping important context. Use this when context usage is high but you want to stay in the same session. You can specify what to keep:
```
/compact retain the authentication flow and error handling patterns
```

**`/clear`** — Complete reset. Deletes all conversation history (frees 100% of tokens) but keeps your `CLAUDE.md` in memory. Use this when switching to a completely different task.

| Situation | Command |
|---|---|
| Context getting full, same task | `/compact` |
| Switching to a new unrelated task | `/clear` |

**`/context`** — Visualises your current context usage as a coloured grid so you can see at a glance how full the context window is. Also warns you if any skills are excluded due to character budget limits.

**Best practice:** Start a fresh session (`/clear`) for each new major task.

📖 See: [Context window management](https://code.claude.com/docs/en/memory)

---

### `/config` — Thinking Mode

Enables **extended thinking**, where Claude reasons step-by-step before answering. This dramatically improves results on hard problems like architecture design, complex bugs, or algorithm challenges.

```
/config → set thinking: true
```

#### Thinking keywords (use in your prompts):

| Keyword | Depth hint | Use When |
|---|---|---|
| `think` | Light | Moderate reasoning needed |
| `think hard` | Medium | Complex problems |
| `ultrathink` | Maximum | The hardest challenges |

> ⚠️ The old fixed token budgets (4k / 16k / 32k) are deprecated and no longer apply. These keywords are now loose hints — the actual reasoning depth is governed by the **effort level** set in `/model`. The keywords still work and do influence behaviour, but don't rely on them as precise controls.

**Example prompt:** `"Ultrathink about the best way to refactor this authentication system"`

> **Beginner tip:** For precise control over reasoning depth, set the effort level in `/model`. Use the thinking keywords as a quick nudge on top of that.

📖 See: [Extended thinking docs](https://code.claude.com/docs/en/overview)

---

### `/resume` — Resume a Previous Conversation

Accidentally closed your terminal? Lost your session?

```bash
/resume
```

This lets you **pick up any previous conversation** exactly where you left off, including all the context Claude had built up.

---

### `/rewind` — Time Travel in a Session

Undoes Claude's last action and returns to a previous conversation state — like Ctrl+Z for your entire AI session.

- **`/rewind`** — Jump back to an earlier state in the *current* session
- **`Esc + Esc`** — Opens the rewind/summarise menu mid-session, where you can:
  - **Rewind code only** — reverts file changes but keeps the conversation history
  - **Rewind conversation** — rolls back the chat as well
  - **Summarise from here** — compacts conversation from a selected point onward

**Why it's useful:** If Claude went down the wrong path and made a mess of your code, `/rewind` lets you undo that whole direction without touching git.

---

### `/fork` — Isolated Sub-Context

```bash
/fork my-experiment
```

Creates a **fork of your current conversation** — a snapshot at this point that continues independently with its own isolated context. Think of it like branching in git, but for your Claude session.

**Use case:** "I want to try two different approaches to this problem — let me fork and compare without losing my current state."

---

### `/sandbox` — Protect Your System

```bash
/sandbox
```

Runs Claude Code in a **restricted environment** where it cannot:
- Format or wipe your hard drive
- Delete important system files or notes
- Execute destructive commands without extra confirmation

> **Beginners should always consider using sandbox mode**, especially when running Claude on large codebases for the first time.

---

### `/doctor` — Diagnose Your Setup

```bash
/doctor
```

Runs a series of checks on your Claude Code installation: API connectivity, Node.js version, project configuration, and file system permissions. If something feels broken, run this first before anything else.

---

### `--dangerously-skip-permissions`

```bash
claude --dangerously-skip-permissions
```

⚠️ **Advanced users only.** This flag bypasses the normal permission prompts Claude shows before taking actions (like editing files or running commands). Useful for fully automated pipelines, but **use with caution** — Claude will act without asking you first.

---

### `!` — Bash Mode (Shell Commands)

Prefix any message with `!` to **run it as a shell command**, just like a terminal:

```
! ls -la
! git status
! npm run build
```

This is powerful because Claude can chain these — run a command, read the output, and react to it intelligently.

---

### `@` — File/Folder Reference

Use `@` to **point Claude at a specific file or folder** in your prompt:

```
Can you refactor @src/utils/auth.ts to use async/await?
Summarize everything in @docs/
```

Claude will read the referenced file(s) and include their content in its context automatically.

---

### Features Blocked in Some Workplaces

These features exist in Claude Code but may be disabled or blocked by your company's internal tooling policy. If any of these don't work for you, it is an organisational restriction, not a bug.

#### ❌ Docker Sandbox

`/sandbox` is available, but it is **not** a full Docker container — it only provides OS-level restrictions. A real Docker sandbox would give Claude a fully isolated filesystem where it could install packages, compile code, and run servers without any risk to your actual machine. Without it, Claude's file operations still happen on your real OS, so mistakes affect real files. The safest workaround is to run Claude Code inside a Docker container yourself, or use it inside a CI/CD environment.

#### ❌ Remote Control (`/remote-control`)

Makes your terminal session available for remote control from claude.ai — so you or a teammate can observe and interact with a running Claude Code session from a browser. Toggled with `/remote-control` (alias `/rc`). Not available internally.

#### ❌ Fast Mode

Makes Opus 4.6 roughly 2.5× faster at the cost of more tokens per response (`/fast` toggle) — not available internally.

📖 See: [Fast mode docs](https://code.claude.com/docs/en/fast-mode)

---

## II. MCP & Agent Teams

### What is MCP?

**MCP (Model Context Protocol)** is an open standard that lets Claude communicate with external tools and software. It's like a universal plugin system for AI models.

Think of it this way: Claude knows *how to code*, but MCP lets it *actually interact* with your IDE, databases, browsers, APIs, and more.

```
Claude LLM  ←→  MCP Protocol  ←→  Any MCP-compatible software
```

📖 See: [MCP documentation](https://code.claude.com/docs/en/mcp)
📖 MCP spec: [modelcontextprotocol.io](https://modelcontextprotocol.io)

---

### JetBrains MCP Setup

Install the **Claude Code [Beta] plugin** in JetBrains IDEs (IntelliJ IDEA, PyCharm, WebStorm, etc.) to give Claude deep integration with your IDE.

**What Claude can see with the plugin:**
- Open files and selected text/rows
- Project structure and file tree
- Git status and history

**What it enables:**
- **Semantic search** — Uses the IDE's built-in index to find relevant code intelligently (not just text search)
- **Reads parts of files, not the full file** — Much more token-efficient
- **Built-in refactor tools** — Class rename, code inspectors, find usages — all available to Claude

#### Why it's token-efficient

Without the JetBrains MCP, Claude's default approach to understanding a codebase is to read full files — sometimes entire directories — and load all that text into the context window. On a large project this is extremely wasteful: Claude ends up reading hundreds of lines of boilerplate, auto-generated code, or unrelated classes just to answer a simple question about one function.

The JetBrains MCP changes this fundamentally. Instead of reading files, Claude queries the IDE's **existing index** — the same one IntelliJ uses for "Go to definition", "Find usages", and autocomplete. This means:

- **Only relevant code is fetched.** If you ask about a method, Claude gets that method and its direct dependencies — not the whole file.
- **Symbol resolution is instant.** Claude can follow a call chain across dozens of files without reading any of them in full.
- **The IDE already parsed everything.** Claude doesn't need to re-read and re-parse source files on every message — the index is always up to date.

In practice, on a complex multi-module project this cuts input token usage by around **90%** compared to plain file reading. For a 1–2 file task the overhead of the MCP call isn't worth it, but that threshold is crossed quickly in any real codebase.

| Scenario | Approach | Token usage |
|---|---|---|
| Large project, many files | JetBrains MCP | ~90% fewer input tokens |
| Small task, 1–2 files | Direct file read (`@file`) | Cheaper, MCP overhead not worth it |

> **Beginner tip:** If you're working on anything bigger than a single script, always have the JetBrains MCP running. The token savings compound quickly across a long session.

📖 Plugin: [JetBrains Marketplace — Claude Code](https://plugins.jetbrains.com/plugin/24358-claude-code)

---

### GIT Worktree Integration

Claude Code is aware of **git worktrees**, which let you work on multiple branches simultaneously in separate directories.

```bash
git worktree add ../my-feature-branch feature-branch
```

Claude can operate in a worktree context, making parallel development safer.

📖 See: [git worktree docs](https://git-scm.com/docs/git-worktree)

---

### MCP Servers: Allowed List

> At some organizations, only a pre-approved list of MCP servers is allowed. Always check with your team's security policy before adding new MCP servers.

To add an MCP server:
```bash
claude mcp add <server-name>
```

📖 See: [MCP server configuration](https://code.claude.com/docs/en/mcp)

---

## III. Commands, Skills, Hooks & Plugins

This is the **extension system** of Claude Code — how you customize, automate, and supercharge it.

### Overview

| Concept | Trigger | What it is |
|---|---|---|
| **Command** | `/cmd` | A custom slash command you define |
| **Skill** | `/skill` or `/prompt` | A reusable prompt template / workflow |
| **Hook** | Automatic | Code that runs at specific lifecycle events |
| **Subagent** | Spawned by Claude | A "mini Claude" with its own isolated context |
| **Template** | Part of skills | Reusable prompt structures |
| **Plugin** | Installed bundle | Skills + targets + hooks + MCP servers packaged together |

---

### Skills (`/skill`)

Skills are **reusable prompt templates** — saved instructions Claude follows for recurring tasks.

Examples:
- A "code review" skill that always checks for security, performance, and style
- A "write tests" skill that follows your team's testing conventions
- A "commit message" skill that formats commits in your style

**You can create your own skills** tailored to your workflow. Skills are stored as files in your project or globally.

> Skills can also be **automatically triggered** — Claude can detect the right skill to use based on context (rather than you manually typing `/skill name` every time).

📖 See: [Claude Code customization](https://code.claude.com/docs/en/skills)

---

### Hooks

Hooks are **automated triggers** that run code at specific points in Claude's lifecycle:

- Before Claude starts a task
- After Claude makes a file edit
- After a conversation ends

**Example use case:** Automatically run your linter every time Claude edits a file, so you always catch syntax errors immediately.

📖 See: [Hooks documentation](https://code.claude.com/docs/en/hooks)

---

### Subagents — Isolated Mini Claudes

Claude can **spawn sub-agents** — independent instances of Claude that work in parallel, each with their own isolated context window.

**Why isolation matters:** Each agent doesn't "know" what the others are doing, which prevents context confusion and makes large tasks parallelizable.

**The `/agents` command** shows you the status of currently running agents.

**Practical example:** Ask Claude to review your entire codebase by spinning up 5 agents, each reviewing a different module simultaneously.

---

### Plugins

A **Plugin** bundles together skills, hooks, MCP servers, and agents — all packaged for one-command installation. Think of them like VS Code extensions but for Claude Code.

> By default, plugins are installed **manually**. But with the right skills configuration, they can activate automatically based on project context.

---

### Official Plugin Store

Anthropic maintains an official, curated plugin directory at:

📖 **[github.com/anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official)**

A few highlighted plugins worth knowing about:

#### 🔧 `feature-dev`

A guided feature development workflow. When you invoke `/feature-dev`, three specialised sub-agents spin up automatically:

- **code-explorer** — reads and maps the relevant parts of the codebase
- **code-architect** — designs the implementation approach
- **code-reviewer** — reviews the result for quality and correctness

Great for structured feature work where you want a plan-then-build flow rather than just asking Claude to "add X".

#### 🔍 `code-review` / `pr-review-toolkit`

Runs a multi-agent PR review with `/pr-review-toolkit:review-pr`. You can target specific review aspects:

| Aspect | What it checks |
|---|---|
| `comments` | Inline code comment quality |
| `tests` | Test coverage and correctness |
| `errors` | Silent failures and unhandled edge cases |
| `types` | Type safety and design |
| `code` | Overall code quality |
| `simplify` | Unnecessary complexity |
| `all` | Everything above |

Each aspect is handled by a dedicated agent — they run in parallel and report back.

#### 🎨 `frontend-design`

Auto-invoked whenever Claude detects frontend work. Provides opinionated guidance on bold design choices, typography, animations, and visual polish — pushing Claude toward more considered UI decisions rather than generic layouts.

#### 🔁 `ralph-loop`

Available in the **[official plugin store](https://github.com/anthropics/claude-plugins-official)**. Turns Claude Code into an autonomous agent that iterates on a task for hours or even days without human intervention. It uses a stop-hook to intercept session exits and re-feed the same prompt, while preserving all file modifications and git history between iterations.

```bash
/ralph-loop "fix all failing tests" --max-iterations 10 --completion-promise "DONE"
```

- The loop runs until Claude outputs the completion promise string (e.g. `DONE`) or hits the iteration limit
- The prompt stays the same each iteration — Claude improves by reading its own previous changes in the codebase
- Always set `--max-iterations` as a safety net

Best for greenfield projects with clear, automated success criteria (e.g. "all tests pass", "no lint errors"). Less suited for vague or design-heavy tasks.

📖 See: [ralph-loop plugin](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/ralph-loop)

---

### Build Your Own: Skills, Plugins & Agents

You're not limited to what Anthropic ships. The entire extension system is open — you can build and share your own.

#### Creating a Skill

A skill is just a folder with a `SKILL.md` file:

```
my-skill/
└── SKILL.md
```

`SKILL.md` starts with YAML frontmatter followed by plain instructions:

```markdown
---
name: my-code-review
description: "Review code for security vulnerabilities and suggest fixes"
---

When reviewing code, always check for:
- SQL injection risks
- Unvalidated user input
- Hardcoded secrets
...
```

Claude reads the description to decide **when** to auto-invoke the skill — no manual trigger needed.

#### Creating a Plugin

A plugin wraps one or more skills (plus optional hooks, agents, MCP servers) into a distributable package:

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json       ← manifest (name, version, description)
├── skills/
│   └── my-skill/
│       └── SKILL.md
├── agents/               ← optional sub-agents
└── hooks/                ← optional lifecycle hooks
```

Test locally with:
```bash
claude --plugin-dir ./my-plugin
```

#### Creating a Sub-Agent

Sub-agents are defined similarly to skills — a markdown file describing their role, tools, and boundaries. They get their own isolated context and can be orchestrated by a parent Claude session or another agent.

#### Sharing with your team

Once built, a plugin can be shared via a private git repo and installed with:
```bash
claude plugin install <repo-url>
```

This makes it trivial to standardise workflows across a whole engineering team — one plugin install and everyone gets the same code review process, the same commit message format, the same architecture review flow.

> 💡 **Tip:** Check out **[skills.sh](https://skills.sh)** — a cross-platform skill directory by Vercel where you can browse, install, and share agent skills with a single command (`npx skills add <skill-name>`), and it works with Claude Code as well as other AI coding tools.

📖 See: [Create plugins — Claude Code Docs](https://code.claude.com/docs/en/plugins) · [Plugin marketplace](https://claude.com/plugins)

---

## IV. Demo Workflows

### Auto Accept vs. Plan Mode

This is one of the most important settings to understand as a beginner.

| Mode | Behavior | Best For |
|---|---|---|
| **Auto Accept** | Claude immediately makes all changes without asking | Trusted, well-defined tasks |
| **Plan Mode** | Claude shows you a plan first; you approve before any changes | New codebases, risky refactors |

**How to switch:**
- `Shift + Tab` — Toggle between modes quickly
- `/plan` — Enter Plan Mode explicitly
- Plan Mode can also be combined with extended **thinking** for maximum safety on critical tasks.

> **Beginners should start in Plan Mode** until they're comfortable with how Claude operates on their codebase.

---

### Model Effort Levels

> ⚠️ There is **no `/effort` slash command**. This is a common misconception. Effort is configured through `/model`, your settings file, or an environment variable.

The effort level controls how eagerly Claude spends tokens on reasoning — trading off speed and cost vs. thoroughness. It is only supported on **Opus 4.6** and **Sonnet 4.6**.

| Level | Description | Typical use case |
|---|---|---|
| `low` | Fastest, cheapest | Quick lookups, high-volume sub-agents, chat |
| `medium` | Balanced | Everyday coding, agentic workflows, code generation |
| `high` | Default | Complex reasoning, hard bugs, architecture decisions |
| `max` | Maximum — **Opus 4.6 only** | Deepest possible analysis |

#### How to set effort level

**Option 1 — In the session UI (the arrow keys you know):**
```
/model   → select a supported model → use ← → arrow keys to move the effort slider
```

**Option 2 — Globally via `settings.json`:**
```json
// ~/.claude/settings.json
{
  "effortLevel": "medium"
}
```

**Option 3 — Environment variable:**
```bash
CLAUDE_CODE_EFFORT_LEVEL=high claude
```

**How effort works:** It's a behavioral signal, not a strict token cap. Claude still reasons at `low` effort — just less deeply than at `high`. At lower effort it responds faster and uses fewer tokens.

**Prompt keywords** (`think`, `think hard`, `ultrathink`) still work as loose hints to push Claude harder, but the effort setting is the authoritative, recommended way to control reasoning depth.

💡 **Pro tip:** Combine Plan Mode + `high` effort for the safest approach to a complex task. Use `low` effort for sub-agents doing repetitive scanning to save costs.

📖 See: [Model configuration docs](https://code.claude.com/docs/en/model-config) · [Effort parameter docs](https://platform.claude.com/docs/en/build-with-claude/effort)

---

### Agent Teams

> ⚠️ **Experimental feature** — Agent Teams must be explicitly enabled before use. It is not on by default.

#### How to enable

**Option 1 — `settings.json` (recommended, persistent):**
```json
// ~/.claude/settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

**Option 2 — Shell profile (persistent):**
```bash
# Add to ~/.bashrc or ~/.zshrc
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

**Option 3 — One-off session:**
```bash
CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 claude
```

Once enabled, just describe the team structure in your prompt — Claude handles spawning and coordinating the agents automatically.

---

For very large codebases, use **multiple agents in parallel** — each scanning or working on a different part of the code simultaneously.

You **don't need TMUX** to use agent teams. Claude Code can spawn and orchestrate sub-agents directly from a single prompt. TMUX is just one way to *visually* manage multiple terminal windows when you want to run agents manually side-by-side.

#### Option 1: Let Claude orchestrate agents automatically (no TMUX needed)

Just ask Claude to use multiple agents in your prompt:

```
"Use separate agents to review each module in /src.
Agent 1: review auth/
Agent 2: review api/
Agent 3: review database/
Then summarize all findings."
```

Claude will spawn sub-agents internally, each with their own isolated context, and collect the results.

#### Option 2: Manual parallel agents with TMUX (advanced)

[TMUX](https://github.com/tmux/tmux/wiki) is a terminal multiplexer — it lets you run multiple terminal sessions visually in one window. Useful when you want full control of each agent.

```bash
# Start a TMUX session with 3 panes
tmux new-session -d -s agents
tmux split-window -h   # split horizontally
tmux split-window -v   # split vertically

# In each pane, run a separate Claude agent
# Pane 1: claude "Review the auth module @src/auth/"
# Pane 2: claude "Review the API layer @src/api/"
# Pane 3: claude "Review database models @src/database/"
```

📖 TMUX guide: [A Beginner's Guide to TMUX](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)

---

### Step-by-Step Task Decomposition

For complex tasks, **don't give Claude one giant instruction**. Instead, tell it:

```
"Split this task into steps first, then we'll tackle each step one at a time."
```

This forces Claude to **plan before acting**, dramatically reducing errors on large refactors, new feature implementations, or architecture changes.

**Why it works:** Claude's reasoning improves when it explicitly breaks a problem down. It catches edge cases it would miss with a single-shot approach.

---

### OpenSpec & Spec-Driven Development (SDD)

**[OpenSpec](https://github.com/Fission-AI/OpenSpec)** (by Fission-AI) is a **Spec-Driven Development (SDD)** framework for AI coding assistants. It has nothing to do with OpenAPI/Swagger — it's a workflow layer that sits on top of Claude Code (and other AI tools).

#### The problem it solves

"Vibe coding" — throwing tasks at an AI through unstructured chat — breaks down on complex projects. Requirements scatter across chat logs, context windows fill up, and the AI gets amnesia, leading to regressions and hallucinations. OpenSpec fixes this by requiring a written spec *before* any code is written.

#### How it works

Running `/opsx:propose` generates a structured **change folder**:

```
openspec/
├── changes/
│   └── my-feature/
│       ├── proposal.md    ← why we're doing this, what's changing
│       ├── specs/         ← requirements and scenarios
│       ├── design.md      ← technical approach (Claude's architecture plan)
│       └── tasks.md       ← implementation checklist
└── config.yaml
```

You review and approve the design *before* Claude writes a single line of code.

#### Integration with Claude Code

OpenSpec adds a `.claude/skills/` directory to your project. This registers custom commands that Claude Code picks up automatically. Claude Code is particularly good at the `/opsx:propose` step — it reads your entire codebase recursively and generates deeply considered architecture in `design.md`.

> **Recommended model:** Opus 4.5 or higher for the planning/proposal step.

📖 GitHub: [github.com/Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec)

---

## V. Bonus Tips

### "There is no one correct way to use Claude Code."

This is an important mindset. Claude Code is a flexible tool — different developers use it completely differently:

- Some run it fully automated in CI/CD pipelines
- Some use it interactively for pair programming
- Some use it as a code reviewer only
- Some use it to generate boilerplate, others for deep debugging

**Experiment** and find what works for your workflow.

---

### Quick Reference Card

```
/init          → Set up project context (CLAUDE.md)
/config        → General settings (thinking mode, preferences)
/compact       → Summarise context to free up space (stay in session)
/clear         → Full context reset — wipe conversation, keep CLAUDE.md
/context       → Visualise context usage as a coloured grid
/resume        → Resume a previous conversation
/rewind        → Jump back in current session (Esc+Esc → rewind/summarise menu)
/fork          → Fork the conversation into an isolated branch
/sandbox       → Run in protected mode
/doctor        → Diagnose your Claude Code installation
/model         → Change model + set effort level with arrow keys
/fast          → Toggle fast mode on/off (Opus 4.6 only)
/plan          → Enter Plan Mode (Shift+Tab to toggle)
/agents        → Manage sub-agent configurations
/diff          → Interactive diff viewer (uncommitted + per-turn diffs)
/review        → Review a PR for quality, security, test coverage
/cost          → Show token usage statistics
/memory        → Edit CLAUDE.md memory files
/skills        → List all available skills
!              → Run shell command directly
@filename      → Reference a specific file or folder
```

---

### Useful Links

| Resource | URL |
|---|---|
| Official Docs | [code.claude.com/docs/en/overview](https://code.claude.com/docs/en/overview) |
| MCP Protocol | [modelcontextprotocol.io](https://modelcontextprotocol.io) |
| JetBrains Plugin | [plugins.jetbrains.com/plugin/24358](https://plugins.jetbrains.com/plugin/24358-claude-code) |
| Anthropic Console | [console.anthropic.com](https://console.anthropic.com) |
| OpenSpec (SDD) | [github.com/Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec) |
| Model config & effort | [code.claude.com/docs/en/model-config](https://code.claude.com/docs/en/model-config) |
| Fast mode | [code.claude.com/docs/en/fast-mode](https://code.claude.com/docs/en/fast-mode) |
| TMUX Guide | [hamvocke.com/blog/a-quick-and-easy-guide-to-tmux](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) |
| Git Worktree | [git-scm.com/docs/git-worktree](https://git-scm.com/docs/git-worktree) |
| Prompting Guide | [platform.claude.com/docs/en/build-with-claude/prompt-engineering](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview) |

---

*Guide based on a live Claude Code demo. All slash commands and features are subject to change — always check the [official docs](https://code.claude.com/docs/en/overview) for the latest.*
