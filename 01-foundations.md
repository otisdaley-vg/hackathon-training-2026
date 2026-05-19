# Lesson 1 — Foundations

**Time:** ~30 min
**Goal:** Build accurate intuitions about what an LLM is, what it can and can't do, and how to talk to one effectively.

## Objectives

By the end of this lesson you will be able to:

- Explain in one sentence what an LLM is doing when it generates text.
- Name the main frontier model families and the companies behind them.
- Define **token**, **context window**, and **temperature** and say why each matters in practice.
- Identify three classes of task LLMs handle well and three where they fail.
- Write a prompt with clear role, context, task, and constraints.
- Describe what a **harness**, an **agent**, and a **skill** are, and say which one you reach for when.
- Explain what an **AGENTS.md / CLAUDE.md** file is for and what belongs in one.

## Concepts

### What an LLM is

A large language model is a next-token predictor. Given a sequence of tokens, it outputs a probability distribution over the next token, samples one, appends it, and repeats. Everything else — "reasoning", "tool use", "memory" — is built on top of this loop.

This has consequences:

- The model has no notion of truth, only of "what tokens usually come next." It will produce confident-sounding wrong answers (hallucinations) when the right answer isn't well-represented in its training.
- It has no persistent memory between calls. Anything you want it to "know" must be in the current context.
- It's stochastic. The same prompt can give different outputs.

> **Like I'm five:** Imagine the world's most obsessive game of *"what word comes next?"* with a kid who's read every book in the library. You say *"Once upon a..."* and they shout *"time!"* — not because they understand stories, but because that's the word that almost always follows. Then they guess the next word, and the next, one at a time, until a whole story spills out. The model isn't *thinking*; it's playing that game very, very fast.

### Top 5 LLMs (and who builds them)

The frontier shifts every few months — by the time you read this, version numbers will be off. The companies stay roughly the same:

| Model family | Company | Notes |
| --- | --- | --- |
| **Claude** (Opus, Sonnet, Haiku) | **Anthropic** | Strong on coding, agentic work, and nuanced instruction-following. Opus is the heavyweight, Sonnet the workhorse, Haiku the fast/cheap option. |
| **GPT** (GPT-5 and successors) | **OpenAI** | Broad general capability, large plugin/tools ecosystem, tight ChatGPT integration. |
| **Gemini** (Pro, Flash) | **Google DeepMind** | Native multimodality, massive context windows, deep Google Workspace integration. |
| **Grok** | **xAI** | Real-time web/X data, favored for fresh-info queries. |
| **Llama** | **Meta** | Open-weights — you can download and run it yourself. The dominant open model family. |

Honorable mentions: **DeepSeek** (open-weights, very strong reasoning), **Mistral** (European open-weights), **Qwen** (Alibaba, open-weights).

For coding-agent work, Claude and GPT lead. For self-hosted or offline use, look at Llama, DeepSeek, or Qwen.

### Tokens

Tokens are sub-word chunks (~4 chars / ~0.75 words on average in English). They matter because:

- Pricing and rate limits are per token.
- The **context window** is measured in tokens — the total input + output the model can see at once. Modern frontier models range from 200K to 1M+ tokens.
- Long contexts degrade quality. "Lost in the middle" is real: information buried halfway through a giant prompt gets ignored more often than information at the start or end.

> **Like I'm five:** A token is a snack-sized piece of a word. *"Dog"* is one snack. *"Antidisestablishmentarianism"* is seven snacks. The model gets paid by the snack, can only fit so many snacks in its lunchbox (**the context window**) at a time, and — weirdly — pays the most attention to the snacks at the *top* and *bottom* of the lunchbox. Stuff in the middle gets glossed over.

### Sampling parameters

- **Temperature** (0–2): higher = more random / creative, lower = more deterministic. For code, lean low (0–0.3).
- **Top-p / top-k**: alternate ways to clip the sampling distribution. You rarely need to touch these.

> **Like I'm five:** Imagine a kid choosing crayons. **Low temperature** — they reach for the same red, every time. Predictable. **High temperature** — they bounce around grabbing weird shades, sometimes brilliant, sometimes mud. For code you want the predictable kid. For poetry you want the bouncy one.

### Capabilities and limits

LLMs are good at:

- Pattern-rich text transformation (translation, summarization, format conversion, boilerplate).
- Tasks where the answer is "in the training data" — common APIs, well-known algorithms, idiomatic code.
- Holding a lot of vague context and producing a plausible synthesis.

LLMs fail at:

