# Lesson 2 — AI Workflows

**Time:** ~30 min
**Goal:** Learn the disciplined patterns that turn AI tools from confident-sounding autocomplete into reliable engineering practice — spec-driven, test-driven, and the smaller set of workflow shapes worth keeping in your back pocket.

## Objectives

By the end of this lesson you will be able to:

- Explain why a *workflow* matters more than the specific model or harness you use.
- Run **spec-driven development (SDD)** — produce a spec as an artifact before any code is written, and hand it to an agent for implementation.
- Pick between hand-rolled SDD and the tools that formalize it: **GitHub Spec Kit**, **OpenSpec**, **Kiro**, **Tessl**.
- Run **TDD with an AI pair**: you write the failing test, the model writes the green, you own the refactor.
- Recognize four other workflow shapes — **plan-then-execute**, **diagnose loop**, **prototype-first**, **critique / grill** — and know when to reach for each.
- Match a workflow to the *shape of the work* instead of dumping every task into a single chat.

## Why workflows matter

A model with no workflow around it gives you one of two things: an answer to a single question, or a long meandering chat that produces ten files and a vague sense of unease.

The workflows below are scaffolding. They give the model a *shape* to work inside — a contract, a test, a plan, a hypothesis — and they give *you* a way to spot trouble before it lands in your repo.

The pattern under all of them is the same: **constrain first, generate second, verify third.** Every workflow is a different answer to *what's the constraint?*

| Constraint | Workflow |
| --- | --- |
| A written specification | Spec-driven development |
| A failing test | Test-driven development |
| An ordered plan | Plan-then-execute |
| A reproducible bug | Diagnose loop |
| A throwaway target | Prototype-first |
| A second pair of eyes | Critique / grill |
| End-to-end slices | Vertical-slice breakdown |
| Mechanical output that feeds code | Structured output |
| Prompt behavior that might have regressed | Evaluations |

You'll mix these in real work. The skill is naming the shape.

## Spec-driven development

The highest-leverage workflow for non-trivial features. You and the model produce a **spec** first; the spec then becomes the source of truth for implementation, review, and tests.

A spec answers: goals, non-goals, interfaces, data shapes, edge cases, test cases. It's the thing two engineers could build the same way from. With AI, the spec also becomes the prompt — agents implementing from a tight spec drift far less than agents implementing from *"build a thing that does X."*

### The flow

1. **Brief.** Tell the model the problem in plain English. Ask it to ask *you* the clarifying questions that would change the design — before it proposes anything.
2. **Draft.** Once the shape is clear, ask for a spec: goals, non-goals, interface, edge cases, test list, open questions.
3. **Iterate.** Treat the spec as a living artifact. Argue with it. Cut scope, add edge cases, demand specifics where it waves hands.
4. **Hand off.** Pass the finalised spec to an agent (or another developer) for implementation. Code review becomes *"does this match the spec?"* — a smaller, faster, less personal question.

You can do all of this in plain markdown in a chat. The reason to pick up a tool is that it gives a *team* the same shape every time.

### Tools that formalize this

- **GitHub Spec Kit** (`github/spec-kit`, installed via the `specify` CLI). Slash commands `/constitution`, `/specify`, `/clarify`, `/plan`, `/tasks`, `/implement` walk a feature from a one-line idea to a discrete task list and implementation. Works with Claude Code, Copilot, Cursor, Gemini, and other harnesses. Artifacts live in `.specify/` in the repo, so the team reviews specs in PRs like any other code.
- **OpenSpec** — lightweight, markdown-first spec management. Specs live in `openspec/specs/`; each one captures purpose, scope, and acceptance criteria. Designed for agents to read, update, and implement against. Lower ceremony than Spec Kit; nice when you just want a folder convention.
- **Kiro** (AWS). Spec-driven IDE. Every feature starts as `requirements.md`, `design.md`, and `tasks.md` files the agent generates and then implements against. Tighter loop, less DIY than Spec Kit or OpenSpec.
- **Tessl.** Hosted spec-driven development platform. Specs are first-class objects with versioning, review, and agents that run against them.
- **Roll your own.** A `specs/` folder, a `SPEC_TEMPLATE.md` checked into the repo, and a habit. Often enough.

