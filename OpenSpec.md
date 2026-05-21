# OpenSpec Starter — From Zero to Feature Implementation

A walkthrough for getting OpenSpec running in a project and using it to drive a feature from idea → spec → implementation → archive.

OpenSpec is a spec-driven development tool: instead of asking an AI to "just code it," you write a **change proposal** that captures what you're changing and why, get it reviewed, then have the AI implement against that approved spec.

---

## 1. Prerequisites — Check Node

OpenSpec is a Node CLI, so you need Node.js installed first.

```bash
node --version
npm --version
```

You want **Node 20.x or newer** and a working npm. If either command errors:

- **macOS:** `brew install node`
- **Linux/WSL:** use [nvm](https://github.com/nvm-sh/nvm): `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash` then `nvm install --lts`
- **Windows:** install from [nodejs.org](https://nodejs.org) or use `winget install OpenJS.NodeJS.LTS`

Verify again after install:

```bash
node --version   # v20.x or higher
npm --version    # 10.x or higher
```

---

## 2. Install OpenSpec

Install globally so the `openspec` command is on your PATH:

```bash
npm install -g @fission-ai/openspec
```

Confirm it's wired up:

```bash
openspec --version
openspec --help
```

---

## 3. Initialise a Project

From the root of your project (e.g. this repo):

```bash
cd /workspace
openspec init
```

This creates an `openspec/` directory with:

```
openspec/
├── project.md          # high-level project context
├── AGENTS.md           # instructions the AI follows when working with specs
├── specs/              # capability specs (the current "truth" of the system)
└── changes/            # proposed changes (drafts, in-progress work)
```

The CLI will also offer to update your `CLAUDE.md` (or equivalent agent instruction file) so Claude knows how to interact with OpenSpec.

---

## 4. Capture Project Context

Open `openspec/project.md` and fill in:

- **What this project is** (one paragraph)
- **Tech stack** (languages, frameworks, key libraries)
- **Conventions** (testing approach, code style, branching model)
- **Constraints** (deployment target, performance budgets, compliance requirements)

This file is loaded as context every time an agent works on a spec, so investing 10 minutes here saves repeated re-explaining later.

---

## 5. Propose a Change

Pick a feature, bug, or refactor. Ask Claude (or another agent) to create a proposal:

> "Create an OpenSpec change proposal to add OAuth login via GitHub."

Or run the CLI directly:

```bash
openspec change new add-github-oauth
```

This scaffolds a folder under `openspec/changes/add-github-oauth/` containing:

- `proposal.md` — the **why**: problem, goals, non-goals, approach
- `tasks.md` — the **how**: ordered checklist of implementation steps
- `specs/` — the **what**: capability deltas (new/modified/removed behavior)

---

## 6. Review the Proposal

Before any code is written, read the proposal yourself. Good things to check:

- Is the **problem statement** accurate?
- Does the **approach** match your mental model?
- Are the **specs** describing observable behavior, not implementation details?
- Are the **tasks** broken into small, verifiable steps?

Validate that the proposal is internally consistent:

```bash
openspec validate add-github-oauth
```

Iterate with the agent until it's right. This is the cheapest place to course-correct — fixing a spec is faster than fixing code.

---

## 7. Implement Against the Spec

Once approved, tell the agent to implement:

> "Implement the `add-github-oauth` change. Work through `tasks.md` in order, checking off tasks as you go."

The agent will:

1. Read `proposal.md`, `tasks.md`, and the spec deltas
2. Make code changes that match the spec
3. Run tests
4. Tick off completed tasks in `tasks.md`

You can pause, review diffs, and resume at any point. The spec acts as a contract — if the agent drifts, you have an objective document to point back to.

---

## 8. Archive the Change

When the feature is shipped and merged, archive it so the deltas roll forward into the canonical specs:

```bash
openspec archive add-github-oauth
```

This:

- Applies the spec deltas to `openspec/specs/` (the source of truth)
- Moves the change folder to `openspec/changes/archive/`
- Updates timestamps

Now the next proposal sees the new behavior as the baseline.

---

## 9. Day-to-Day CLI Cheatsheet

```bash
openspec init                          # set up openspec/ in a project
openspec change new <slug>             # scaffold a new proposal
openspec change list                   # show in-flight proposals
openspec validate <slug>               # check a proposal for consistency
openspec archive <slug>                # apply deltas, archive the change
openspec specs list                    # show all current capability specs
openspec --help                        # full command reference
```

---

## 10. Using OpenSpec with Claude Code

OpenSpec is designed to be driven by an AI agent. Here's how to wire it up to Claude Code specifically.

### Initial Setup

When you ran `openspec init`, it should have created or updated your `CLAUDE.md` with OpenSpec instructions. Verify:

```bash
grep -i openspec CLAUDE.md
```

If nothing matches, re-run init or manually add a pointer to `openspec/AGENTS.md` from your `CLAUDE.md`:

```markdown
## OpenSpec
This project uses OpenSpec for spec-driven development.
Follow the instructions in `openspec/AGENTS.md` whenever proposing, implementing, or archiving changes.
```

### Slash Commands

OpenSpec ships with custom slash commands that Claude Code picks up automatically once `openspec init` runs (they live in `.claude/commands/`). The common ones:

| Command | What it does |
|---|---|
| `/openspec:proposal <slug>` | Draft a new change proposal from a short description |
| `/openspec:apply <slug>` | Implement an approved proposal end-to-end |
| `/openspec:archive <slug>` | Validate, archive, and roll deltas into canonical specs |

Type `/` in Claude Code to see the full list — anything under `openspec:` came from init.

### Prompt Patterns That Work Well

**Drafting a proposal:**

> `/openspec:proposal add-github-oauth` — I want users to be able to sign in with GitHub. We already have email/password; OAuth should sit alongside it, not replace it. Focus on the spec — don't write code yet.

**Reviewing before implementation:**

> Read `openspec/changes/add-github-oauth/` and tell me what's missing or ambiguous. Don't change anything yet.

**Implementing:**

> `/openspec:apply add-github-oauth` — work through `tasks.md` in order, run the test suite after each task, and stop if anything fails.

**Course-correcting mid-flight:**

> The spec says sessions expire after 24h but you wrote 1h in `auth/session.ts`. Fix the code to match the spec — don't change the spec.

**Archiving:**

> `/openspec:archive add-github-oauth` — validate first, then archive if it passes.

### Tips for Driving Claude

- **Reference the slug, not the files.** Say "the `add-github-oauth` change" — Claude will read the right files via `AGENTS.md` guidance.
- **Make the spec the source of truth.** When Claude and the code disagree, say "the spec wins" — that's the whole point of OpenSpec.
- **Use plan mode for proposals.** Drafting a spec is exploratory; let Claude think through it before writing files.
- **Don't skip validate.** `openspec validate` is cheap and catches silent inconsistencies the agent might otherwise hand-wave through.

---

## 11. Tips for a Smooth Workflow

- **One change = one cohesive concern.** Don't bundle unrelated work — small proposals review faster and roll back cleaner.
- **Specs describe behavior, not code.** "Users can sign in with GitHub" beats "Add `GitHubAuthProvider` class to `auth/`."
- **Treat `tasks.md` as a living todo.** Reorder or split tasks as you learn during implementation.
- **Validate before implementing.** A 30-second `openspec validate` catches contradictions that would otherwise show up as half-finished code.
- **Archive promptly.** Stale proposals in `changes/` create ambiguity about what's actually shipped.

---

## 12. Where to Go Next

- Read `openspec/AGENTS.md` to understand exactly what instructions your agents are following.
- Browse `openspec/specs/` after archiving a few changes — this is your project's behavioral documentation, generated as a byproduct of doing the work.
- Hook OpenSpec into PR templates: link the proposal slug in every PR description so reviewers have full context.
