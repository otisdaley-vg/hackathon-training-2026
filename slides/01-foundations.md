---
marp: true
theme: default
paginate: true
size: 16:9
header: 'AI for Developers — Lesson 1'
footer: 'Foundations'
style: |
  section { font-size: 24px; }
  h1 { color: #1f2937; }
  h2 { color: #2563eb; }
  code { background: #f3f4f6; padding: 2px 6px; border-radius: 4px; }
  table { font-size: 0.85em; }
  blockquote { border-left: 4px solid #2563eb; color: #4b5563; }
---

# Lesson 1 — Foundations

**Time:** ~30 min
**Goal:** Build accurate intuitions about what an LLM is, what it can and can't do, and how to talk to one effectively.

---

## Questions?

**Interject anytime** — please ask if anything's unclear.

**Drop them in the chat too** — I'll collect everything so we can build a shared `faq.md` together after the session, for everyone to reference later.

---

## Objectives

By the end of this lesson you will be able to:

- Explain in one sentence what an LLM is doing when it generates text
- Name the main frontier model families and the companies behind them
- Define **token**, **context window**, and **temperature**
- Identify three classes of task LLMs handle well and three where they fail
- Write a prompt with clear role, context, task, and constraints
- Describe what a **harness**, an **agent**, and a **skill** are
- Explain what an **AGENTS.md / CLAUDE.md** file is for

---

## What an LLM is

A large language model is a **next-token predictor**.

Given a sequence of tokens, it outputs a probability distribution over the next token, samples one, appends it, and repeats.

Everything else — *"reasoning"*, *"tool use"*, *"memory"* — is built on top of this loop.

---

## Consequences of being a token predictor

- No notion of truth, only of *"what tokens usually come next"* → confident hallucinations
- No persistent memory between calls — anything it should "know" must be in the current context
- Stochastic — the same prompt can give different outputs

---

## What an LLM is — Like I'm five

> The world's most obsessive game of *"what word comes next?"* with a kid who's read every book in the library.
>
> You say *"Once upon a..."* — they shout *"time!"*
> Not because they understand stories. Because that's the word that almost always follows.
>
> Then they guess the next word. And the next.
>
> The model isn't *thinking* — it's playing that game very, very fast.

---

## Top 5 LLMs (and who builds them)

| Model family | Company | Notes |
|---|---|---|
| **Claude** (Opus, Sonnet, Haiku) | **Anthropic** | Strong on coding & agentic work |
| **GPT** (GPT-5 and successors) | **OpenAI** | Broad general capability, big plugin ecosystem |
| **Gemini** (Pro, Flash) | **Google DeepMind** | Native multimodal, huge context windows |
| **Grok** | **xAI** | Real-time web/X data |
| **Llama** | **Meta** | Open-weights — download and run yourself |

Honorable mentions: **DeepSeek**, **Mistral**, **Qwen**.

---

## Tokens

Tokens are sub-word chunks (~4 chars / ~0.75 words on average in English).

- Pricing and rate limits are **per token**
- **Context window** is measured in tokens (200K–1M+ on modern frontier models)
- Long contexts degrade quality — **"lost in the middle"** is real

---

## Tokens — Like I'm five

> A token is a **snack-sized piece of a word**.
>
> *"Dog"* — one snack.
> *"Antidisestablishmentarianism"* — seven snacks.
>
> The model:
> - Gets paid by the snack
> - Can only fit so many in its lunchbox at a time (**the context window**)
> - Pays the most attention to the snacks at the **top** and **bottom** of the lunchbox
>
> Stuff in the middle gets glossed over.

---

## Sampling parameters

- **Temperature** (0–2): higher = more random / creative, lower = more deterministic
  - For code, lean low (0–0.3)
- **Top-p / top-k**: alternate ways to clip the sampling distribution
  - You rarely need to touch these

---

## Temperature — Like I'm five

> A kid choosing crayons.
>
> **Low temperature** — they reach for the same red, every time.
> Predictable. Repeatable.
>
> **High temperature** — they bounce around grabbing weird shades.
> Sometimes brilliant, sometimes mud.
>
> For code → predictable kid.
> For poetry → bouncy kid.

---

## What LLMs are good at

- Pattern-rich text transformation (translation, summarization, format conversion, boilerplate)
- Tasks where the answer is "in the training data" — common APIs, well-known algorithms, idiomatic code
- Holding a lot of vague context and producing a plausible synthesis

---

## What LLMs fail at

- Arithmetic and counting → use a tool/code instead
- Up-to-the-minute knowledge they weren't trained on
- Reasoning across many steps without external scaffolding
- Saying *"I don't know"* — they'd rather guess confidently

---

## Prompting — the 4 ingredients

A good prompt usually has:

1. **Role / framing** — *"You are a senior Rust engineer reviewing a PR."*
2. **Context** — the relevant code, data, or background. More signal, less noise.
3. **Task** — what you actually want done, as a verb.
4. **Constraints / format** — output shape, style, what to avoid.

---

## Two prompting techniques worth knowing

- **Few-shot** — include 1–3 input/output examples
  - Cheaper and more reliable than long instructions for format-y tasks
- **Chain-of-thought** — *"think step by step"*
  - Lets the model spend tokens on intermediate reasoning
  - Modern models often do this automatically

---

## Harness

A **harness** is the program that wraps an LLM and runs the show.

- Takes your input → sends to the model → manages context → executes tools → decides when to stop
- The LLM alone is just `text in → text out`
- The harness is everything around it that makes it usable

**You're never really talking to the raw model. You're talking *through* a harness.**

---

## Harnesses you'll meet

- **Claude.ai** (web) — chat harness
- **Claude Code** (CLI) — agentic coding harness
- **Cursor / Windsurf** (IDE) — editor harnesses
- **GitHub Copilot** — completion + chat/agent modes
- **Aider** — open-source git-diff coding harness
- **ChatGPT** (web/app) — chat with browsing & code interpreter
- **OpenCode / T3 Chat / T3 Code** — multi-model harnesses
- **Anthropic / OpenAI SDK** — write your own loop in ~30 lines

---

## Harness — Like I'm five

> Imagine the model is a brilliant kid locked in a soundproof room — you can only talk to them through a slot in the door.
>
> The **harness** is the person standing at the door: they pass notes in, read notes out, and decide whether the kid is allowed to ask for things.
>
> Same kid, same brain — but a strict door-keeper gives you a pen-pal; a helpful one gives you a kid who can actually get stuff done.

---

## Agents

An **agent** is an LLM running in a loop with tools.

Instead of just emitting text, the model can call functions:

- read a file
- run a shell command
- fetch a URL

…observe the result, and decide what to do next.

**Shape to hold in your head: LLM + tools + loop.**

---

## What an agent actually looks like

**The agent runtime** — a program that runs the loop:

```
/usr/local/bin/claude    # the binary you invoke
~/.claude/               # config, sessions, skills, sub-agents
```

**Sub-agents** — smaller workers the main agent can spawn:

```
~/.claude/agents/code-reviewer.md
```

---

## A sub-agent definition

```markdown
---
name: code-reviewer
description: Reviews a diff for bugs, style, and security.
  Use when the user asks for a code review.
tools: Read, Grep, Glob, Bash
---

You are a focused code reviewer. Read the diff, flag concrete
issues with file:line references, and prefer specific over
general comments.
```

---

## Where agents live

- The runtime binary (e.g. `claude`) — installed via `npm`, `brew`, or desktop app
- `~/.claude/agents/<name>.md` — your personal sub-agents
- `<project>/.claude/agents/<name>.md` — repo-scoped sub-agents (commit to share)

---

## Agent — Like I'm five

> You tell your big sister, *"I'm hungry, make me a sandwich."*
>
> She walks to the kitchen, opens the fridge, finds the bread, spreads the peanut butter, and brings it back.
>
> A regular chat model is the sister who **tells you the recipe**. An agent is the sister who **gets up and makes the sandwich**.

---

## Skills

A **skill** is a packaged, on-demand instruction set the model loads only when it's relevant.

A small folder containing:

- A short **description** — so the model knows *when* to use it
- **Instructions** for *how* to perform the task
- Optionally: examples, helper scripts, reference files

Skills teach a generalist model your team's specific patterns.

---

## What a skill actually looks like

```
~/.claude/skills/commit-messages/
└── SKILL.md
```

```markdown
---
name: commit-messages
description: Write commit messages in our team's conventional format.
  Use when the user asks to commit or amend code.
---

Format: `<type>(<scope>): <subject>` — type ∈ {feat, fix, chore,
refactor, docs}.

- Subject is imperative, ≤ 60 chars, no trailing period.
- Body (optional) wraps at 72 chars and explains *why*, not *what*.
- Footer references issues: `Refs: #123`.
```

---

## Where skills live

- `~/.claude/skills/<name>/` — your personal library (all projects on your machine)
- `<project>/.claude/skills/<name>/` — scoped to one repo (commit to share)
- Bundled with **plugins** — Claude Code ships some out of the box (`init`, `review`)

---

## Skill — Like I'm five

> A cook with a drawer full of recipe cards.
>
> They don't keep every recipe in their head — too much to remember at once.
>
> When someone orders pancakes, the cook pulls the pancake card and follows it.
> Soup comes next — they swap the card.
>
> Cards are **skills**: small, labeled, and only opened when the order matches.

---

## Project memory: AGENTS.md / CLAUDE.md

A markdown file at the **root of your repo** that the harness reads automatically and keeps in context for the whole session.

> The project's **standing orders**. Anything in it is "always loaded."
> Anything *not* in it has to be re-explained every conversation.

- **AGENTS.md** — emerging cross-vendor convention (Cursor, Codex, OpenCode, Aider…)
- **CLAUDE.md** — Claude-flavored, takes priority when present

---

## What goes in it

**Put in:**
- **How to run things** — build, test, lint, dev server commands
- **Layout** — *"frontend in `apps/web/`, API in `services/api/`"*
- **Conventions** — naming, file structure, commit format
- **Watch-outs** — *"migrations folder is append-only"*, *"don't edit `generated/`"*
- **Glossary** — internal terms

**Don't put in:**
- Anything that belongs in a skill (task-specific)
- Long reference material — link out instead
- Stale notes — treat it like code; prune it

---

## Project memory — Like I'm five

> A sticky note on the kitchen fridge:
>
> *"We're out of milk. Dishwasher is broken. Don't touch the green plate — it's Grandma's."*
>
> Anyone who walks in reads it. You don't re-explain the basics every time.
>
> A recipe card (skill) is for one dish; the fridge note is for **everything that's always true in this kitchen**.

---

## Agent vs Skill

|  | Agent | Skill |
|---|---|---|
| What it is | A runtime — model + tools + loop | A package of on-demand instructions |
| Question | *How do I take actions?* | *How do I do task X well?* |
| Without the other | Works, just less specialized | Inert — needs an agent to invoke |
| Authored by | Configuring tools, permissions, prompts | Writing description + instructions |

**Agents are about doing; skills are about knowing how to do specific things.**

---

## Walkthroughs

Short hands-on exercises — 2–5 min each.

Don't just read them — actually run them. The point is to *see* the behavior.

You only need a browser and **claude.ai** for most.

---

## 1. Same prompt, different answers

**Concept:** LLMs are stochastic.

1. Open claude.ai. New chat.
2. Send: *"Write a one-sentence opening line for a fantasy novel."*
3. Two more fresh chats — same prompt.

**Watch for:** three different openings.

**Takeaway:** *"I ran it once and it worked"* ≠ *"it'll always work."*

---

## 2. Context is everything

**Concept:** No memory between calls. Context *is* the agent.

**Get the sandbox — pick one:**

- **Docker:** `docker run -it --rm ghcr.io/otisdaley-vg/hackathon-foundation`
- **VS Code:** open `environments/foundation/` → *Reopen in Container*

1. Inside the container, run `claude`
2. Ask: *"What does this project do? Suggest one improvement."*
3. Response comes back as **Yoda the space pirate** — you didn't ask for that. The project's standing orders did.
4. Edit the project's `CLAUDE.md` to a different persona. Restart `claude`. Same question.

**Watch for:** same model, same harness, same prompt — radically different agent.

**Takeaway:** when an agent feels off, ask *"what context is it carrying — and what's it missing?"* before *"did I pick the wrong model?"*

---

## 3. See your tokens

**Concept:** Tokenization.

Open a tokenizer. Paste:
- *"I went to the store."*
- *"Antidisestablishmentarianism"*
- A random string like *"qwzx8f7kpvnm"*
- 5 lines of your own code

**Watch for:** common English ≈ 1 token/word. Rare strings explode. Code is denser than prose.

**Takeaway:** "200K context" means very different things for prose vs JSON vs minified code.

---

## 4. Lazy prompt vs structured prompt

**Concept:** Role / context / task / constraints.

Send *"Fix this code"* with a SQL-injection-vulnerable C# method.

Then send a **structured** version: role + context + task + constraints.

**Watch for:** the structured prompt zeroes in on the SQL injection. The lazy one often misses or buries it.

**Takeaway:** the same model acts senior or junior depending entirely on how you frame the ask.

---

## 5. Same brain, different hands

**Concept:** Harness ≠ model.

1. In claude.ai: *"List the files in my current working directory."*
2. In a terminal — run `claude` — ask the same thing.

**Watch for:** identical model, wildly different capability.

**Takeaway:** when you pick "an AI tool," you're picking a **harness** as much as a model.

---

## 6. Read someone's project memory

**Concept:** AGENTS.md / CLAUDE.md.

1. Open a real repo with one: `anthropics/claude-code`, `anthropics/anthropic-cookbook`
2. Read it. Ask: *what conversation would I have had to have, repeatedly, if this didn't exist?*
3. Write a 10-line CLAUDE.md for one of your own projects:
   - One command
   - One layout fact
   - One watch-out

---

## 7. Pick the right tool — decision framework

For any request, ask three questions in order:

1. **Does this need to *touch* anything?** → if **yes**, you need an **agent**
2. **Repeating task with a specific recipe?** → write a **skill**
3. **Just knowledge or one-off transformation?** → **chat** is fine

**Short version: act → agent. repeat → skill. answer → chat.**

---

## Quiz — pick agent, skill, or chat

1. *"Explain what a Kalman filter does."*
2. *"Find every file in this repo that imports `lodash` and replace `_.cloneDeep` with `structuredClone`."*
3. *"Format every commit message I write to follow our team's convention."*
4. *"What's a good name for my band?"*
5. *"Run the test suite, find the failing test, propose a fix, and open a PR."*
6. *"Translate this paragraph into French."*

---

## Quiz — answers

1. **Chat** — knowledge lookup
2. **Agent** — read repo, grep, edit, typecheck
3. **Skill** — repeating recipe, specific format
4. **Chat** — pure ideation
5. **Agent** — multi-step, must observe output
6. **Chat** — one-shot transformation

**Pattern:** *find / edit / run / open / deploy* → agent.
*explain / summarize / translate / name / draft* → chat.

---

## Recap

- **LLMs predict tokens.** Everything else is scaffolding.
- The frontier is a handful of model families: **Claude, GPT, Gemini, Grok, Llama**.
- **Context is finite and noisy contexts hurt.** Be deliberate.
- **Good prompts** have role, context, task, constraints — in that order.
- **The model will lie confidently.** Verify anything load-bearing.
- **Harness** = how you talk to the model. **Agent** = LLM + tools + loop.
  **Skill** = on-demand instructions. **CLAUDE.md** = always-loaded standing orders.

---

## Next

**Lesson 2 — AI Workflows**

The disciplined patterns — spec-driven development, TDD, plan-then-execute — that decide whether your AI tools produce real engineering or expensive autocomplete.
