# claude.ai and Claude Desktop

## Connect with your VocaLingo account (recommended)

1. **Settings → Connectors → Add custom connector**.
2. Name: `VocaLingo`, URL: `https://nodejs.vocalingo.app/mcp`.
3. Click **Connect**, enter the email of your VocaLingo account, type the code from the email and press **Allow**.

The same connector then works in claude.ai, Claude Desktop and Claude mobile.

To let Claude upload your chat attachments (voice messages, videos), allow the upload domain once:
**Settings → Capabilities → Code execution → Additional allowed domains** → add `sb.vocalingo.app`.
Without it, uploads from Claude's sandbox fail with a network error — then ask Claude for the browser upload link instead.

To disconnect: remove the connector in Claude, or turn MCP off in the VocaLingo app (**Settings → API / MCP Token → Disable**).

## Alternative: personal token via `mcp-remote` (Claude Desktop only)

If you prefer a token, connect through the `mcp-remote` bridge in the config file.

1. Make sure Node.js is installed (`node -v`; if not — [nodejs.org](https://nodejs.org)).

2. Open the config file:
   - **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
   - **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

   (Or in Claude Desktop: **Settings → Developer → Edit Config**.)

3. Add the server (replace `vlmcp_YOUR_TOKEN` with your token from the app: **Settings → API / MCP Token**):

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
