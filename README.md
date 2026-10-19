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
| **V2** | `https://mcp.fiber.ai/mcp/v2` | API key | Auto-generated direct tools for the top ~10 priority operations (`companySearch_tool`, `peopleSearch_tool`, etc.) |
| **V3** | `https://mcp.fiber.ai/mcp/v3` | OAuth (SSO) | Direct tools for every public operation with compact descriptions; the model can call or expand any tool on demand |
| **Core** | `https://mcp.fiber.ai/mcp` | API key | 5 meta-tools that discover and call any of 100+ API endpoints (`search_endpoints`, `list_tag_packs`, `list_all_endpoints`, `get_endpoint_details_full`, `call_operation`) |

> **Which one should I use?** **V2** is the fastest path for the most common flows with an API key. **V3** is the most ergonomic for broad agent coverage — it exposes every public operation as a direct tool with compact descriptions, authenticated via browser-based SSO login instead of a pasted key. **Core** keeps the tool count tiny (5) and lets the agent discover endpoints at runtime; it uses the same API-key auth as V2. You can register more than one; the server names (`fiber-ai-v2`, `fiber-ai-v3`, `fiber-ai-core`) stay distinct.

## Want more than MCP? Use the Fiber AI plugin

The [Fiber AI plugin](https://github.com/fiber-ai/fiber-ai-plugin) bundles these MCP servers with 16 skills (search, enrich, audiences, recruiting and SDR playbooks) and 7 persona subagents (`ai-sdr`, `ai-recruiter`, `gtm-strategist`, and more) for Claude Code, Cursor, Gemini CLI, OpenCode, Codex, and VS Code. See its [install guide](https://github.com/fiber-ai/fiber-ai-plugin#install). This repo covers the MCP servers only.

---

## Setup

Get an API key at [fiber.ai/app/api](https://fiber.ai/app/api) for V2 and Core. The examples read it from a `FIBER_API_KEY` environment variable. V3 needs no key: your client opens a browser for SSO login on first connect.

Connect directly with the config for your client below, or through [Smithery](#smithery).

### Smithery

Fiber AI is listed on Smithery as [`fiberai/fiber-mcp`](https://smithery.ai/servers/fiberai/fiber-mcp) (V2) and [`fiberai/fiber-core`](https://smithery.ai/servers/fiberai/fiber-core) (Core). Install into your client with one command:

```bash
npx -y @smithery/cli@latest mcp add fiberai/fiber-mcp --client cursor
```

Replace `cursor` with your client: `claude-code`, `claude`, `vscode`, `codex`, `gemini-cli`, `windsurf`, `opencode`, `cline`, `goose`, and more (`npx -y @smithery/cli@latest mcp add --help` lists them all). Use `fiberai/fiber-core` for Core. On first connect your client opens Smithery in the browser to sign in and set your Fiber API key.

To use it as a Smithery connection instead (for agents that call tools through the Smithery CLI):

```bash
npx -y @smithery/cli@latest mcp add fiberai/fiber-mcp --id fiber-ai-v2 \
  --headers "{\"x-api-key\": \"$FIBER_API_KEY\"}"
npx -y @smithery/cli@latest tool list fiber-ai-v2
```

---

### Cursor

Paste a link into your browser address bar and press Enter (Cursor must be installed):

**V2:**

```
cursor://anysphere.cursor-deeplink/mcp/install?name=fiber-ai-v2&config=eyJ0eXBlIjoiaHR0cCIsInVybCI6Imh0dHBzOi8vbWNwLmZpYmVyLmFpL21jcC92MiJ9
```

**V3 (OAuth):**

```
cursor://anysphere.cursor-deeplink/mcp/install?name=fiber-ai-v3&config=eyJ0eXBlIjoiaHR0cCIsInVybCI6Imh0dHBzOi8vbWNwLmZpYmVyLmFpL21jcC92MyJ9
```

**Core:**

```
cursor://anysphere.cursor-deeplink/mcp/install?name=fiber-ai-core&config=eyJ0eXBlIjoiaHR0cCIsInVybCI6Imh0dHBzOi8vbWNwLmZpYmVyLmFpL21jcCJ9
```

The links don't include your key; give it to the agent in chat, or add it to `~/.cursor/mcp.json` (or `.cursor/mcp.json` in your project):

```json
{
  "mcpServers": {
    "fiber-ai-v2": {
      "type": "http",
      "url": "https://mcp.fiber.ai/mcp/v2",
      "headers": { "x-api-key": "${env:FIBER_API_KEY}" }
    },
    "fiber-ai-v3": {
      "type": "http",
      "url": "https://mcp.fiber.ai/mcp/v3"
    },
    "fiber-ai-core": {
      "type": "http",
      "url": "https://mcp.fiber.ai/mcp",
      "headers": { "x-api-key": "${env:FIBER_API_KEY}" }
    }
  }
}
```

---

### Claude Code

```bash
claude mcp add --transport http fiber-ai-v2 https://mcp.fiber.ai/mcp/v2 \
  --header "x-api-key: $FIBER_API_KEY"

claude mcp add --transport http fiber-ai-v3 https://mcp.fiber.ai/mcp/v3

claude mcp add --transport http fiber-ai-core https://mcp.fiber.ai/mcp \
  --header "x-api-key: $FIBER_API_KEY"
```

Run `/mcp` inside a Claude Code session to verify the connection and complete the V3 login.

---

### Claude Desktop and claude.ai

Claude Desktop and claude.ai add remote servers as **custom connectors**, which authenticate with OAuth and can't send an API-key header. Use V3:

1. Open **Settings → Connectors → Add custom connector**.
2. Enter `https://mcp.fiber.ai/mcp/v3` and sign in when prompted.

You can also add `https://mcp.fiber.ai/mcp/v2` as a connector and give the agent your key in chat (see [Authentication](#authentication)).

---

### Codex

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.fiber-ai-v2]
url = "https://mcp.fiber.ai/mcp/v2"
env_http_headers = { "x-api-key" = "FIBER_API_KEY" }

[mcp_servers.fiber-ai-v3]
url = "https://mcp.fiber.ai/mcp/v3"

[mcp_servers.fiber-ai-core]
url = "https://mcp.fiber.ai/mcp"
env_http_headers = { "x-api-key" = "FIBER_API_KEY" }
```

`env_http_headers` sends the value of the `FIBER_API_KEY` environment variable as the `x-api-key` header. Then sign in to V3:

```bash
codex mcp login fiber-ai-v3
```

The Codex IDE extension and the Codex app read the same `~/.codex/config.toml`.

---

### Gemini CLI

```bash
gemini mcp add -t http -H "x-api-key: $FIBER_API_KEY" fiber-ai-v2 https://mcp.fiber.ai/mcp/v2
gemini mcp add -t http fiber-ai-v3 https://mcp.fiber.ai/mcp/v3
gemini mcp add -t http -H "x-api-key: $FIBER_API_KEY" fiber-ai-core https://mcp.fiber.ai/mcp
```

Add `-s user` to install for every project instead of the current one. Run `gemini mcp list` to verify, and `/mcp auth fiber-ai-v3` inside a session to sign in to V3.

---

### OpenCode

Add to `opencode.json` in your project (or `~/.config/opencode/opencode.json`):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "fiber-ai-v2": {
      "type": "remote",
      "url": "https://mcp.fiber.ai/mcp/v2",
      "headers": { "x-api-key": "{env:FIBER_API_KEY}" }
    },
    "fiber-ai-v3": {
      "type": "remote",
      "url": "https://mcp.fiber.ai/mcp/v3"
    },
    "fiber-ai-core": {
      "type": "remote",
      "url": "https://mcp.fiber.ai/mcp",
      "headers": { "x-api-key": "{env:FIBER_API_KEY}" }
    }
  }
}
```

Run `opencode mcp list` to verify, and `opencode mcp auth fiber-ai-v3` to sign in to V3.

---

### Visual Studio Code

Add to `.vscode/mcp.json` in your workspace (VS Code reads servers from the top-level `servers` key):

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "fiber-api-key",
      "description": "Fiber AI API key",
      "password": true
    }
  ],
  "servers": {
    "fiber-ai-v2": {
      "type": "http",
      "url": "https://mcp.fiber.ai/mcp/v2",
      "headers": { "x-api-key": "${input:fiber-api-key}" }
    },
    "fiber-ai-v3": {
      "type": "http",
      "url": "https://mcp.fiber.ai/mcp/v3"
    },
    "fiber-ai-core": {
      "type": "http",
      "url": "https://mcp.fiber.ai/mcp",
      "headers": { "x-api-key": "${input:fiber-api-key}" }
    }
  }
}
```

VS Code prompts for the key once and stores it securely. Start the servers from **MCP: List Servers**.

---

### Windsurf (Devin Desktop)

Open the Cascade panel's `...` menu → **Open MCP config file**, and add:

```json
{
  "mcpServers": {
    "fiber-ai-v2": {
      "serverUrl": "https://mcp.fiber.ai/mcp/v2",
      "headers": { "x-api-key": "${env:FIBER_API_KEY}" }
    },
    "fiber-ai-v3": {
      "serverUrl": "https://mcp.fiber.ai/mcp/v3"
    },
    "fiber-ai-core": {
      "serverUrl": "https://mcp.fiber.ai/mcp",
      "headers": { "x-api-key": "${env:FIBER_API_KEY}" }
    }
  }
}
```

---

### Zed

Open **Settings → AI → MCP Servers → Add Server → Add Remote Server**, or add to your settings file:

```json
{
  "context_servers": {
    "fiber-ai-v2": {
      "url": "https://mcp.fiber.ai/mcp/v2",
      "headers": { "Authorization": "Bearer sk_live_..." }
    },
    "fiber-ai-v3": {
      "url": "https://mcp.fiber.ai/mcp/v3"
    },
    "fiber-ai-core": {
      "url": "https://mcp.fiber.ai/mcp",
      "headers": { "Authorization": "Bearer sk_live_..." }
    }
  }
}
```

Zed starts an OAuth login for any remote server without an `Authorization` header, so use the Bearer form for V2 and Core.

---

### Others

Most MCP-compatible tools can be configured with:

- **URL**: `https://mcp.fiber.ai/mcp/v2` (or `/mcp/v3`, `/mcp`)
- **Transport**: HTTP (Streamable HTTP)
- **Auth**: header `x-api-key: <your key>` (or `Authorization: Bearer <your key>`) for V2 and Core; OAuth for V3
- For stdio-only clients: `npx -y mcp-remote https://mcp.fiber.ai/mcp/v2 --header x-api-key:$FIBER_API_KEY`

---

## Available Tools

### V2 Server (`/mcp/v2`)

Direct API tools — each Fiber AI endpoint is exposed as an individual tool:

| Tool | Description |
|------|-------------|
| `companySearch_tool` | Search for companies by industry, location, size, funding, etc. |
| `peopleSearch_tool` | Search for people by title, seniority, department, etc. |
| `profileLiveEnrich_tool` | Live-fetch a LinkedIn profile |
| `companyLiveEnrich_tool` | Live-fetch a LinkedIn company |
| `getOrgCredits_tool` | Check your credit balance |
| `getRateLimits_tool` | Check your organization's rate limits |
| `getAutoTopupSettings_tool` | Get auto top-up settings |
| `getAllApiKeys_tool` | List your API keys |
| `cardsAttach_tool` | Attach a card to a trial account |
| `getQuickstartScript_tool` | Get the quickstart install script |

### V3 Server (`/mcp/v3`)

One direct tool per public operation (100+), with compact descriptions the model can expand on demand.

### Core Server (`/mcp`)

Meta tools for dynamic endpoint discovery:

| Tool | Description |
|------|-------------|
| `search_endpoints` | Search for API endpoints by keyword |
| `list_tag_packs` | List curated groups of endpoints to narrow the catalog |
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
Agent:  [calls companySearch_tool with { "apiKey": "sk_live_...", "searchParams": {...} }]
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

Most clients accept a `headers` field on each MCP server and can read the key from an environment variable instead of hard-coding it. The field name and variable syntax differ per client; see the [Setup](#setup) section for yours.

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
Ensure your editor supports HTTP (Streamable HTTP) MCP transport. If it only supports stdio, use the `npx mcp-remote` wrapper shown under [Others](#others).

**Getting authentication errors?**
Make sure you're supplying a valid key via one of the three supported paths (body `apiKey`, `x-api-key` header, or `Authorization: Bearer`). See [Authentication](#authentication) above. Get a key at [fiber.ai/app/api](https://fiber.ai/app/api).

**Should I put the key in chat or in headers?**
For personal sessions, chat is fine. For shared sessions, team-config files committed to git, or anywhere chat history might leak, configure the `x-api-key` header at the MCP client layer so the agent never sees the raw key.

**Can I use more than one server at the same time?**
Yes. You can register V2, V3, and Core side by side; their names stay distinct.

---

## License

MIT
