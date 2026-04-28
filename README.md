# Fiber AI — MCP Server

The Model Context Protocol (MCP) server provides a standardized interface that allows any compatible AI agent to access Fiber AI's data and tools — search companies, enrich contacts, reveal emails, and more — directly from your editor.

## For AI agents

If you are a coding agent and need to pick between MCP and the REST API,
start with the canonical agent-facing docs on the API:

- **Routing index + critical rules:** https://api.fiber.ai/llms.txt
- **Per-operation markdown:** `https://api.fiber.ai/ai-docs/<operationId>.md`
- **Operation index:** https://api.fiber.ai/ai-docs/index.md
- **MCP walkthrough for LLMs:** https://docs.fiber.ai/article/using-mcp-in-llms

Rule of thumb: use **MCP** when you're acting inside an IDE / chat and
each operation is a tool call; use the **REST API** (via
[`@fiberai/sdk`](https://github.com/fiber-ai/typescript-sdk) or
[`fiberai`](https://github.com/fiber-ai/python-sdk)) when you're building
an autonomous script or a production pipeline.

## Servers

Fiber AI offers three remote MCP servers:

| Server | URL | Auth | Best For |
|--------|-----|------|----------|
| **V2** | `https://mcp.fiber.ai/mcp/v2` | API key | Auto-generated direct tools for the top ~10 priority operations (`api_companySearch`, `api_peopleSearch`, etc.) |
| **V3** | `https://mcp.fiber.ai/mcp/v3` | OAuth (SSO) | Direct tools for every public operation with compact descriptions; the model can call or expand any tool on demand |
| **Core** | `https://mcp.fiber.ai/mcp` | API key | 5 meta-tools that discover and call any of 100+ API endpoints (`search_endpoints`, `list_tag_packs`, `list_all_endpoints`, `get_endpoint_details_full`, `call_operation`) |

> **Which one should I use?** **V2** is the fastest path for the most common flows with an API key. **V3** is the most ergonomic for broad agent coverage — it exposes every public operation as a direct tool with compact descriptions, authenticated via browser-based SSO login instead of a pasted key. **Core** keeps the tool count tiny (5) and lets the agent discover endpoints at runtime; it uses the same API-key auth as V2. You can register more than one; the server names (`fiber-ai-v2`, `fiber-ai-v3`, `fiber-ai-core`) stay distinct.

---

## Setup

### Smithery

Install via [Smithery](https://smithery.ai) with a single command:

```bash
npx -y @smithery/cli install @fiber-ai/mcp --client cursor
```

Replace `cursor` with your client: `claude`, `windsurf`, `vscode`, `zed`, etc.

Or browse and install from the Smithery web UI at [smithery.ai/server/@fiber-ai/mcp](https://smithery.ai/server/@fiber-ai/mcp).

---

### Cursor

Click the link below to install automatically — paste it into your browser address bar and press Enter:

**Install V2:**

```
cursor://anysphere.cursor-deeplink/mcp/install?name=FiberAI-V2&config=eyJ1cmwiOiJodHRwczovL21jcC5maWJlci5haS9tY3AvdjIifQ==
```

**Install Core:**

```
cursor://anysphere.cursor-deeplink/mcp/install?name=FiberAI&config=eyJ1cmwiOiJodHRwczovL21jcC5maWJlci5haS9tY3AifQ==
```

Or manually: open Cursor Settings → Features → MCP → "+ Add New MCP Server" → Type: `HTTP` → URL: `https://mcp.fiber.ai/mcp/v2`

---

### Claude Code

The simplest path passes your key as a header so the agent never sees it in chat:

```bash
claude mcp add --transport http fiber-ai-v2 https://mcp.fiber.ai/mcp/v2 \
  --header "x-api-key: $FIBER_API_KEY"
```

To also add the Core server:

```bash
claude mcp add --transport http fiber-ai https://mcp.fiber.ai/mcp \
  --header "x-api-key: $FIBER_API_KEY"
```

Or, if you prefer to give the key via chat, drop the `--header` flag and the agent will pass `apiKey` in the request body when you tell it your key.

Run `/mcp` inside a Claude Code session to verify the connection.

---

### Claude Desktop

From Claude settings → Connectors, add a new MCP server with the URL `https://mcp.fiber.ai/mcp/v2`.

Or edit your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "fiber-ai-v2": {
      "url": "https://mcp.fiber.ai/mcp/v2",
      "transport": { "type": "http" },
      "headers": { "x-api-key": "sk_live_..." }
    },
    "fiber-ai": {
      "url": "https://mcp.fiber.ai/mcp",
      "transport": { "type": "http" },
      "headers": { "x-api-key": "sk_live_..." }
    }
  }
}
```

The `headers` field is optional - if you omit it, the agent will pass `apiKey` in the request body when you give it your key in chat.

---

### Codex

```bash
codex mcp add fiber-ai --url https://mcp.fiber.ai/mcp/v2
```

Or add to `~/.codex/config.toml`:

```toml
[mcp_servers.fiber-ai]
url = "https://mcp.fiber.ai/mcp/v2"
transport = "http"

[mcp_servers.fiber-ai.headers]
x-api-key = "sk_live_..."
```

The `[mcp_servers.fiber-ai.headers]` block is optional - drop it and give the key in chat instead.

---

### Visual Studio Code

Press `Ctrl/Cmd + P`, search for **MCP: Add Server**, select **Command (stdio)**, and enter:

```
npx mcp-remote https://mcp.fiber.ai/mcp/v2
```

Name it `FiberAI` and activate it via **MCP: List Servers**.

Or add to `.vscode/mcp.json`. If your VS Code version supports HTTP MCP natively, prefer the header-based shape:

```json
{
  "mcpServers": {
    "fiber-ai": {
      "type": "http",
      "url": "https://mcp.fiber.ai/mcp/v2",
      "headers": { "x-api-key": "${env:FIBER_API_KEY}" }
    }
  }
}
```

Older VS Code releases need the stdio wrapper (`mcp-remote` forwards your `FIBER_API_KEY` env var as a header):

```json
{
  "mcpServers": {
    "fiber-ai": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://mcp.fiber.ai/mcp/v2",
        "--header",
        "x-api-key:${FIBER_API_KEY}"
      ]
    }
  }
}
```

---

### Windsurf

Press `Ctrl/Cmd + ,` → Cascade → MCP servers → Add custom server:

```json
{
  "mcpServers": {
    "fiber-ai": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://mcp.fiber.ai/mcp/v2",
        "--header",
        "x-api-key:${FIBER_API_KEY}"
      ]
    }
  }
}
```

---

### Zed

Press `Cmd + ,` and add:

```json
{
  "context_servers": {
    "fiber-ai": {
      "source": "custom",
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://mcp.fiber.ai/mcp/v2",
        "--header",
        "x-api-key:${FIBER_API_KEY}"
      ],
      "env": { "FIBER_API_KEY": "sk_live_..." }
    }
  }
}
```

---

### Others

Most MCP-compatible tools can be configured with:

- **URL**: `https://mcp.fiber.ai/mcp/v2`
- **Transport**: HTTP (Streamable HTTP)
- **Auth**: header `x-api-key: <your key>` (or `Authorization: Bearer <your key>`)
- For stdio-only clients: `npx -y mcp-remote https://mcp.fiber.ai/mcp/v2 --header x-api-key:$FIBER_API_KEY`

