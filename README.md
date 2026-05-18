# AI for Developers — 4-Lesson Track

A short workshop track to get developers productive with modern AI tooling. Each lesson is ~30 minutes: a short concept section, a hands-on exercise, and a recap.

## Lessons

1. [Foundations](./01-foundations.md) — how LLMs actually work, what they're good and bad at, and how to prompt them
2. [AI Workflows](./02-ai-workflows.md) — the disciplined patterns (spec-driven development, TDD with AI, plan-then-execute, diagnose, critique) that turn AI from autocomplete into reliable engineering practice
3. [Terminal Agents & Sandboxing](./03-terminal-agents.md) — running an autonomous coding agent (Claude Code) that reads, edits, and runs commands on your behalf, and containing it with permissions and sandboxes
4. [MCPs](./04-mcps.md) — extending agents with the Model Context Protocol so they can talk to external tools and data sources

## Requirements

| Tool | Install | Notes |
|---|---|---|
| Browser | [google.com/chrome](https://google.com/chrome) or any modern browser | Required |
| Claude account | [claude.ai](https://claude.ai) | Required — free tier works |
| Node.js v18+ | [nodejs.org](https://nodejs.org) | Required — brings npm and npx with it |
| Claude Code CLI | [claude.ai/code](https://claude.ai/code) | Required — or: `npm install -g @anthropic-ai/claude-code` |
| Git | [git-scm.com](https://git-scm.com) | Required — for Lesson 3 worktree exercises |
| Docker | [docker.com/get-started](https://docker.com/get-started) | Optional — Lesson 3 containerization exercise, skippable |
| Dev Containers (VSCode extension) | [marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) | Optional — Lesson 3 devcontainer exercise, requires Docker |
| devcontainer CLI | `npm install -g @devcontainers/cli` | Optional — Lesson 3 devcontainer exercise, requires Docker |
| API token (GitHub, Linear, Slack, etc.) | Whichever service you use | Optional — Lesson 4, read-only token is fine |

## How to run

- Read in order; each lesson assumes the previous one.
- Exercises assume a Mac/Linux terminal and a recent VSCode install.

## Audience

Developers who already write code daily but haven't yet folded AI tooling into their workflow.
