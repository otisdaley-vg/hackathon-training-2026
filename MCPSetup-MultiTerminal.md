# DBHub + Claude MCP setup

How to run [`@bytebase/dbhub`](https://github.com/bytebase/dbhub) against the dev container's Postgres so Claude Code can query the `movies` database via MCP, and so you get the DBHub web workbench at `http://localhost:8080/`.

## Prerequisites

- Dev container is up (see `README.md`) — the `db` service is reachable at `db:5432` from inside the `app` container.
- Port `8080` is forwarded (already configured in `.devcontainer/`).

## 1. MCP client config

`.mcp.json` at the repo root tells Claude Code where to find the MCP server:

```json
{
  "mcpServers": {
    "moviedb": {
      "type": "http",
      "url": "http://localhost:8080/mcp"
    }
  }
}
```

This expects an HTTP MCP server on port 8080 — DBHub provides this when run with `--transport=http`.

## 2. Start DBHub

From inside the dev container, run:

```bash
npx -y @bytebase/dbhub \
  --transport=http \
  --port=8080 \
  --dsn "postgres://hackathon:hackathon@db:5432/movies"
```

To run it in the background (and free up your shell):

```bash
nohup npx -y @bytebase/dbhub \
  --transport=http \
  --port=8080 \
  --dsn "postgres://hackathon:hackathon@db:5432/movies" \
  > /tmp/dbhub.log 2>&1 &
```

On a successful start the log shows:

```
Workbench at http://localhost:8080/
MCP server endpoint at http://localhost:8080/mcp
```

## 3. Verify

- **Web workbench:** open <http://localhost:8080/> in your browser.
- **MCP endpoint:** `curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8080/` should print `200`.
- **From Claude Code:** run `/mcp` — `moviedb` should report as connected. Try a query: "how many rows in the `movie` table?"

## 4. Stop DBHub

If started in the background with `nohup`:

```bash
pkill -f '@bytebase/dbhub'
```

If started via Claude's background-task runner, kill the background task by ID.

## Alternative: stdio transport

If you don't need the web workbench and only want the MCP server, you can register DBHub directly with Claude over stdio (no port, no background process — Claude spawns it):

```bash
claude mcp add moviedb -- npx -y @bytebase/dbhub \
  --transport stdio \
  --dsn "postgres://hackathon:hackathon@db:5432/movies"
```

This is the command in `.scripts/mcp.txt`. Use one transport or the other, not both — the HTTP setup above is preferred because it also gives you the workbench UI.

## Troubleshooting

- **`/mcp` shows `ConnectionRefused at http://localhost:8080/mcp`** — DBHub isn't running. Start it (step 2).
- **DBHub errors with `Database connection configuration is required`** — `--dsn` was missing or unquoted.
- **`db:5432` not resolving** — you're running outside the dev container. Use `localhost:5432` instead, since Postgres is published on the host.