- Arithmetic and counting (use a tool/code instead).
- Anything requiring up-to-the-minute knowledge they weren't trained on. (ish** prebaked Skills are used to fetch up to date data)
- Reasoning that requires keeping rigorous state across many steps without external scaffolding.
- Saying "I don't know." They'd rather guess confidently. (ish** pre GPT 5.4 & Opus 4.6)

### Prompting

A good prompt usually has:

1. **Role / framing** — "You are a senior Rust engineer reviewing a PR."
2. **Context** — the relevant code, data, or background. More signal, less noise.
3. **Task** — what you actually want done, as a verb.
4. **Constraints / format** — output shape, style, what to avoid.

Two techniques worth knowing:

- **Few-shot**: include 1–3 input/output examples. Cheaper and more reliable than long instructions for format-y tasks.
- **Chain-of-thought / "think step by step"**: lets the model spend tokens on intermediate reasoning before answering. Modern models often do this automatically (or via an explicit "thinking" mode).

### Harness

A **harness** is the program that wraps an LLM and runs the show — it takes your input, sends it to the model, manages context, executes any tools the model wants to call, and decides when to stop. The LLM by itself is just a function: `text in → text out`. The harness is everything around it that makes it usable.

Put another way: when you "use Claude," you're never really talking to the raw model. You're talking *through* a harness. The same model can feel very different depending on which one you use — Claude Sonnet in a browser can only write text back to you; Claude Sonnet in a coding harness can read your repo, run your tests, and open a PR.

Examples you'll meet:

1. **Claude.ai** (web) — chat harness. Browser UI, conversation history, file uploads, no shell access by default.
2. **Claude Code** (CLI) — agentic coding harness. Runs in your terminal with tools for reading files, editing, running bash, spawning sub-agents.
3. **Cursor** (IDE) — editor harness. Wraps multiple models and integrates deeply with your codebase via the editor.
4. **Windsurf** (IDE) — LLM-native editor similar to Cursor, with an agent mode.
5. **GitHub Copilot** — completion harness embedded in your editor; the newer Chat / Agent modes are fuller harnesses.
6. **Aider** (CLI) — open-source coding harness that edits files via git diffs.
7. **ChatGPT** (web/app) — OpenAI's chat harness with browsing, code interpreter, and custom GPTs.
8. **OpenCode** (SST) — open-source terminal coding agent, model-agnostic; a Claude Code-style harness you can point at any provider.
9. **T3 Chat** (t3.chat) — fast multi-model chat harness; pick between Claude, GPT, Gemini, Grok, etc. from one UI.
10. **T3 Code** — coding-focused harness from the T3 team, same multi-model philosophy applied to agentic dev work.
11. **Anthropic / OpenAI SDK in your own script** — the minimal harness. You write the loop yourself in ~30 lines of Python.

The harness is the *thing you choose*. The model is the *brain you plug into it*. Most of this track is about getting fluent in one specific harness — Claude Code — but the concepts transfer.

> **Like I'm five:** Imagine the model is a brilliant kid locked in a soundproof room — you can only talk to them through a slot in the door. The harness is the person standing at the door: they pass notes in, read notes out, and decide whether the kid is allowed to ask for things (like "fetch me a book" or "run downstairs and check the oven"). Same kid, same brain — but a strict door-keeper gives you a pen-pal; a helpful one gives you a kid who can actually get stuff done.

### Agents

An **agent** is an LLM running in a loop with tools. Instead of just emitting text, the model can call functions — read a file, run a shell command, fetch a URL — observe the result, and decide what to do next. The loop continues until the task is done or it hits a guardrail.

Why it matters: a plain chat model can only output tokens. An agent can change the world. That shift — from "give me an answer" to "make this happen" — is what makes modern coding assistants feel different from a search box.

The shape to hold in your head: **LLM + tools + loop**. Lesson 3 goes deep on running one.

**What an agent actually looks like.** Two flavors you'll meet:

1. **The agent runtime** — a program that runs the loop. Claude Code is a CLI you launch in a terminal:

   ```
   /usr/local/bin/claude    # the binary you invoke
   ~/.claude/               # its config, sessions, skills, sub-agents
   ```

   When you run `claude`, that program *is* the loop: it asks the model what to do, calls tools (read, edit, bash, etc.), observes the output, and repeats.

