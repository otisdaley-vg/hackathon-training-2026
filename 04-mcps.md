# Lesson 4 — MCPs

**Time:** ~30 min
**Goal:** Understand the Model Context Protocol (MCP), why it exists, and how to wire an MCP server into your agent so it can talk to external systems.

## Objectives

By the end of this lesson you will be able to:

- Explain what MCP is and the problem it solves.
- Distinguish a **tool**, a **resource**, and a **prompt** in MCP terms.
- Install and configure an existing MCP server in Claude Code.
- Sketch the shape of an MCP server you'd build for your own team.

## Concepts

### Why MCP exists

A terminal agent with `bash` and `Edit` is already powerful. But most real work lives outside your filesystem: tickets in Jira, designs in Figma, data in BigQuery, errors in Sentry, runbooks in Notion. Without a way in, the agent has to ask you to paste things — which means it can't act, only advise.

Before MCP, every agent vendor wrote their own integrations and every tool vendor wrote their own agent plugin. **N × M** maintenance.

MCP (Model Context Protocol, open spec from Anthropic, late 2024) standardizes this. One protocol; any compliant agent can talk to any compliant server. **N + M**.

> **Like I'm five:** Before USB-C, every gadget needed its own cable — phone cable, camera cable, weird-printer cable. A drawer full of them and nothing fit anything. **MCP is the USB-C of agent tools** — one plug shape, everything fits.

### The mental model

An **MCP server** is a small process — usually a local subprocess, sometimes a remote service — that exposes capabilities to an agent over a JSON-RPC connection. The agent is the **client**.

A server can expose three kinds of things:

- **Tools** — functions the agent can call. `create_jira_ticket(title, body)`, `query_bigquery(sql)`, `send_slack(channel, text)`. Tools have schemas; the agent learns about them from the server and decides when to use them.
- **Resources** — read-only data the agent can fetch by URI. `linear://issue/ENG-123`, `file:///path/to/log`. Think "GET endpoints."
- **Prompts** — reusable prompt templates the server offers. Less common in day-to-day use; useful for vendor-curated workflows ("triage this ticket").

You won't always use all three. Most servers in the wild are mostly tools.

> **Like I'm five:** A kitchen drawer. **Tools** are gadgets that *do* things — the can-opener, the whisk. **Resources** are things you *read* — the cookbook, the label on the jar. **Prompts** are post-it notes saying *"how Grandma makes the stew."* You'll mostly grab tools; the others are there if you need them.

### Transport

Two common transports:

- **stdio**: the server is launched as a subprocess by the agent, communication over stdin/stdout. This is the default for local tools.
- **HTTP / SSE**: remote servers, useful when the server holds creds you don't want on every dev's laptop, or wraps a SaaS.

You rarely care about transport in practice — you just paste a config block.

### What the agent actually sees

When the agent connects to a server, it asks "what can you do?" and the server replies with a list of tool names, descriptions, and JSON schemas for arguments. From the agent's perspective, MCP tools look identical to its built-in tools. It picks the right one based on the description and the user's request.

This is why **tool descriptions are load-bearing**. If you build a server, write the description like a docstring aimed at the model: when should this be used, what does it return, what are the gotchas.

### Trust and risk

Same blast-radius thinking as Lesson 3, but bigger:

- An MCP server can read your filesystem, hit your APIs, spend money. Only install servers you trust.
- A malicious or buggy server can confuse the agent into doing the wrong thing — "tool injection" is a real attack surface. Treat tool output as untrusted input.
- For shared/remote servers, audit the auth model. Who can call this? What can they do?

> **Like I'm five:** Letting a new toy into your house. The toy might be lovely. The toy might also be a tiny robot with sharp edges that walks into the kitchen and unplugs the fridge. Once you plug it in, it can do things — so check what it does *before* you plug it in, not after.

## Hands-on (15 min)

