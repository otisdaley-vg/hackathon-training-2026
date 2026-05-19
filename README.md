# Hackathon Training Environments

Prebuilt development environments for the AI tooling workshop. Each environment ships with [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) installed and ready to run.

Languages available under `environments/`:

| Folder | Toolchain |
| --- | --- |
| `python` | Python 3.11 |
| `typescript` | Node 20 + TypeScript 5 |
| `php` | PHP 8.2 + Composer |
| `csharp` | .NET 9 SDK |

Every image also includes Node 20, `git`, and `@anthropic-ai/claude-code` on the global `PATH`.

## Prerequisites

- [Docker](https://www.docker.com/) (Docker Desktop on macOS/Windows, Docker Engine on Linux)
- For Option 1 only: [VS Code](https://code.visualstudio.com/) with the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

## Option 1 — VS Code Dev Containers

1. Clone this repo.
2. Open the environment you want: `code environments/python` (or `typescript` / `php` / `csharp`).
3. When prompted, click **Reopen in Container** — or run the **Dev Containers: Reopen in Container** command from the palette.
4. First build takes a few minutes (Docker pulls the base image, installs Claude Code, etc.); subsequent opens are instant.
5. Open the integrated terminal and run `claude` to start Claude Code.

This also works in [GitHub Codespaces](https://github.com/features/codespaces) — open the repo, create a codespace on the environment folder, and step 4 happens in the cloud.

## Option 2 — Dev Containers CLI (no VS Code)

Same workflow as Option 1, driven from the terminal. Requires Node.

```bash
npm install -g @devcontainers/cli

# Build the image and run postCreateCommand
devcontainer up --workspace-folder environments/python

# Drop into a shell inside the container
devcontainer exec --workspace-folder environments/python bash
```

Inside the shell, run `claude`. Swap `python` for `typescript`, `php`, or `csharp`.

## Option 3 — Docker CLI (no VS Code)

Pick the environment you want and build its image. Run these commands from the repo root.

```bash
# Python
docker build -t hackathon-python environments/python

# TypeScript
docker build -t hackathon-typescript environments/typescript

# PHP
docker build -t hackathon-php environments/php

# C#
docker build -t hackathon-csharp environments/csharp
```

Then start an interactive container with your code bind-mounted, so edits on your host show up inside:

```bash
docker run -it --rm \
  -v "$PWD/environments/python":/workspace \
  hackathon-python
```

You'll land in a `bash` shell at `/workspace` with `claude` available. Swap the image and bind-mount path for the language you built.

> **Note:** the bind mount hides the dependencies installed at image build time. The first time you mount, re-run the install command for that language inside the container: `npm install`, `composer install`, `dotnet restore`, or `pip install -r requirements.txt`. Subsequent runs reuse what landed on your host.

## Authenticating Claude Code

The first time you run `claude` in a fresh container it will prompt you to sign in. Follow the on-screen flow. Alternatively, set `ANTHROPIC_API_KEY` before launching the container:

```bash
docker run -it --rm \
  -e ANTHROPIC_API_KEY \
  -v "$PWD/environments/python":/workspace \
  hackathon-python
```

For Dev Containers, add the same `remoteEnv` entry to the relevant `devcontainer.json` if you'd rather not log in interactively.