2. **Sub-agents** — smaller workers the main agent can spawn for focused jobs (code review, research, test running). Same file-shape as skills, but the markdown describes a *worker* with its own tools and prompt:

   ```
   ~/.claude/agents/code-reviewer.md
   ```

   And the file itself:

   ```markdown
   ---
   name: code-reviewer
   description: Reviews a diff for bugs, style, and security. Use when the user asks for a code review.
   tools: Read, Grep, Glob, Bash
   ---

   You are a focused code reviewer. Read the diff, flag concrete issues
   with file:line references, and prefer specific over general comments.
   ```

**Where agents live on your machine:**

- The runtime binary (e.g. `claude`) — installed via `npm`, `brew`, or a downloaded desktop app.
- `~/.claude/agents/<name>.md` — your personal sub-agents, available in every project.
- `<project>/.claude/agents/<name>.md` — sub-agents scoped to one repo; commit them to share with the team.

> **Like I'm five:** You tell your big sister, "I'm hungry, make me a sandwich." She walks to the kitchen, opens the fridge, finds the bread, spreads the peanut butter, and brings it back. You didn't tell her every step — you said what you wanted, and she did the steps. That's an agent. A regular chat model is the sister who *tells you the recipe*. An agent is the sister who *gets up and makes the sandwich*.

### Skills

A **skill** is a packaged, on-demand instruction set the model loads only when it's relevant. Think of it as a small folder containing:

- A short description, so the model knows *when* to use it.
- Instructions for *how* to perform the task.
- Optionally: example inputs/outputs, helper scripts, reference files.

The model pulls a skill into context only when the current task matches its description. That keeps the base prompt small — instead of stuffing every possible instruction into every conversation, you have a library of skills and the model fetches what it needs.

Skills are how you teach a generalist model your team's specific patterns: *how we write commit messages*, *how we run the test suite*, *how we structure a migration*.

**What a skill actually looks like.** A skill is just a folder with a `SKILL.md` inside (plus any extras it needs — example inputs, helper scripts, reference docs). Minimal layout:

```
~/.claude/skills/commit-messages/
└── SKILL.md
```

And `SKILL.md` itself is a markdown file with a small YAML header — the description is what the model reads to decide whether to load the skill:

