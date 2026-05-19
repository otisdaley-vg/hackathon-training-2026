---
marp: true
theme: default
paginate: true
size: 16:9
header: 'AI for Developers — Lesson 4'
footer: 'MCPs'
style: |
  section { font-size: 24px; }
  h1 { color: #1f2937; }
  h2 { color: #2563eb; }
  code { background: #f3f4f6; padding: 2px 6px; border-radius: 4px; }
  table { font-size: 0.85em; }
  blockquote { border-left: 4px solid #2563eb; color: #4b5563; }
---

# Lesson 4 — MCPs

**Time:** ~30 min

**Goal:** Understand the Model Context Protocol (MCP), why it exists, and how to wire an MCP server into your agent so it can talk to external systems.

---

## Questions?

**Interject anytime** — please ask if anything's unclear.

**Drop them in the chat too** — I'll collect everything so we can build a shared `faq.md` together after the session, for everyone to reference later.

---

## Objectives

By the end of this lesson you will be able to:

- Explain what MCP is and the problem it solves
- Distinguish a **tool**, a **resource**, and a **prompt** in MCP terms
- Install and configure an existing MCP server in Claude Code
- Sketch the shape of an MCP server you'd build for your own team

---

## Why MCP exists

A terminal agent with `bash` and `Edit` is already powerful.

But most real work lives **outside your filesystem**:
- Tickets in Jira
- Designs in Figma
- Data in BigQuery
- Errors in Sentry
- Runbooks in Notion

Without a way in, the agent can only ask you to paste things — it can advise, not act.

---

## The N × M problem

**Before MCP:**

Every agent vendor wrote their own integrations.
Every tool vendor wrote their own agent plugin.

→ **N × M** maintenance.

**MCP** (Model Context Protocol, open spec from Anthropic, late 2024):

One protocol. Any compliant agent talks to any compliant server.

→ **N + M**.

---

## MCP — Like I'm five

> Before USB-C, every gadget needed its own cable —
> phone cable, camera cable, weird-printer cable.
>
> A drawer full of them, and nothing fit anything.
>
> **MCP is the USB-C of agent tools.**
>
> One plug shape. Everything fits.

---

## The mental model

An **MCP server** is a small process — usually a local subprocess, sometimes a remote service — that exposes capabilities to an agent over a **JSON-RPC connection**.

The **agent is the client**.

---

## What a server exposes

A server can expose three kinds of things:

- **Tools** — functions the agent can call
  e.g. `create_jira_ticket(title, body)`, `query_bigquery(sql)`, `send_slack(channel, text)`
- **Resources** — read-only data the agent fetches by URI
  e.g. `linear://issue/ENG-123`, `file:///path/to/log`
  Think *"GET endpoints"*
- **Prompts** — reusable prompt templates the server offers
  Less common; useful for vendor-curated workflows

> Most servers in the wild are mostly **tools**.

---

## Tools / resources / prompts — Like I'm five

> A kitchen drawer.
>
> **Tools** are gadgets that *do* things — the can-opener, the whisk.
>
> **Resources** are things you *read* — the cookbook, the label on the jar.
>
> **Prompts** are post-it notes saying *"how Grandma makes the stew."*
>
> You'll mostly grab tools; the others are there if you need them.

---

## Transport

Two common transports:

- **stdio** — the server is launched as a subprocess by the agent; communication over stdin/stdout
  → default for local tools
- **HTTP / SSE** — remote servers
  → useful when the server holds creds you don't want on every laptop, or wraps a SaaS

You rarely care about transport in practice — you just paste a config block.

---

## What the agent actually sees

When the agent connects to a server, it asks *"what can you do?"*
The server replies with a list of tool names, descriptions, and JSON schemas.

From the agent's perspective, **MCP tools look identical to its built-in tools.**

It picks the right one based on the description and the user's request.

→ **Tool descriptions are load-bearing.**

If you build a server, write descriptions like docstrings aimed at the model:
*when should this be used, what does it return, what are the gotchas.*

---

## Trust and risk

Same blast-radius thinking as Lesson 3, but **bigger**:

- An MCP server can read your filesystem, hit your APIs, **spend money**.
  Only install servers you **trust**.
- A malicious or buggy server can confuse the agent — **"tool injection"** is a real attack surface.
  Treat tool output as untrusted input.
- For shared/remote servers, **audit the auth model**.
  Who can call this? What can they do?

---

## Trust & risk — Like I'm five

> Letting a new toy into your house.
>
> The toy might be lovely.
>
> The toy might also be a tiny robot with sharp edges that walks into the kitchen and unplugs the fridge.
>
> Once you plug it in, it can do things.
>
> So check what it does **before** you plug it in. Not after.

---

## Hands-on

We'll wire Claude Code to a real database via MCP and let it write the SQL.

Adapted from [ai-training.re-cinq.com/mcp-db](https://ai-training.re-cinq.com/mcp-db/).

---

## Exercise 1 — Stand up the Netflix DB

Pull and run the pre-baked container (pick one):

```bash
# MariaDB / MySQL
docker pull ghcr.io/re-cinq/ai-training-netflixdb-mariadb:latest
docker run -d --name netflixdb -p 3306:3306 \
  ghcr.io/re-cinq/ai-training-netflixdb-mariadb

# OR Postgres
docker pull ghcr.io/re-cinq/ai-training-netflixdb-postgres:latest
docker run -d --name netflixdb -p 5432:5432 \
  ghcr.io/re-cinq/ai-training-netflixdb-postgres
```

Confirm with `docker ps`.

→ 4 tables (`movie`, `tv_show`, `season`, `view_summary`), ~37K rows.

---

## Exercise 2 — Where does the agent run?

Two options, same outcome:

- **Agent on host** — your existing Claude Code talks to the DB on `localhost`.
  → Needs Node/npm locally.
- **Claude Code inside the container** — `docker exec -it netflixdb bash`, run Claude Code there.
  → More isolated, no host install.

---

## Exercise 3 — Add the DBHub MCP server

From the CLI:

```bash
# MariaDB
claude mcp add netflix-mysql npx @bytebase/dbhub \
  -- --dsn mysql://root:mariadb@localhost:3306/netflixdb

# Postgres
claude mcp add netflix-psql npx @bytebase/dbhub \
  -- --dsn postgres://postgres:postgres@localhost:5432/netflixdb
```

Verify with `claude mcp list`. Restart Claude Code if needed.

---

## Exercise 4 — Warm-up queries

Ask in natural language. Watch it pick MCP tools, not guess:

- *"Which TV show has the highest viewing minutes per runtime minute?"*
- *"How does Stranger Things compare across seasons vs Squid Game?"*
- *"Pick my favourite show — how did it actually perform?"*

---

## Exercise 5 — The scorecard

Paste this prompt **verbatim**:

> Create a "Content Performance Scorecard" ranking all movies and TV shows using a composite score (0–100) built from five weighted metrics: **Efficiency (25%)**, **Longevity (20%)**, **Momentum (20%)**, **Consistency (15%)**, **Peak Performance (20%)**.
>
> Classify each as **Dominant Force**, **Rising Star**, **Flash Hit**, **Steady Performer**, or **Other**. Show the top 25.

Read the SQL **before** you trust the table.

---

## Exercise 6 — Reproducibility check

Once it answers:

1. Clear the chat.
2. Submit the **exact same prompt** again.
3. Compare:
   - Did the SQL change?
   - Did the numbers change?
   - Did the classifications change?

The agent is non-deterministic.
Same DB, same prompt, different answers.

→ What would you do differently before showing this to a stakeholder?

---

## Reflection

Park these in your notes:

- How would you make this safe against a **production** DB?
  (Read-only user? Query allow-list? Approval step?)
- Does the scorecard *looking* impressive make you trust it more than you should?
- Where at your team would "agent + DB over MCP" actually save real time?

---

## Recap

- **MCP is the USB-C of agent integrations** — one protocol, many servers, many clients.
- Servers expose **tools** (callable), **resources** (readable), and sometimes **prompts** (templates).
- The agent treats MCP tools like its native ones — **descriptions are how it learns when to call them.**
- Every server you install widens the agent's blast radius.
  → Install with the same care you'd give a browser extension.

---

## Where to go next

- Read the MCP spec at [modelcontextprotocol.io](https://modelcontextprotocol.io)
- Browse the official server list — there's likely already one for tools your team uses
- If you build one:
  - **Start with read-only tools**
  - Expose write tools only after you've watched the agent use them safely
- Loop back to Lesson 1: every MCP tool you add is **more tokens in the context window**.
  **Curate, don't accumulate.**

---

## End of track

You now have:

- An accurate mental model of **what an LLM is** and how to talk to it
- A toolbox of **workflows** that turn AI from autocomplete into engineering
- A **containerized agent** you can let off the leash safely
- A way to **extend that agent** to anything your team uses

Go build.
