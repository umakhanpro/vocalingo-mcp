# Claude Desktop

Claude Desktop's "Add custom connector" UI currently has **no field for a static `Authorization` header** (only URL + OAuth), so connect VocaLingo through the `mcp-remote` bridge in the config file instead.

1. Make sure Node.js is installed (`node -v` in a terminal; if not — [nodejs.org](https://nodejs.org)).

2. Open the config file:
   - **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
   - **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

   (Or in Claude Desktop: **Settings → Developer → Edit Config**.)

3. Add the server (replace `vlmcp_YOUR_TOKEN` with your token):

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

   Note: `Authorization:${AUTH_HEADER}` is written **without a space** on purpose — spaces inside args break on some platforms, so the value goes through `env`.

4. **Fully restart Claude Desktop** (Quit, not just close the window). The `vocalingo` tools appear under the 🔌 icon.

5. To let Claude upload your chat attachments (voice messages, videos) to VocaLingo, allow the upload domain once:
   **Settings → Capabilities → Code execution → Additional allowed domains** → add `sb.vocalingo.app`.
   Without this, file uploads from Claude's sandbox fail with a network error.

## Claude.ai (browser)

The browser version currently has no way to pass an `Authorization` header to a custom connector and cannot run the `mcp-remote` bridge, so VocaLingo MCP is **not supported on claude.ai yet**. Use Claude Desktop.

Get your token in the VocaLingo app: **Settings → API / MCP Token**.
