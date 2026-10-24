# claude.ai and Claude Desktop

## Connect with your VocaLingo account (recommended)

1. Open **[Add VocaLingo to Claude](https://claude.ai/customize/connectors?modal=add-custom-connector&connectorName=VocaLingo&connectorUrl=https%3A%2F%2Fnodejs.vocalingo.app%2Fmcp)** — the dialog opens with the name and URL filled in (or add it by hand: Customize → **Connectors** → **Add custom connector**, URL `https://nodejs.vocalingo.app/mcp`).
2. Click **Add**, then **Connect**.
3. Sign in to VocaLingo with an email code, Google, or a one-time code from the app (**Settings → AI agents (MCP)**), and press **Allow**.

The same connector then works in claude.ai, Claude Desktop and Claude mobile.

**Files:** paste a link, or ask Claude to upload a file — an upload button appears right in the chat; pick the file and Claude continues on its own. No button? Ask for the upload page link. If Claude uploads from its code sandbox instead, allow the domain once: **Settings → Capabilities → Code execution → Additional allowed domains** → `sb.vocalingo.app`.

To disconnect: remove the connector in Claude, or turn MCP off in the VocaLingo app (**Settings → AI agents (MCP) → Disable**).

## Alternative: personal token via `mcp-remote` (Claude Desktop only)

If you prefer a token, connect through the `mcp-remote` bridge in the config file.

1. Make sure Node.js is installed (`node -v`; if not — [nodejs.org](https://nodejs.org)).

2. Open the config file:
   - **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
   - **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

   (Or in Claude Desktop: **Settings → Developer → Edit Config**.)

3. Add the server (replace `vlmcp_YOUR_TOKEN` with your token from the app: **Settings → AI agents (MCP)**):

```json
{
  "mcpServers": {
    "vocalingo": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://nodejs.vocalingo.app/mcp",
        "--header",
        "Authorization:${AUTH_HEADER}"
      ],
      "env": {
        "AUTH_HEADER": "Bearer vlmcp_YOUR_TOKEN"
      }
    }
  }
}
```

   `Authorization:${AUTH_HEADER}` is written **without a space** on purpose — spaces inside args break on some platforms, so the value goes through `env`.

4. **Fully restart Claude Desktop** (Quit, not just close the window).
