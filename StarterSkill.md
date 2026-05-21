# Starter Skills

A reference for all currently available skills in this environment, with **what each one does** and **how to install it**.

---

## How Skills Are Installed (Overview)

Skills come from three places in Claude Code:

1. **Built-in skills** — ship with Claude Code itself. No install needed.
2. **Plugins** — installed via the `/plugin` command from a marketplace. Each plugin can bundle many skills, commands, and hooks.
3. **Personal skills** — markdown files dropped into `~/.claude/skills/<skill-name>/SKILL.md`. Anything you author or copy from a friend's repo lives here.

To install a plugin in Claude Code:

```text
/plugin marketplace add <marketplace-url-or-name>
/plugin install <plugin-name>@<marketplace>
```

The official marketplace is `claude-plugins-official` and is added by default.

To install a personal skill, use the **Skills CLI** (`npx skills`) from [skills.sh](https://skills.sh/) — the package manager for the open agent skills ecosystem:

```bash
npx skills find [query]        # search for a skill by keyword
npx skills add <package>       # install a skill from GitHub or other sources
npx skills check               # check for updates
npx skills update              # update all installed skills
```

Installed skills land in `~/.claude/skills/<skill-name>/`. Restart Claude Code after installing so the skill index picks them up.

---

## Built-in Claude Code Skills

These ship with Claude Code — no install steps required.

| Skill | Description |
|---|---|
| **update-config** | Configure the Claude Code harness via `settings.json`. Use for permissions, env vars, hooks, and automated behaviors like "from now on when X" or "whenever Y happens". |
| **keybindings-help** | Customize keyboard shortcuts, rebind keys, add chord bindings, or modify `~/.claude/keybindings.json`. |
| **fewer-permission-prompts** | Scan transcripts for common read-only Bash/MCP calls and add a prioritized allowlist to `.claude/settings.json` to reduce permission prompts. |
| **find-skills** | Discover and install agent skills when looking for functionality that might exist as an installable skill. |
| **init** | Initialize a new `CLAUDE.md` file with codebase documentation. |
| **review** | Review a pull request. |
| **security-review** | Complete a security review of pending changes on the current branch. |
| **simplify** | Review changed code for reuse, quality, and efficiency, then fix any issues found. |
| **loop** | Run a prompt or slash command on a recurring interval (e.g., `/loop 5m /foo`). Omit the interval for self-paced execution. |
| **schedule** | Create, update, list, or run scheduled remote agents (routines) that execute on a cron schedule. |
| **claude-api** | Build, debug, and optimize Claude API / Anthropic SDK apps. Handles prompt caching, model migrations, and SDK feature work. |

**Install:** none — these are available out of the box.

---

## Plugin Skills

### `claude-md-management` plugin

Provides skills for auditing and maintaining `CLAUDE.md` files.

| Skill | Description |
|---|---|
| **claude-md-management:revise-claude-md** | Update `CLAUDE.md` with learnings from the current session. |
| **claude-md-management:claude-md-improver** | Audit and improve `CLAUDE.md` files across a repository. Outputs a quality report and makes targeted updates. |

**Install:**

```text
/plugin install claude-md-management@claude-plugins-official
```

---

## Personal Skills (`~/.claude/skills/`)

These live under your home directory. Install them through the Skills CLI (`npx skills`) from [skills.sh](https://skills.sh/).

### Development workflow

| Skill | Description |
|---|---|
| **tdd** | Test-driven development using a red-green-refactor loop. |
| **diagnose** | Disciplined diagnosis loop for hard bugs and performance regressions. |
| **prototype** | Build a throwaway prototype — terminal app for state/logic or toggleable UI variations. |
| **handoff** | Compact the current conversation into a handoff document for another agent. |
| **improve-codebase-architecture** | Find refactoring opportunities informed by `CONTEXT.md` and `docs/adr/`. |

### Planning & issues

| Skill | Description |
|---|---|
| **triage** | Triage issues through a state machine driven by triage roles. |
| **to-issues** | Break a plan, spec, or PRD into independently-grabbable tracker issues. |
| **to-prd** | Turn the current conversation context into a PRD on the project issue tracker. |
| **grill-me** | Interview the user relentlessly about a plan or design until shared understanding is reached. |
| **grill-with-docs** | Grilling session that challenges your plan against the existing domain model and ADRs. |

### Skill authoring & communication

| Skill | Description |
|---|---|
| **write-a-skill** | Create new agent skills with proper structure and progressive disclosure. |
| **caveman** | Ultra-compressed communication mode — cuts tokens ~75% while keeping technical accuracy. |
| **browsing** | Direct browser control via Chrome DevTools Protocol — multi-tab, form automation, content extraction. |

**Install via the Skills CLI:**

```bash
# Search for a skill
npx skills find tdd

# Install a specific skill (replace with the package name from search results)
npx skills add <github-org>/<repo>

# Most of the skills above come from Matt Pocock's bundle:
npx skills add mattpocock/skills

# Or browse the leaderboard at https://skills.sh/ and grab top bundles:
npx skills add vercel-labs/agent-skills
npx skills add anthropics/skills
```

**Install (assisted):** ask Claude to run the `find-skills` skill — it will search skills.sh and install the right package for you.

> "Use find-skills to install the `tdd` skill."

After installing, restart Claude Code so the new skills appear in the index.

---

## Verifying What's Installed

```bash
ls ~/.claude/skills/                                    # personal skills
cat ~/.claude/plugins/installed_plugins.json            # installed plugins
ls ~/.claude/plugins/cache/claude-plugins-official/     # cached plugin sources
```

Inside Claude Code, type `/` to see all available slash commands (plugin-provided commands appear here), and ask "what skills do you have available?" to confirm the skill index.

---

## Uninstalling

- **Personal skill:** `npx skills remove <package>` (or `rm -rf ~/.claude/skills/<skill-name>` for a manual clean-up).
- **Plugin:** `/plugin uninstall <plugin-name>@<marketplace>`
- **Built-in:** can't be removed — they're part of Claude Code itself.

Restart Claude Code after changes so the skill index refreshes.
