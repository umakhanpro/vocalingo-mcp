# ChatGPT

1. Settings → **Apps & Connectors** → **Advanced** → enable **Developer mode** (required for custom connectors).
2. **Create** a connector: name `VocaLingo`, MCP server URL `https://nodejs.vocalingo.app/mcp`, authentication **OAuth**.
3. Sign in with the email of your VocaLingo account and allow access.

ChatGPT can't run `curl`, so for local files ask it to give you the `browserUploadUrl` from `get_upload_url` and drop the file there, or paste a public link instead.
