# Claude Desktop / Claude.ai

1. **Settings → Connectors → Add custom connector**
   - URL: `https://nodejs.vocalingo.app/mcp`
   - Header: `Authorization: Bearer vlmcp_YOUR_TOKEN`

2. To let Claude upload your chat attachments (voice messages, videos) to VocaLingo, allow the upload domain once:
   **Settings → Capabilities → Code execution → Additional allowed domains** → add `sb.vocalingo.app`

Get your token in the VocaLingo app: **Settings → API / MCP Token**.
