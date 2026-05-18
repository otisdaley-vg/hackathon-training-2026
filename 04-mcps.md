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

### The mental model

An **MCP server** is a small process — usually a local subprocess, sometimes a remote service — that exposes capabilities to an agent over a JSON-RPC connection. The agent is the **client**.

A server can expose three kinds of things:

- **Tools** — functions the agent can call. `create_jira_ticket(title, body)`, `query_bigquery(sql)`, `send_slack(channel, text)`. Tools have schemas; the agent learns about them from the server and decides when to use them.
- **Resources** — read-only data the agent can fetch by URI. `linear://issue/ENG-123`, `file:///path/to/log`. Think "GET endpoints."
- **Prompts** — reusable prompt templates the server offers. Less common in day-to-day use; useful for vendor-curated workflows ("triage this ticket").

You won't always use all three. Most servers in the wild are mostly tools.

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

## Hands-on (15 min)

We'll install a couple of MCP servers in Claude Code and use them.

**Exercise 1 — Install a filesystem server.** From your Claude Code config (`~/.claude/settings.json` or via `/config`), add an MCP server entry. A typical config block looks like:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/you/scratch"]
    }
  }
}
```

Restart Claude Code. Ask: *"List the files in my scratch directory and tell me which is the largest."* Notice that it uses the new tools instead of `bash`.

**Exercise 2 — Pick one server from the registry.** Browse [modelcontextprotocol.io](https://modelcontextprotocol.io) or the community awesome-mcp lists. Install one that maps to a real tool you use (GitHub, Linear, Slack, Postgres, Sentry…). Configure it with a read-only token first.

Ask the agent something only that integration could answer: *"What are my open GitHub PRs, and which has the most review comments?"* Confirm it uses MCP tools, not screen-scraping or guessing.

**Exercise 3 — Audit a tool.** Pick one tool exposed by the server you installed. Ask the agent: *"Show me the schema and description of the `<tool_name>` tool."* Read it. Could a malicious server description trick the agent into doing something unsafe? This is the attack surface.

**Exercise 4 (stretch) — Sketch your own server.** Pick something internal at your team that the agent currently can't see — a deployment dashboard, an internal directory, a custom ticketing system. Outline (in a paragraph) the 3–5 tools you'd expose. What are the names? Descriptions? Inputs? Which would be read-only vs write?

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