```markdown
---
name: commit-messages
description: Write commit messages in our team's conventional format. Use when the user asks to commit or amend code.
---

Format: `<type>(<scope>): <subject>` — type ∈ {feat, fix, chore, refactor, docs}.

- Subject is imperative, ≤ 60 chars, no trailing period.
- Body (optional) wraps at 72 chars and explains *why*, not *what*.
- Footer references issues: `Refs: #123`.
```

**Where skills live on your machine:**

- `~/.claude/skills/<name>/` — your personal library, available in every project on your machine.
- `<project>/.claude/skills/<name>/` — scoped to one repo. Commit it and teammates get it too.
- Bundled with **plugins** — Claude Code ships some skills out of the box (e.g. `init`, `review`), and installing a plugin can add more.

> **Like I'm five:** Imagine a cook with a drawer full of recipe cards. They don't keep every recipe in their head — that's too much to remember at once. When someone orders pancakes, the cook pulls out the pancake card and follows it. When someone orders soup, they put the pancake card back and grab the soup card. The cards are skills: small, labeled, and only opened when the order matches.

### Project memory (AGENTS.md / CLAUDE.md)

Skills are pulled in *on demand*. But every project has a baseline of stuff the agent should know from the moment it starts: how to run the tests, which folder is the frontend, what not to touch, the team's coding style. That's what an **AGENTS.md** or **CLAUDE.md** file is for — a markdown file at the root of your repo that the harness reads automatically and keeps in context for the whole session.

Think of it as the project's standing orders. Anything in it is "always loaded." Anything *not* in it has to be re-explained every conversation.

The two names are the same idea from different vendors:

- **AGENTS.md** — the emerging cross-vendor convention. Cursor, Codex, OpenCode, Aider and others look for it. If you want one file that works everywhere, use this.
- **CLAUDE.md** — Claude Code's flavor of the same thing. Slightly Claude-specific, takes priority when present.

What a good one contains:

- **How to run things** — build, test, lint, dev server commands.
- **Layout** — "frontend is in `apps/web/`, API in `services/api/`, shared types in `packages/types/`."
- **Conventions** — naming, file structure, commit format, what gets a test and what doesn't.
- **Watch-outs** — "the migrations folder is append-only," "don't edit `generated/`," "auth lives in `lib/auth.ts` and is load-bearing."
- **Glossary** — internal terms a newcomer would have to ask about.

What *not* to put in it:

- Anything that belongs in a skill (task-specific instructions only relevant sometimes).
- Long reference material — link out instead. Every line in this file costs context on every turn.
- Stale notes. This file rots; treat it like code and prune it.

**Where it lives:**

- `<project>/AGENTS.md` or `<project>/CLAUDE.md` — repo-wide, committed, shared with the team.
- `~/.claude/CLAUDE.md` — your personal global instructions, applied to every project you open. Good for "I prefer pnpm," "always use ripgrep over grep," "I'm colorblind, no red/green diffs."

> **Like I'm five:** Imagine a sticky note on the kitchen fridge: *"We're out of milk. Dishwasher is broken. Don't touch the green plate — it's Grandma's."* Anyone who walks into the kitchen — your sister, your dad, the babysitter — reads it the second they arrive, so you don't have to explain the same things over and over. The fridge note is the project's standing rules. A recipe card (skill) is for one specific dish; the fridge note is for *everything that's always true in this kitchen*.

### Agent vs skill

These get confused because both live around the model, but they answer different questions:

|  | Agent | Skill |
| --- | --- | --- |
| What it is | A runtime — model + tools + loop | A package of instructions the model loads on demand |
| Question it answers | *How do I take actions?* | *How do I do task X well?* |
| Without the other | Still works, just less specialized | Inert — needs an agent (or chat session) to invoke it |
| You author it by | Configuring tools, permissions, prompts | Writing a description + instructions |

Short version: **agents are about doing; skills are about knowing how to do specific things**. A capable agent with good skills beats a capable agent without them, the same way a senior engineer with a runbook beats one without.

> **Like I'm five:** The agent is the cook. The skill is a recipe card. A cook with no cards can still cook from memory — fine for everyday stuff, shaky on weird dishes. A card with no cook is just a piece of paper; it can't cook anything by itself. You need the cook to *do* anything at all. You add cards to make the cook reliably great at specific things.


## Walkthroughs

Short hands-on exercises. Each takes 2–5 minutes and makes one concept concrete. Don't just read them — actually run them. The whole point is to *see* the behavior, not take my word for it.

You only need a browser and [claude.ai](https://claude.ai) for most of these. A couple are better with a terminal and Claude Code installed, but those are marked.

#### 1. Same prompt, different answers

**Concept:** LLMs are stochastic.

1. Open [claude.ai](https://claude.ai). Start a new chat.
2. Send: *"Write a one-sentence opening line for a fantasy novel."*
3. Start *another* new chat (not a reply — fully fresh). Send the exact same prompt.
4. Do it once more.

**Watch for:** three different openings. The model didn't "decide" once; each call sampled a fresh path through the probability distribution.

**Takeaway:** never assume "I ran it once and it worked" means it'll always work. If reliability matters, test repeatedly — or lower temperature and constrain the output shape.

#### 2. Context is everything

**Concept:** The model has no persistent memory. Whatever's in its context *is* its world. Change the context, change the agent — same binary, same model, same harness.

The fastest way to feel this is to drop into a project that has a deliberately weird `CLAUDE.md` and watch the agent inherit a personality you never asked for. We've prepared one for you: **`environments/foundation/`** — a minimal TypeScript project with a `CLAUDE.md` at the root that tells Claude it's *"Yoda the space pirate."*

*(Requires Docker. Pick **one** of the two paths below.)*

**Path A — Docker pull (fastest):**

```bash
docker pull ghcr.io/otisdaley-vg/hackathon-foundation
docker run -it --rm ghcr.io/otisdaley-vg/hackathon-foundation
```

You land in `/workspace` inside the container with `claude` already installed and the foundation project's `CLAUDE.md` already on disk.

**Path B — VS Code devcontainer:**

Open `environments/foundation/` in VS Code and click **"Reopen in Container"** when prompted. The build uses the pre-pulled image as a cache, so it's near-instant after the first run.

---

1. Once you're inside the container, run `claude`. Confirm the model says **Claude Opus 4.7** (or whatever the latest is on `/model`).
2. Ask something completely ordinary:

   > What does this project do? Suggest one improvement.

3. Read the response. It will come back in *pirate-Yoda voice* — *"Hmm, a hello-world this is, matey…"* The model wasn't fine-tuned. You didn't pick a persona. The project's standing orders did it, silently, on turn one.
4. Now open the `CLAUDE.md` in the project root and replace the line with something completely different — for example:

   ```
   You are a curmudgeonly 1990s sysadmin who is suspicious of JavaScript
   and answers in two sentences max.
   ```

5. Type `/exit` to end the session, then run `claude` again from the same folder. Ask the exact same question.
6. Same model, same harness, same prompt — completely different personality, length, and vocabulary.

**Watch for:** how *thoroughly* the agent inhabits whatever you put in front of it. It's not just tone — it changes what it volunteers, what it omits, what it considers worth flagging. You didn't ask for any of that explicitly.

**Takeaway:** when an agent feels dumb, snarky, lazy, or off-target, *your first instinct should not be "I picked the wrong model."* It should be **"what context is it carrying — and what context is it missing?"** Most of the day-to-day leverage in working with LLMs is in shaping context: project memory, the prompt, the files you put in front of it, the order you put them in. The model is a constant. The context is the variable.

> **Aside — same lesson, different failure mode.** Earlier generations of models would happily invent five peer-reviewed papers, complete with DOIs that resolve to *different* papers, when asked for a literature review they couldn't actually do. Frontier models in 2026 mostly refuse or hedge. The underlying cause is the same — *the model has no notion of truth, only of what tokens usually come next* — but the failure surface has moved. Verifying load-bearing details (citations, API signatures, numeric facts) is still your job, not the model's.

#### 3. See your tokens

**Concept:** Tokenization and why prompt length costs you.

1. Open a tokenizer in your browser ([Anthropic's tokenizer](https://www.anthropic.com/tokenizer) or [OpenAI's](https://platform.openai.com/tokenizer)).
2. Paste *"I went to the store."* — note the token count.
3. Paste *"Antidisestablishmentarianism"* — note it.
4. Paste a random string like *"qwzx8f7kpvnm"* — note it.
5. Paste a 5-line snippet of your own code.

**Watch for:** common English ≈ 1 token per word. Rare words and random strings explode into many tokens. Code is denser than prose because punctuation and whitespace each take tokens.

**Takeaway:** "the context window is 200k tokens" means very different things for English prose vs. JSON vs. minified code. Cost and "lost in the middle" both scale with this.

#### 4. Lazy prompt vs. structured prompt

**Concept:** Role / context / task / constraints.

1. New chat. Send: *"Fix this code"* — and paste this snippet:

   ```csharp
   public List<int> GetUserIds(SqlConnection conn, string status)
   {
       using var cmd = conn.CreateCommand();
       cmd.CommandText = "SELECT id FROM users WHERE status = '" + status + "'";
       using var reader = cmd.ExecuteReader();
       var ids = new List<int>();
       while (reader.Read()) ids.Add(reader.GetInt32(0));
       return ids;
   }
   ```

2. Read the answer. It probably fixes *something* but may scope-creep, rewrite too much, or miss the actual issue.
3. New chat. Send a structured version:

   > You are a senior C# engineer reviewing a teammate's PR.
   >
   > Context: this method pulls user IDs for a status filter passed in from an ASP.NET controller. It's used in our public API.
   >
   > ```csharp
   > public List<int> GetUserIds(SqlConnection conn, string status)
   > {
   >     using var cmd = conn.CreateCommand();
   >     cmd.CommandText = "SELECT id FROM users WHERE status = '" + status + "'";
   >     using var reader = cmd.ExecuteReader();
   >     var ids = new List<int>();
   >     while (reader.Read()) ids.Add(reader.GetInt32(0));
   >     return ids;
   > }
   > ```
   >
   > Task: identify the single most important bug and propose the smallest fix.
   > Constraints: don't rewrite the method. Don't add logging or error handling. Show only the changed line(s).

4. Compare the two responses side by side.

**Watch for:** the structured prompt zeroes in on the SQL injection (status is user-controlled, string-concatenated into SQL). The lazy prompt often misses it or buries it.

**Takeaway:** the same model can act senior or junior depending entirely on how you frame the ask.

#### 5. Same brain, different hands

**Concept:** Harness ≠ model. *(Requires Claude Code installed, or skip and watch a demo.)*

1. In [claude.ai](https://claude.ai), ask: *"List the files in my current working directory."*
2. Note the response — it'll explain it can't, or guess, or ask you to paste a listing.
3. Open a terminal in any folder. Run `claude`. Ask the same thing.
4. Note: Claude Code calls `bash`, runs `ls`, and answers with the real listing.

**Watch for:** identical model. Wildly different capability. The chat harness has no shell tool; the coding harness does.

**Takeaway:** when you pick "an AI tool," you're picking a harness as much as a model. The model is the brain. The harness decides what hands it has.

#### 6. Read someone's project memory

**Concept:** AGENTS.md / CLAUDE.md.

1. Open a real repo that has one. Good examples: [`anthropics/claude-code`](https://github.com/anthropics/claude-code), [`anthropics/anthropic-cookbook`](https://github.com/anthropics/anthropic-cookbook), or any large open-source project that's been updated recently. Search for `CLAUDE.md` or `AGENTS.md` at the root.
2. Read it top to bottom. Ask yourself for each section: *what conversation would I have had to have, repeatedly, if this file didn't exist?*
3. Now open one of *your own* projects. In 5 minutes, write a 10-line `CLAUDE.md`. Include:
   - One command (how to run tests or the dev server).
   - One layout fact (where the main code lives).
   - One watch-out (something you'd warn a new contributor about).
4. Save it at the repo root.

**Takeaway:** this file is where you stop re-briefing. Every line you put here is a sentence you don't have to type again.

#### 7. Pick the right tool for the job

**Concept:** Agent vs. skill vs. just-ask.

Before the quiz, here's the decision framework. For any request, ask three questions in order:

1. **Does this need to *touch* anything?** (read files, run commands, hit the network, edit code) → if **yes**, you need an **agent**. If **no**, keep going.
2. **Is this a repeating task with a specific recipe my team follows?** (commits, migrations, PR descriptions, code-review style) → if **yes**, write a **skill** so the recipe loads automatically when it's relevant. If **no**, keep going.
3. **Is it just knowledge or a one-off transformation?** → **chat** is fine.

In short: **act → agent. repeat → skill. answer → chat.**

**Worked example — let's do one together.** *"Summarize this PDF I'm pasting in."*

- Does it need to touch anything? No — the PDF is right there in the message. ❌
- Is it a repeating recipe? No — just a one-off summary. ❌
- Is it a one-off transformation? Yes. ✅ → **Chat.**

Now your turn. For each request below, run it through the three questions and commit to an answer *before* opening the spoiler.

1. *"Explain what a Kalman filter does."*
2. *"Find every file in this repo that imports `lodash` and replace `_.cloneDeep` with `structuredClone` where safe."*
3. *"Format every commit message I write to follow our team's convention."*
4. *"What's a good name for my band?"*
5. *"Run the test suite, find the failing test, propose a fix, and open a PR."*
6. *"Translate this paragraph into French."*

<details>
<summary>Answers (with reasoning)</summary>

1. **Chat.** Touch anything? No. Repeating recipe? No. Just knowledge → chat.
2. **Agent.** Touch anything? Yes — it has to read every file in the repo, grep, edit, and ideally re-run typecheck. That's a loop with tools.
3. **Skill.** Touch anything? Only when committing — and only *sometimes*. Repeating recipe with a specific format? Yes. Lives at `~/.claude/skills/commit-messages/` and the agent pulls it in when the task matches.
4. **Chat.** Touch anything? No. Repeating recipe? No. Pure ideation → chat.
5. **Agent.** Touch anything? Yes — runs `npm test`, reads source, edits files, runs `git`, calls `gh`. Multi-step, stateful, must observe output before deciding the next step. Textbook agent loop.
6. **Chat.** Touch anything? No. Repeating recipe? No. One-shot transformation → chat.

**Pattern to notice:** anything with the verbs *find / edit / run / open / deploy* almost always needs an agent. Anything with *explain / summarize / translate / name / draft* is usually chat. Skills sit in between — they're for the "we always do X *this way*" tasks that recur.

</details>

## Recap

- LLMs predict tokens. Everything else is scaffolding.
- The frontier is a handful of model families from a handful of companies — Claude (Anthropic), GPT (OpenAI), Gemini (Google), Grok (xAI), Llama (Meta).
- Context is finite and noisy contexts hurt quality. Be deliberate about what you put in.
- Good prompts have role, context, task, and constraints — in that order.
- The model will lie to you confidently. Build the habit of verifying anything load-bearing.
- A **harness** is the program you talk to the model through. An **agent** wraps the model in a loop with tools so it can act. A **skill** packages task-specific know-how the model loads on demand. An **AGENTS.md / CLAUDE.md** holds the standing rules of a project so the agent doesn't have to be re-briefed every session. You'll use all of these in the rest of the track.

**Next:** [Lesson 2 — AI Workflows](./02-claude.md), where we cover the disciplined patterns — spec-driven development, TDD, and friends — that decide whether your AI tools produce real engineering or expensive autocomplete.
