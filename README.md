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

Prebuilt images are published to GHCR on every push to `main`:

- `ghcr.io/otisdaley-vg/hackathon-python:latest`
- `ghcr.io/otisdaley-vg/hackathon-typescript:latest`
- `ghcr.io/otisdaley-vg/hackathon-php:latest`
- `ghcr.io/otisdaley-vg/hackathon-csharp:latest`

You can either pull these directly (see Option 3) or let your tooling build locally from each environment's `Dockerfile` — the devcontainers use the published image as a layer cache, so the first build is fast once CI has run.

> **First-time setup:** GHCR packages are private by default. After CI runs for the first time, open each package at `https://github.com/users/otisdaley-vg/packages/container/hackathon-<lang>/settings` and set its visibility to **Public** so attendees can `docker pull` without authenticating. Repeat for all four environments.

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

### 3a. Pull the prebuilt image (fastest)

```bash
docker pull ghcr.io/otisdaley-vg/hackathon-python:latest

docker run -it --rm \
  -v "$PWD/environments/python":/workspace \
  ghcr.io/otisdaley-vg/hackathon-python:latest
```

Swap `python` for `typescript`, `php`, or `csharp`.

### 3b. Build the image locally

```bash
docker build -t hackathon-python environments/python

docker run -it --rm \
  -v "$PWD/environments/python":/workspace \
  hackathon-python
```

Either way you'll land in a `bash` shell at `/workspace` with `claude` available.

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
