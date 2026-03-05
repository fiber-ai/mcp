# Fiber AI — MCP Server

The Model Context Protocol (MCP) server provides a standardized interface that allows any compatible AI agent to access Fiber AI's data and tools — search companies, enrich contacts, reveal emails, and more — directly from your editor.

## Servers

Fiber AI offers two remote MCP servers:

| Server | URL | Best For |
|--------|-----|----------|
| **V2** | `https://mcp.fiber.ai/mcp/v2/` | ~10 curated, high-priority API tools (`api_companySearch`, `api_peopleSearch`, `api_individualRevealSync`, etc.) |
| **Core** | `https://mcp.fiber.ai/mcp` | 4 meta tools that can discover and call any of 100+ API endpoints (`search_endpoints`, `call_operation`, `list_all_endpoints`, `get_endpoint_details_full`) |

> **Which one should I use?** **V2** gives your agent ~10 direct tools for the most common operations (search, enrich, reveal) — easiest for most agents. **Core** gives access to all 100+ API endpoints through meta tools that discover and call any endpoint dynamically. Most users start with V2; power users or those building complex workflows benefit from Core or both.

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
cursor://anysphere.cursor-deeplink/mcp/install?name=FiberAI-V2&config=eyJ1cmwiOiJodHRwczovL21jcC5maWJlci5haS9tY3AvdjIvIn0=
```

**Install Core:**

```
cursor://anysphere.cursor-deeplink/mcp/install?name=FiberAI&config=eyJ1cmwiOiJodHRwczovL21jcC5maWJlci5haS9tY3AifQ==
```

Or manually: open Cursor Settings → Features → MCP → "+ Add New MCP Server" → Type: `HTTP` → URL: `https://mcp.fiber.ai/mcp/v2/`

---

### Claude Code

```bash
claude mcp add --transport http fiber-ai-v2 https://mcp.fiber.ai/mcp/v2/
```

To also add the Core server:

```bash
claude mcp add --transport http fiber-ai https://mcp.fiber.ai/mcp
```

Run `/mcp` inside a Claude Code session to verify the connection.

---

### Claude Desktop

From Claude settings → Connectors, add a new MCP server with the URL `https://mcp.fiber.ai/mcp/v2/`.

Or edit your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "fiber-ai-v2": {
      "url": "https://mcp.fiber.ai/mcp/v2/",
      "transport": { "type": "http" }
    },
    "fiber-ai": {
      "url": "https://mcp.fiber.ai/mcp",
      "transport": { "type": "http" }
    }
  }
}
```

---

### Codex

```bash
codex mcp add fiber-ai --url https://mcp.fiber.ai/mcp/v2/
```

Or add to `~/.codex/config.toml`:

```toml
[mcp_servers.fiber-ai]
url = "https://mcp.fiber.ai/mcp/v2/"
```

---

### Visual Studio Code

Press `Ctrl/Cmd + P`, search for **MCP: Add Server**, select **Command (stdio)**, and enter:

```
npx mcp-remote https://mcp.fiber.ai/mcp/v2/
```

Name it `FiberAI` and activate it via **MCP: List Servers**.

Or add to `.vscode/mcp.json`:

```json
{
  "mcpServers": {
    "fiber-ai": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.fiber.ai/mcp/v2/"]
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
      "args": ["-y", "mcp-remote", "https://mcp.fiber.ai/mcp/v2/"]
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
      "args": ["-y", "mcp-remote", "https://mcp.fiber.ai/mcp/v2/"],
      "env": {}
    }
  }
}
```

---

### Others

Most MCP-compatible tools can be configured with:

- **URL**: `https://mcp.fiber.ai/mcp/v2/`
- **Transport**: HTTP (Streamable HTTP)
- For stdio-only clients: `npx -y mcp-remote https://mcp.fiber.ai/mcp/v2/`

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

All API calls require your Fiber AI API key. Pass it in the request body as `apiKey`:

```
"Search for AI companies" → Agent calls api_companySearch with { "apiKey": "sk_live_...", ... }
```

Get your API key at [fiber.ai/app/api](https://fiber.ai/app/api).

---

## Example Usage

Once connected, ask your AI agent:

- *"Search for SaaS companies in New York with 50-200 employees"*
- *"Find the CEO of Stripe and get their work email"*
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
Make sure you're passing a valid `apiKey` in your requests. Get one at [fiber.ai/app/api](https://fiber.ai/app/api).

**Can I use both servers at the same time?**
Yes. Many users add both V2 and Core for maximum flexibility.

---

## License

MIT