Pick the lightest one that fits your team. Spec Kit and OpenSpec are the best entry points if you want a real workflow without buying into an IDE.

> **Like I'm five:** Before you build LEGO, you read the booklet. The spec is the booklet. Without it, the model picks bricks from the pile that *look* right and hopes the end result looks like a fire truck. With it, every brick has a place — and when someone hands you their build, you check it against the booklet, not against your mood.

## Test-driven development with AI

TDD predates AI by twenty years; AI makes it cheaper than ever.

### The loop

1. **Red.** *You* write the failing test. Describe behaviour, not implementation. Run it; watch it fail for the *right reason*.
2. **Green.** Hand the failing test to the model: *"make this test pass, change as little as possible, don't touch the tests."* Read the diff.
3. **Refactor.** With tests green, ask the model (or do it yourself) to clean up — extract helpers, rename, dedupe. Re-run the suite.

Why this works well with AI: the test is a verifier the model can't bluff past. Either it goes green or it doesn't. That single, mechanical signal is worth more than any amount of *"looks good to me."*

### Things to watch for

- Models love to "fix" a failing test by editing the *test*. State up front: *"do not modify the test files."* Some harnesses have a TDD skill that bakes this rule in — Claude Code's bundled `tdd` skill, for instance.
- Models will sometimes hardcode the expected value to make the test pass. Read the diff before you trust the green bar.
- Keep tests behaviour-focused. Coupling tests to implementation details turns a future refactor into a future rewrite.

### Where TDD fits next to SDD

- **Behaviour you can name as a function** — pure functions, parsers, formatters, validators, calculators: lead with **TDD**.
- **Behaviour that spans files, state, or UX**: lead with **SDD**, do **TDD inside** each slice as you implement.

A spec without tests is a wish. Tests without a spec are local optima. Use both.

## Evaluations

TDD gives you a test suite for your code. **Evals** give you a test suite for your *prompts*.

The problem they solve: you change a prompt to fix one failure and it silently breaks three other cases. Without evals, you find out in production. With evals, you find out before the merge.

### The minimal eval loop

1. **Collect examples.** Start with 10–20 input/output pairs that represent the behavior you care about — include edge cases and known failure modes. These are your "golden" outputs.
2. **Run them.** After any prompt change, run your eval set and compare outputs to goldens.
3. **Score.** For exact-match outputs (structured JSON, classifications), score automatically. For prose or reasoning, use a **model-as-judge**: feed the output and the golden to a second model call and ask *"is this correct? Reply yes or no and give a one-sentence reason."*
4. **Track over time.** A regression is a prompt change that moves the score down. An improvement moves it up.

Evals are not a replacement for human review on novel inputs — they're a regression harness. They catch the thing you already knew could break.

### Lightweight tooling

- **promptfoo** — open source, YAML-configured, runs evals locally and in CI. A good starting point.
- **LangSmith / Braintrust** — hosted platforms with tracing. Worth it when you're running many experiments.
- **Roll your own.** A folder of `{input, expected}` pairs and a short comparison script often suffices.

### Where evals sit next to TDD

| | TDD | Evals |
| --- | --- | --- |
| What's being tested | Code correctness | Prompt/model behavior |
| Pass/fail signal | Deterministic | Often probabilistic |
| Written when | Before implementation | From observed behavior |
| Catches | Logic bugs | Prompt regressions |

Use both. TDD covers the code; evals cover the AI. A green test suite and a green eval run together give real confidence.

## Other workflows worth knowing

### Plan-then-execute

The agent drafts a plan; you approve or edit; the agent executes against the approved plan. Most coding harnesses ship a version:

