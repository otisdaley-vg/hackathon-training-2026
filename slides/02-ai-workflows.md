---
marp: true
theme: default
paginate: true
size: 16:9
header: 'AI for Developers — Lesson 2'
footer: 'AI Workflows'
style: |
  section { font-size: 24px; }
  h1 { color: #1f2937; }
  h2 { color: #2563eb; }
  code { background: #f3f4f6; padding: 2px 6px; border-radius: 4px; }
  table { font-size: 0.85em; }
  blockquote { border-left: 4px solid #2563eb; color: #4b5563; }
---

# Lesson 2 — AI Workflows

**Time:** ~30 min

**Goal:** Learn the disciplined patterns that turn AI tools from confident-sounding autocomplete into reliable engineering practice.

---

## Questions?

**Interject anytime** — please ask if anything's unclear.

**Drop them in the chat too** — I'll collect everything so we can build a shared `faq.md` together after the session, for everyone to reference later.

---

## Objectives

By the end of this lesson you will be able to:

- Explain why a **workflow** matters more than the specific model or harness
- Run **spec-driven development (SDD)** — produce a spec before any code
- Pick between hand-rolled SDD and tools that formalize it
- Run **TDD with an AI pair** — you write the test, the model writes the green
- Recognize **plan-then-execute, diagnose, prototype-first, critique/grill**
- Match a workflow to the *shape of the work*

---

## Why workflows matter

A model with no workflow gives you one of two things:

- An answer to a single question
- A long meandering chat that produces ten files and a vague sense of unease

Workflows give the model a **shape** to work inside — a contract, a test, a plan, a hypothesis.

> **Constrain first, generate second, verify third.**

---

## Every workflow answers: *what's the constraint?*

| Constraint | Workflow |
|---|---|
| A written specification | Spec-driven development |
| A failing test | Test-driven development |
| An ordered plan | Plan-then-execute |
| A reproducible bug | Diagnose loop |
| A throwaway target | Prototype-first |
| A second pair of eyes | Critique / grill |
| End-to-end slices | Vertical-slice breakdown |
| Mechanical output → code | Structured output |
| Prompt might have regressed | Evaluations |

---

## Spec-driven development (SDD)

The highest-leverage workflow for non-trivial features.

You and the model produce a **spec** first; the spec becomes the source of truth.

A spec answers:
- Goals, non-goals
- Interfaces, data shapes
- Edge cases, test cases

**With AI, the spec also becomes the prompt** — agents drift far less from a tight spec than from *"build a thing that does X."*

---

## The SDD flow

1. **Brief.** Tell the model the problem in plain English. Ask it to ask *you* clarifying questions first.
2. **Draft.** Ask for a spec: goals, non-goals, interface, edge cases, test list, open questions.
3. **Iterate.** Argue with it. Cut scope, add edge cases, demand specifics.
4. **Hand off.** Pass the finalized spec to an agent for implementation.

Code review becomes *"does this match the spec?"* — smaller, faster, less personal.

---

## Tools that formalize SDD

- **GitHub Spec Kit** (`specify` CLI) — `/constitution`, `/specify`, `/clarify`, `/plan`, `/tasks`, `/implement`. Artifacts in `.specify/`.
- **OpenSpec** — lightweight, markdown-first. Specs in `openspec/specs/`. Lower ceremony.
- **Kiro** (AWS) — spec-driven IDE. Generates `requirements.md`, `design.md`, `tasks.md`.
- **Tessl** — hosted, first-class spec objects with versioning.
- **Roll your own** — a `specs/` folder, `SPEC_TEMPLATE.md`, and a habit.

Pick the lightest one that fits your team.

---

## SDD — Like I'm five

> Before you build LEGO, you read the booklet.
>
> The **spec is the booklet**.
>
> Without it, the model picks bricks that *look* right and hopes the result looks like a fire truck.
>
> With it, every brick has a place — and you check the build against the booklet, not against your mood.

---

## Test-driven development with AI

TDD predates AI by twenty years; AI makes it **cheaper than ever**.

The loop:

1. **Red.** *You* write the failing test. Describe behaviour, not implementation. Watch it fail for the *right reason*.
2. **Green.** Hand the failing test to the model: *"make this pass, change as little as possible, don't touch the tests."*
3. **Refactor.** Tests green → clean up. Re-run.

---

## Why TDD works with AI

The test is a **verifier the model can't bluff past**.

Either it goes green or it doesn't.

That single mechanical signal is worth more than any amount of *"looks good to me."*

---

## TDD — things to watch for

- Models love to *"fix"* a failing test by editing the **test**.
  → State up front: *"do not modify test files."*
- Models will sometimes hardcode the expected value to pass the test.
  → **Read the diff before trusting the green bar.**
- Keep tests **behaviour-focused**.
  → Coupling to implementation details turns a future refactor into a future rewrite.

---

## TDD vs SDD

- **Behaviour you can name as a function** — pure functions, parsers, formatters, validators:
  → lead with **TDD**

- **Behaviour that spans files, state, or UX**:
  → lead with **SDD**, do **TDD inside** each slice

A spec without tests is a wish.
Tests without a spec are local optima.
**Use both.**

---

## TDD — Like I'm five

> Order a sandwich and put a **picture of it on the table** before the kid is allowed in the kitchen.
>
> *"You're done when what's on the table matches the picture — ham, no mustard, crusts off."*
>
> The kid can't fudge it. Can't say *"good enough."* Can't bring back a hot dog.
>
> Either the sandwich matches the picture, or it doesn't.
>
> **The test is the picture.**

---

## Evaluations

> TDD gives you a test suite for your **code**. Evals give you a test suite for your **prompts**.

The problem: you change a prompt to fix one failure and it silently breaks three others.

Without evals → you find out in production.
With evals → you find out before merge.

---

## The minimal eval loop

1. **Collect examples.** 10–20 input/output pairs covering edge cases and known failures. Your **goldens**.
2. **Run them.** After any prompt change, compare outputs to goldens.
3. **Score.**
   - Exact-match outputs → automatic
   - Prose / reasoning → **model-as-judge**: *"is this correct? Reply yes/no + one-sentence reason."*
4. **Track over time.** Regression = score moves down. Improvement = score moves up.

Not a replacement for human review — a **regression harness**.

---

## Eval tooling

- **promptfoo** — open source, YAML-configured, runs locally and in CI
- **LangSmith / Braintrust** — hosted platforms with tracing
- **Roll your own** — a folder of `{input, expected}` pairs and a comparison script

---

## Evals vs TDD

|  | TDD | Evals |
|---|---|---|
| What's tested | Code correctness | Prompt/model behavior |
| Pass/fail signal | Deterministic | Often probabilistic |
| Written when | Before implementation | From observed behavior |
| Catches | Logic bugs | Prompt regressions |

**Use both.** TDD covers the code; evals cover the AI.

---

## Evals — Like I'm five

> **TDD** is one teacher checking today's homework.
>
> **Evals are the end-of-term report card** — 20 questions you got right last term.
>
> When you change *how* you do math, you re-take all 20 to make sure you didn't accidentally forget how to add while learning to multiply.

---

## Plan-then-execute

The agent drafts a plan → you approve or edit → agent executes against the approved plan.

- **Claude Code** — *Plan mode* (Shift+Tab)
- **Cursor / Windsurf** — agent modes with planning steps
- **Generic chat** — *"Make a plan first. Wait for my approval before changing code."*

**Reach for it** when the task is multi-step but well-bounded.
**Skip it** for tiny edits — the plan overhead exceeds the work.

---

## The diagnose loop

For bugs — especially the gnarly intermittent kind:

1. **Reproduce.** A one-liner that fails consistently. *No repro, no bug.*
2. **Minimize.** Strip until removing one more line makes it pass.
3. **Hypothesize.** State the one explanation you'd bet on. Make it falsifiable.
4. **Instrument.** Logging, prints, breakpoints to falsify the hypothesis.
5. **Fix.** Only after the hypothesis survives instrumentation.
6. **Regression test.** Fails before the fix, passes after.

AI's role: hold the hypothesis, generate the minimal repro, write the regression test.

---

## Diagnose — the big anti-pattern

Asking the model to *"fix the bug"* without a repro.

You'll get a confident patch that addresses **a** bug — possibly not **yours**.

---

## Diagnose — Like I'm five

> Your bike makes a weird noise.
>
> **Detective way:** ride it slow, ride it fast, find the *exact* moment it squeaks. Take off one part at a time until it stops squeaking. *That* was the broken part.
>
> **Guess-and-replace way:** *"it's probably the wheel,"* swap the wheel, still squeaks, guess again, still squeaks.
>
> The detective wins every time.

---

## Prototype-first

For design questions:
- *"Should the state machine look like this or this?"*
- *"What should this dashboard feel like?"*

**Build a throwaway. Ship it nowhere.** Use it to extract the *real* requirements.

Two flavors:
- **State / logic prototype** — runnable terminal app or notebook
- **UI prototype** — several radically different mocks side by side

**The discipline:** agree it's throwaway *before* you start, or you'll quietly promote it to production.

---

## Critique / grill

The model is its own first reviewer.

- **Self-critique.** *"Now review your last answer. What's wrong? What did you assume?"*
- **Adversarial second pass.** Fresh chat (no context): *"Find the three biggest weaknesses in this design."*
- **Grilling.** Have the model interview *you* — push back on hand-wavy claims.

**Cheapest quality lever available.** Most AI failures are *"nobody asked it to look again."*

---

## Critique — Like I'm five

> You hand in your homework. The teacher says:
>
> *"Now read it out loud and tell me what's wrong with it."*
>
> You almost always catch something — a missing word, a wrong number.
>
> The first pass is the kid hurrying.
> The second pass is the kid actually paying attention.
>
> Models are the same.

---

## Vertical-slice breakdown

Break a plan into independent, ship-shaped slices — each end-to-end through the stack (*"tracer bullet"*).

- Each slice = a separate task or PR
- The first slice teaches you what the spec got wrong before you've written nine

Claude Code's `to-issues` skill formalises this — turns a plan into independently-grabbable issues.

---

## Structured output

When the output feeds **code** instead of a human reader, constrain the shape:

```json
{
  "bugs": [
    { "line": 23, "severity": "high",
      "description": "SQL injection via string concat" },
    { "line": 41, "severity": "low",
      "description": "unused import" }
  ]
}
```

- **Reach for** when output plugs into code: classification, extraction, pipelines
- **Skip** for exploratory conversation or drafting

Most frontier APIs support this natively.

---

## Structured output — Like I'm five

> Asking a friend for their phone number, you can get:
>
> *"oh yeah it's like six-oh-something, ends in eight, I think"*
>
> Or you can hand them a form with ten boxes:
>
> *"fill in the digits."*
>
> The form is **structured output**.
> You can dial it. You can't dial a story.

---

## Model and cost selection

Pick the lightest model that handles the job:

| Model | Reach for when |
|---|---|
| **Haiku** (fast, cheap) | Extraction, classification, yes/no, high-volume |
| **Sonnet** (workhorse) | Most coding tasks, multi-step reasoning, tool use |
| **Opus** (expensive) | Hard multi-step problems, frontier reasoning |

**Start with Sonnet.** Quality issue → try Opus. Cost/latency issue on a mechanical task → try Haiku.

---

## Prompt caching

The highest-leverage cost lever most teams overlook.

If you send the same large prefix on every call — an agent re-reading CLAUDE.md, a pipeline with a shared doc — the API caches the stable prefix.

**On repeated-context workloads, caching cuts costs 80–90%.**

> The cheapest token is the one you don't send.
> Before adding context, ask: *does this change the output?* If not, cut it.

---

## Picking a workflow — decision tree

- Small, well-defined? → **just ask**
- New feature / non-trivial change? → **SDD**
- Pure function-shaped behavior? → **TDD** inside the SDD slice
- A bug? → **Diagnose loop**
- Design unclear? → **Prototype**
- Artifact drafted but worried? → **Critique / grill**
- Big plan, multiple people? → **Vertical-slice breakdown**
- Output feeds code? → **Structured output**
- Prompt regressed? → **Evals**
- High-volume / latency-sensitive? → try **Haiku**

---

## A real feature, end-to-end

A real feature often runs:

**brief → spec (SDD) → plan (plan-mode) → slice (vertical) → TDD each slice → critique the whole thing before merge**

---

## Hands-on — Exercise 1

**Spec and build a todo list or kanban board with OpenSpec.**

1. Fresh folder. Install OpenSpec: `npx openspec@latest init`
2. Open in Claude Code. Tell the agent: *"Use OpenSpec to propose a change that introduces a todo list CLI."*
3. Agent creates a change proposal under `openspec/changes/` — **before** writing code
4. Read it. Push back on vague acceptance criteria
5. Approve → agent implements against the spec

The **artifact you care about is the spec file** as much as the working code.

---

## Hands-on — Exercise 2

**Add a feature with OpenSpec.**

Same project. Pick a new feature — tags, due dates, priority colours, drag-and-drop, search, archive.

1. Agent drafts a **new** OpenSpec change proposal first
2. **Do not let it write code** until the proposal exists
3. Review: testable acceptance criteria? Edge cases listed (empty state, conflicts, undo)?
4. Iterate, then green-light implementation

Notice what the spec process catches that a one-line *"add tags"* prompt would have missed.

---

## Recap

- **Constrain first, generate second, verify third** is the spine of every workflow.
- **SDD** — the spec is the contract. Code review becomes *"does this match the spec?"*
- **TDD with AI** — you write the test, the model can't bluff past it.
- **Evals** — a test suite for your prompts, not your code.
- **Structured output** + smart **model selection** + **prompt caching** = cheaper, more reliable pipelines.
- **Plan-then-execute, diagnose, prototype, critique, vertical-slice** — each is a different constraint.
- The skill is **matching the workflow to the shape of the work**.

---

## Next

**Lesson 3 — Terminal Agents & Sandboxing**

Hand a spec from this lesson to an actual agent and watch it run.
