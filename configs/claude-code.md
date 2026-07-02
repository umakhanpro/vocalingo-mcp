# Claude Code

```bash
claude mcp add --transport http vocalingo https://nodejs.vocalingo.app/mcp \
  --header "Authorization: Bearer vlmcp_YOUR_TOKEN"
```

Or in `.mcp.json` at the project root:

```json
{
  "mcpServers": {
    "vocalingo": {
      "type": "http",
      "url": "https://nodejs.vocalingo.app/mcp",
      "headers": { "Authorization": "Bearer vlmcp_YOUR_TOKEN" }
    }
  }
}
```

Get your token in the VocaLingo app: **Settings → API / MCP Token**.