- **Claude Code** — *Plan mode* (Shift+Tab). Produces a numbered plan, waits for approval, then executes.
- **Cursor / Windsurf** — agent modes with an explicit planning step before edits.
- **ChatGPT / generic chat** — *"Make a plan first. Wait for my approval before changing any code."*

Reach for it when the task is multi-step but well-bounded: *"migrate this folder,"* *"add this endpoint."* Skip it for tiny edits — the plan overhead exceeds the work.

### The diagnose loop

For bugs, especially the gnarly intermittent kind:

1. **Reproduce.** Get a one-liner that fails consistently. No repro, no bug.
2. **Minimize.** Strip the repro until removing one more line makes it pass.
3. **Hypothesize.** State the one explanation you'd bet on. Make it falsifiable.
4. **Instrument.** Add logging, prints, breakpoints to falsify the hypothesis.
5. **Fix.** Only after the hypothesis survives instrumentation.
6. **Regression test.** Write a test that fails before the fix and passes after.

AI's role: hold the hypothesis with you, generate the minimal repro, write the regression test. Claude Code's bundled `diagnose` skill wraps exactly this loop.

The big anti-pattern: asking the model to *"fix the bug"* without a repro. You'll get a confident patch that addresses *a* bug, possibly not *yours*.

### Prototype-first

For design questions — *"should the state machine look like this or this?"*, *"what should this dashboard feel like?"* — build a throwaway. Ship it nowhere. Use it to extract the *real* requirements before you commit to a spec.

Two flavours:

- **State / logic prototype** — a runnable terminal app or notebook you can poke. Surfaces edge cases before you build the production version.
- **UI prototype** — several radically different mocks side by side. Pick by feel, not by argument.

The discipline: agree the prototype is throwaway *before* you start, or you'll quietly promote it to production.

### Critique / grill

The model is its own first reviewer. Three useful shapes:

- **Self-critique.** *"Now review the answer you just gave. What's wrong with it? What did you assume?"*
- **Adversarial second pass.** Open a fresh conversation (no prior context) and paste the artifact: *"Find the three biggest weaknesses in this design."*
- **Grilling.** Have the model interview *you* — push back on hand-wavy claims until the plan is concrete. Claude Code's `grill-me` skill packages this.

The cheapest quality lever available. Most AI failures aren't *"the model couldn't do it"* — they're *"nobody asked it to look again."*

### Vertical-slice breakdown

For larger work: break a plan into independent, ship-shaped slices — each one end-to-end through the stack (a "tracer bullet"). Each slice is a separate task or PR. The first slice teaches you what the spec got wrong before you've written nine.

Claude Code's `to-issues` skill formalises this — takes a plan and emits independently-grabbable issues sized as vertical slices.

### Structured output

Most workflows produce text for a human to read. When the output needs to feed *code* — a database write, a branch condition, a downstream prompt — constrain the model to return a specific shape rather than prose.

Tell the model the exact JSON schema you want:

```json
{
  "bugs": [
    { "line": 23, "severity": "high", "description": "SQL injection via string concat" },
    { "line": 41, "severity": "low",  "description": "unused import" }
  ]
}
```

You can then sort by severity, store in a database, or pipe to another tool — no prose parsing required.

Reach for structured output when output plugs into code rather than a human reader: classification, extraction, multi-step pipelines. Skip it for exploratory conversation or drafting — schema ceremony adds noise with no benefit there.

Most frontier APIs support this natively. Include a JSON schema in the system prompt and instruct the model to return only JSON. Claude and GPT-4o are reliable at it.

### Model and cost selection

Not every task needs the same model. Pick the lightest one that handles the job:

| Model | Reach for when |
| --- | --- |
| **Haiku** (fast, cheap) | Simple extraction, classification, yes/no routing, high-volume pipelines |
| **Sonnet** (the workhorse) | Most coding tasks, multi-step reasoning, tool use |
| **Opus** (expensive) | Hard multi-step problems, frontier reasoning, the cases Sonnet fumbles |

