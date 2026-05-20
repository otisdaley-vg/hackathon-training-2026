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
# Hackathon Training Environments

Prebuilt development environments for the AI tooling workshop. Each environment ships with [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) installed and ready to run.

Languages available under `environments/`:

| Folder | Toolchain |
| --- | --- |
| `csharp` | .NET 9 SDK |
| `daytwo` | .NET 9 SDK + Postgres 16 (compose) |
| `typescript` | Node 20 + TypeScript 5 |
| `foundation` | Node 20 + TypeScript 5 |

Every image also includes Node 20, `git`, and `@anthropic-ai/claude-code` on the global `PATH`.

Prebuilt images are published to GHCR on every push to `main`:

- `ghcr.io/otisdaley-vg/hackathon-csharp:latest`
- `ghcr.io/otisdaley-vg/hackathon-daytwo:latest`
- `ghcr.io/otisdaley-vg/hackathon-typescript:latest`
- `ghcr.io/otisdaley-vg/hackathon-foundation:latest`

You can either pull these directly (see Option 3) or let your tooling build locally from each environment's `Dockerfile` — the devcontainers use the published image as a layer cache, so the first build is fast once CI has run.

> **First-time setup:** GHCR packages are private by default. After CI runs for the first time, open each package at `https://github.com/users/otisdaley-vg/packages/container/hackathon-<lang>/settings` and set its visibility to **Public** so attendees can `docker pull` without authenticating. Repeat for every environment.

## Prerequisites

- [Docker](https://www.docker.com/) (Docker Desktop on macOS/Windows, Docker Engine on Linux)
- For Option 1 only: [VS Code](https://code.visualstudio.com/) with the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

## Option 1 — VS Code Dev Containers

1. Clone this repo.
2. Open the environment you want: `code environments/csharp` (or `daytwo` / `typescript` / `foundation`).
3. When prompted, click **Reopen in Container** — or run the **Dev Containers: Reopen in Container** command from the palette.
4. First build takes a few minutes (Docker pulls the base image, installs Claude Code, etc.); subsequent opens are instant.
5. Open the integrated terminal and run `claude` to start Claude Code.

This also works in [GitHub Codespaces](https://github.com/features/codespaces) — open the repo, create a codespace on the environment folder, and step 4 happens in the cloud.

## Option 2 — Dev Containers CLI (no VS Code)

Same workflow as Option 1, driven from the terminal. Requires Node.

```bash
npm install -g @devcontainers/cli

# Build the image and run postCreateCommand
devcontainer up --workspace-folder environments/csharp

# Drop into a shell inside the container
devcontainer exec --workspace-folder environments/csharp bash
```

Inside the shell, run `claude`. Swap `csharp` for `daytwo`, `typescript`, or `foundation`.

## Option 3 — Docker CLI (no VS Code)

### 3a. Pull the prebuilt image (fastest)

```bash
# bash / zsh
docker pull ghcr.io/otisdaley-vg/hackathon-csharp:latest

docker run -it --rm \
  -v "$PWD/environments/csharp":/workspace \
  ghcr.io/otisdaley-vg/hackathon-csharp:latest
```

```powershell
# PowerShell (Windows / pwsh)
docker pull ghcr.io/otisdaley-vg/hackathon-csharp:latest

docker run -it --rm `
  -v "${PWD}/environments/csharp:/workspace" `
  ghcr.io/otisdaley-vg/hackathon-csharp:latest
```

Swap `csharp` for `daytwo`, `typescript`, or `foundation`.

### 3b. Build the image locally

```bash
# bash / zsh
docker build -t hackathon-csharp environments/csharp

docker run -it --rm \
  -v "$PWD/environments/csharp":/workspace \
  hackathon-csharp
```

```powershell
# PowerShell (Windows / pwsh)
docker build -t hackathon-csharp environments/csharp

docker run -it --rm `
  -v "${PWD}/environments/csharp:/workspace" `
  hackathon-csharp
```

Either way you'll land in a `bash` shell at `/workspace` with `claude` available.

> **Note:** the bind mount hides the dependencies installed at image build time. The first time you mount, re-run the install command for that language inside the container: `dotnet restore` or `npm install`. Subsequent runs reuse what landed on your host.

## Authenticating Claude Code

The first time you run `claude` in a fresh container it will prompt you to sign in. Follow the on-screen flow. Alternatively, set `ANTHROPIC_API_KEY` before launching the container:

```bash
# bash / zsh — set the variable in the parent shell first:
#   export ANTHROPIC_API_KEY=sk-ant-...
docker run -it --rm \
  -e ANTHROPIC_API_KEY \
  -v "$PWD/environments/csharp":/workspace \
  hackathon-csharp
```

```powershell
# PowerShell — set the variable in the parent shell first:
#   $env:ANTHROPIC_API_KEY = "sk-ant-..."
docker run -it --rm `
  -e ANTHROPIC_API_KEY `
  -v "${PWD}/environments/csharp:/workspace" `
  hackathon-csharp
```

For Dev Containers, add the same `remoteEnv` entry to the relevant `devcontainer.json` if you'd rather not log in interactively.
