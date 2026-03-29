# Usage with Claude Desktop

## Option 1: Python / uvx (local install)

Add this to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "mikrotik": {
      "command": "uvx",
      "args": ["mcp-server-mikrotik", "--host", "<HOST>", "--username", "<USERNAME>", "--password", "<PASSWORD>", "--port", "22"]
    }
  }
}
```

## Option 2: Docker — stdio (Claude Desktop manages the container)

Claude Desktop can spawn the Docker container as a subprocess using the `stdio` transport. The container starts when a session begins and exits when it ends.

```json
{
  "mcpServers": {
    "mikrotik": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "-e", "MIKROTIK_HOST=<HOST>",
        "-e", "MIKROTIK_USERNAME=<USERNAME>",
        "-e", "MIKROTIK_PASSWORD=<PASSWORD>",
        "-e", "MIKROTIK_PORT=22",
        "-e", "MIKROTIK_MCP__TRANSPORT=stdio",
        "ghcr.io/gcormier/mikrotik-mcp:latest"
      ]
    }
  }
}
```

## Option 3: Docker — persistent SSE server

Run the container once as a long-lived server, then point Claude Desktop at it. This is useful when you want the server always available without Claude Desktop having to start it.

**Step 1 — start the persistent container:**

```bash
docker run -d \
  --name mikrotik-mcp \
  --restart unless-stopped \
  -p 8000:8000 \
  -e MIKROTIK_HOST=<HOST> \
  -e MIKROTIK_USERNAME=<USERNAME> \
  -e MIKROTIK_PASSWORD=<PASSWORD> \
  ghcr.io/gcormier/mikrotik-mcp:latest
```

**Step 2 — add the SSE URL to `claude_desktop_config.json`:**

```json
{
  "mcpServers": {
    "mikrotik": {
      "url": "http://localhost:8000/sse"
    }
  }
}
```