Start with Sonnet. If quality is the problem, try Opus. If cost or latency is the problem and the task is mechanical, try Haiku.

**Prompt caching** is the highest-leverage cost lever most teams overlook. If you send the same large system prompt or document on every call — an agent re-reading CLAUDE.md on every turn, a pipeline with a shared reference doc — the API caches the stable prefix after the first call and charges a fraction of the price on subsequent reads. On repeated-context workloads, caching cuts costs 80–90%.

The cheapest token is the one you don't send. Before adding context to a prompt, ask: does this actually change the output? If not, cut it.

## Picking a workflow

Quick decision tree:

- *Small, well-defined?* — just ask. Workflows have overhead.
- *New feature or non-trivial change?* — **SDD**. Spec first.
- *Pure function-shaped behaviour?* — **TDD**, inside the SDD slice.
- *A bug?* — **Diagnose loop**.
- *Design unclear?* — **Prototype**.
- *Artifact drafted but worried it's wrong?* — **Critique / grill**.
- *Big plan, multiple people or agents?* — **Vertical-slice breakdown**.
- *Output needs to feed code, not a human reader?* — **Structured output**.
- *Prompt changed and worried it regressed?* — **Evals** against your golden set.
- *Task is high-volume or latency-sensitive?* — **Model selection**: try Haiku before assuming Sonnet is the floor.

A real feature often runs: brief → spec (SDD) → plan (plan-mode) → slice (vertical) → TDD each slice → critique the whole thing before merge.

## Hands-on (15 min)

**Exercise 1 — Spec and build a todo list or kanban board with OpenSpec.** In a fresh folder (`todo-app` or `kanban`), install OpenSpec following its current docs (typically `npx openspec@latest init`). Open the folder in Claude Code (or your agent of choice). Tell the agent: *"Use OpenSpec to propose a change that introduces a todo list CLI"* (or a kanban board web app — pick one). The agent should create a change proposal under `openspec/changes/` capturing goals, scope, and acceptance criteria *before* writing any code. Read the proposal. Push back on anything vague — make the acceptance criteria specific and testable. Once you're happy, have the agent implement against the approved spec. The artifact you care about is the spec file as much as the working code.

**Exercise 2 — Add a feature with OpenSpec.** Same project. Pick a new feature — tags, due dates, priority colours, drag-and-drop between columns, search, archive. Ask the agent to draft a *new* OpenSpec change proposal first; do not let it write code until the proposal exists. Review it: are the acceptance criteria testable? Are the edge cases (empty state, conflicts, undo) listed? Iterate until they are, then green-light the implementation. Notice what the spec process catches that a one-line *"add tags"* prompt would have missed.


## Recap

- **Constrain first, generate second, verify third** is the spine of every AI workflow worth using.
- **SDD** — the spec is the contract. Use Spec Kit, OpenSpec, Kiro, Tessl, or roll your own. Code review becomes *"does this match the spec?"*
- **TDD with AI** — you write the failing test, the model writes the code, you own the refactor. The test is a verifier the model can't bluff.
- **Evals** — a test suite for your prompts, not your code. Golden examples + a comparison script catch regressions before merge. Use a model-as-judge to score prose outputs.
- **Structured output** — when the output feeds code rather than a human, constrain the shape. Pair with model selection: Haiku for high-volume mechanical tasks, Sonnet for most work, Opus for hard reasoning. Prompt caching cuts costs 80–90% on stable repeated contexts.
- **Plan-then-execute, diagnose, prototype, critique, vertical-slice** are the other shapes worth knowing — each is a different constraint.
- The skill is matching the workflow to the *shape of the work*, not running every task through the same chat.

**Next:** [Lesson 3 — Terminal Agents](./03-terminal-agents.md), where we hand a spec from this lesson to an actual agent and watch it run.