---

## Available Tools

### V2 Server (`/mcp/v2/`)

Direct API tools — each Fiber AI endpoint is exposed as an individual tool:

| Tool | Description |
|------|-------------|
| `api_companySearch` | Search for companies by industry, location, size, funding, etc. |
| `api_peopleSearch` | Search for people by title, seniority, department, etc. |
| `api_individualRevealSync` | Reveal work email and phone for a LinkedIn profile |
| `api_companyLiveFetch` | Get live LinkedIn data for a company |
| `api_personLiveFetch` | Get live LinkedIn data for a person |
| `api_getOrgCredits` | Check your credit balance |

### Core Server (`/mcp`)

Meta tools for dynamic endpoint discovery:

| Tool | Description |
|------|-------------|
| `search_endpoints` | Search for API endpoints by keyword |
| `list_all_endpoints` | List all available API endpoints |
| `get_endpoint_details_full` | Get full schema details for an endpoint |
| `call_operation` | Call any API endpoint by its operation ID |

---

## Authentication

V2 and Core require a Fiber AI API key. V3 uses OAuth (Clerk SSO) - the client opens a browser window for login and reuses the session token, no key configuration needed. Get an API key at [fiber.ai/app/api](https://fiber.ai/app/api).

You can supply your key in **any** of these three ways - pick whichever your client makes easiest:

### Option A - via chat (zero config)

The agent passes the key in the request body as `apiKey`. Simplest path: paste your key into the agent once and it'll keep using it.

```
You:    Use sk_live_... as my Fiber API key.
Agent:  [calls api_companySearch with { "apiKey": "sk_live_...", "searchParams": {...} }]
```

Caveat: the key ends up in chat history. Fine for personal use, not ideal for shared sessions.

### Option B - via `x-api-key` header (recommended for IDEs)

Configure the header once in your MCP client config; the agent never sees the key.

```json
{
  "mcpServers": {
    "fiber-ai-v2": {
      "type": "http",
      "url": "https://mcp.fiber.ai/mcp/v2",
      "headers": { "x-api-key": "sk_live_..." }
    },
    "fiber-ai-core": {
      "type": "http",
      "url": "https://mcp.fiber.ai/mcp",
      "headers": { "x-api-key": "sk_live_..." }
    }
  }
}
```

Most clients (Claude Code, Cursor, Claude Desktop, Codex, Windsurf, VS Code) accept a `headers` field on each MCP server. Reference your env var instead of hard-coding:

```json
"headers": { "x-api-key": "${env:FIBER_API_KEY}" }
```

### Option C - via `Authorization: Bearer` header

Same shape as Option B, but using the standard bearer-token header. Useful if your MCP client only supports `Authorization`-style auth.

```json
"headers": { "Authorization": "Bearer sk_live_..." }
```

### Resolution order

When more than one is present, the server resolves in this order: body/query `apiKey` -> `x-api-key` header -> `Authorization: Bearer`. The first non-empty value wins.

### V3 (OAuth)

V3 ignores all of the above. On first connect, the client opens `https://app.fiber.ai` for browser-based SSO login; the resulting session token is reused on subsequent calls. No env vars, no headers, no config to write.

---

## Example Usage

Once connected, ask your AI agent:

- *"Search for SaaS companies in New York with 50-200 employees"*
- *"Find the CEO of <a company you care about> and get their work email"*
- *"How many credits do I have left?"*
- *"Enrich this LinkedIn profile: linkedin.com/in/..."*

---

## SDKs

For building applications programmatically (not via MCP):

- **TypeScript**: `npm install @fiberai/sdk` — [GitHub](https://github.com/fiber-ai/typescript-sdk)
- **Python**: `pip install fiberai` — [GitHub](https://github.com/fiber-ai/python-sdk)

---

## FAQ

**Connection not working?**
Ensure your editor supports HTTP (Streamable HTTP) MCP transport. If it only supports stdio, use the `npx mcp-remote` wrapper shown in the VS Code / Windsurf / Zed instructions.

**Getting authentication errors?**
Make sure you're supplying a valid key via one of the three supported paths (body `apiKey`, `x-api-key` header, or `Authorization: Bearer`). See [Authentication](#authentication) above. Get a key at [fiber.ai/app/api](https://fiber.ai/app/api).

**Should I put the key in chat or in headers?**
For personal sessions, chat is fine. For shared sessions, team-config files committed to git, or anywhere chat history might leak, configure the `x-api-key` header at the MCP client layer so the agent never sees the raw key.

**Can I use both servers at the same time?**
Yes. Many users add both V2 and Core for maximum flexibility.

---

## License

MIT