We'll wire Claude Code up to a real database via an MCP server and let it write the SQL for us. Exercise adapted from [ai-training.re-cinq.com/mcp-db](https://ai-training.re-cinq.com/mcp-db/).

**Exercise 1 — Stand up the Netflix database.** Pull and run the pre-baked container (pick one):

```bash
# MariaDB / MySQL
docker pull ghcr.io/re-cinq/ai-training-netflixdb-mariadb:latest
docker run -d --name netflixdb -p 3306:3306 ghcr.io/re-cinq/ai-training-netflixdb-mariadb

# OR Postgres
docker pull ghcr.io/re-cinq/ai-training-netflixdb-postgres:latest
docker run -d --name netflixdb -p 5432:5432 ghcr.io/re-cinq/ai-training-netflixdb-postgres
```

Confirm with `docker ps`. You should see 4 tables (`movie`, `tv_show`, `season`, `view_summary`) once connected — ~37K rows total.

**Exercise 2 — Pick where the agent runs.** Two options, same outcome:

- **Agent on host** — your existing Claude Code talks to the DB on `localhost`. Needs Node/npm locally.
- **Claude Code inside the DB container** — `docker exec -it netflixdb bash`, run Claude Code there. More isolated, no host install needed.

**Exercise 3 — Add the DBHub MCP server.** From the CLI:

```bash
# MariaDB
claude mcp add netflix-mysql npx @bytebase/dbhub -- --dsn mysql://root:mariadb@localhost:3306/netflixdb

# Postgres
claude mcp add netflix-psql npx @bytebase/dbhub -- --dsn postgres://postgres:postgres@localhost:5432/netflixdb
```

Verify with `claude mcp list`. Restart Claude Code if needed.

**Exercise 4 — Warm-up queries.** Ask the agent natural-language questions and watch it pick the MCP tools instead of guessing:

- *"Which TV show has the highest viewing minutes per runtime minute?"*
- *"How does Stranger Things compare across seasons vs Squid Game?"*
- *"Pick my favourite show — how did it actually perform?"*

**Exercise 5 — The scorecard.** Paste this prompt verbatim:

> Create a "Content Performance Scorecard" ranking all movies and TV shows using a composite score (0–100) built from five weighted metrics: Efficiency (25%, viewing hours per runtime minute), Longevity (20%, weeks in top 10), Momentum (20%, recent vs overall average), Consistency (15%, week-to-week stability), Peak Performance (20%, best ranking achieved). Classify each as Dominant Force, Rising Star, Flash Hit, Steady Performer, or Other. Show the top 25.

Read the SQL it writes before you trust the table. Does it match your intent?

**Exercise 6 — Reproducibility check.** Once it answers, clear the chat and submit the *exact same* prompt again. Compare:

- Did the SQL change?
- Did the numbers change?
- Did the classifications change?

This is the part nobody warns you about. The agent is non-deterministic; even with the same DB and the same prompt, you can get different answers. What would you do differently before putting this in front of a stakeholder?

**Reflection.** Park these in your notes:

- How would you make this safe to run against a production DB? (Read-only user? Query allow-list? Approval step?)
- Does the scorecard *looking* impressive make you trust it more than you should?
- What's one place at your team where "agent + DB over MCP" would actually save real time?

## Recap

- MCP is the USB-C of agent integrations: one protocol, many servers, many clients.
- Servers expose **tools** (callable), **resources** (readable), and sometimes **prompts** (templates).
- The agent treats MCP tools like its native ones; descriptions are how it learns when to call them.
- Every server you install widens the agent's blast radius. Install with the same care you'd give a browser extension.

## Where to go next

- Read the MCP spec at [modelcontextprotocol.io](https://modelcontextprotocol.io).
- Browse the official server list — there's likely already a server for the tools your team uses.
- If you build one, start with read-only tools, expose write tools only after you've watched the agent use them safely.
- Loop back to Lesson 1: every MCP tool you add is more tokens in the context window. Curate, don't accumulate.
