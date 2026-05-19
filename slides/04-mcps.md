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

## Hands-on

We'll install a couple of MCP servers in Claude Code and use them.

---

## Exercise 1 — Install a filesystem server

From Claude Code config (`~/.claude/settings.json` or via `/config`):

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/you/scratch"
      ]
    }
  }
}
```

Restart Claude Code.
Ask: *"List the files in my scratch directory and tell me which is the largest."*

Notice that it uses the **new tools** instead of `bash`.

---

## Exercise 2 — Pick one server from the registry

Browse [modelcontextprotocol.io](https://modelcontextprotocol.io) or community awesome-mcp lists.

Install one that maps to a **real tool you use**:
- GitHub
- Linear
- Slack
- Postgres
- Sentry…

**Configure with a read-only token first.**

Ask the agent something only that integration could answer:
*"What are my open GitHub PRs, and which has the most review comments?"*

Confirm it uses **MCP tools**, not screen-scraping or guessing.

---

## Exercise 3 — Audit a tool

Pick one tool exposed by the server you installed.

Ask the agent:

> *"Show me the schema and description of the `<tool_name>` tool."*

Read it.

**Could a malicious server description trick the agent into doing something unsafe?**

This is the attack surface.

---

## Exercise 4 (stretch) — Sketch your own server

Pick something internal at your team the agent currently can't see:
- A deployment dashboard
- An internal directory
- A custom ticketing system

Outline in a paragraph the **3–5 tools** you'd expose:

- Names?
- Descriptions?
- Inputs?
- Which would be read-only vs write?

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
