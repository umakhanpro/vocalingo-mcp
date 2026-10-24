# Claude Code

Sign in with your VocaLingo account (OAuth, recommended):

```bash
claude mcp add --transport http vocalingo https://nodejs.vocalingo.app/mcp
```

Then run `/mcp` in Claude Code, pick `vocalingo` and sign in in the browser with the email of your VocaLingo account.

Or with a personal token (copy it in the VocaLingo app: **Settings → API / MCP Token**):

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
