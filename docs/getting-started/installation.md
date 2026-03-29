# Installation

## Prerequisites
- Python 3.8+
- MikroTik RouterOS device with API access enabled
- Python dependencies (routeros-api or similar)

## Manual Installation

```bash
# Clone the repository
git clone https://github.com/jeff-nasseri/mikrotik-mcp/tree/master
cd mcp-mikrotik

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -e .

# Run the server (stdio, default)
mcp-server-mikrotik

# Run with SSE transport
mcp-server-mikrotik --mcp.transport sse

# Run with streamable HTTP transport
mcp-server-mikrotik --mcp.transport streamable-http
```

### CLI Options

| Flag | Description | Default |
|------|-------------|---------|
| `--host` | MikroTik device IP/hostname | from config |
| `--username` | SSH username | from config |
| `--password` | SSH password | from config |
| `--key-filename` | SSH key filename | from config |
| `--port` | SSH port | `22` |
| `--mcp.transport` | Transport type: `stdio`, `sse`, `streamable-http` | `stdio` |
| `--mcp.host` | HTTP server listen address | `0.0.0.0` |
| `--mcp.port` | HTTP server listen port | `8000` |

HTTP-based transports (`sse`, `streamable-http`) expose a `GET /health` endpoint for health checks. This endpoint is **not available** in `stdio` mode.

## Docker Installation

The easiest way to run the MCP MikroTik server is using Docker. The pre-built image is available on the GitHub Container Registry — no local build required.

```bash
docker pull ghcr.io/gcormier/mikrotik-mcp:latest
```

### Transport modes

The Docker container supports two operating modes controlled by `MIKROTIK_MCP__TRANSPORT`:

| Mode | Transport value | Container lifecycle | Best for |
|------|----------------|---------------------|----------|
| **Persistent server** *(default)* | `sse` or `streamable-http` | Stays running, listens on port 8000 | Shared/remote deployments, any client that connects over HTTP |
| **stdio** | `stdio` | Started and stopped by the MCP client per session | Local IDE integrations (Cursor, Claude Desktop) that manage the container as a subprocess |

### Persistent server (default)

By default the container starts an **SSE server** on port 8000 and remains open waiting for connections. This is the recommended mode for most deployments.

```bash
docker run -d \
  -p 8000:8000 \
  -e MIKROTIK_HOST=192.168.88.1 \
  -e MIKROTIK_USERNAME=admin \
  -e MIKROTIK_PASSWORD=your_password \
  ghcr.io/gcormier/mikrotik-mcp:latest
```

The server will be available at:
- **SSE:** `http://localhost:8000/sse`
- **Streamable HTTP** (use `MIKROTIK_MCP__TRANSPORT=streamable-http`): `http://localhost:8000/mcp`

A `GET /health` endpoint is also available for health checks.

### stdio mode (for IDE / desktop client integration)

If your MCP client (e.g. Cursor, Claude Desktop) manages the container as a subprocess over stdin/stdout, set the transport to `stdio`. The client spawns the container when needed and the container exits when the session ends.

Add this to your `~/.cursor/mcp.json` or equivalent client config:

```json
{
  "mcpServers": {
    "mikrotik-mcp-server": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "-e", "MIKROTIK_HOST=192.168.88.1",
        "-e", "MIKROTIK_USERNAME=admin",
        "-e", "MIKROTIK_PASSWORD=your_password",
        "-e", "MIKROTIK_PORT=22",
        "-e", "MIKROTIK_MCP__TRANSPORT=stdio",
        "ghcr.io/gcormier/mikrotik-mcp:latest"
      ]
    }
  }
}
```

> **Note:** The `-i` flag is required to keep stdin open so the MCP client can communicate with the container.

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `MIKROTIK_HOST` | MikroTik device IP/hostname | `192.168.88.1` |
| `MIKROTIK_USERNAME` | SSH username | `admin` |
| `MIKROTIK_PASSWORD` | SSH password | _(empty)_ |
| `MIKROTIK_PORT` | SSH port | `22` |
| `MIKROTIK_MCP__TRANSPORT` | Transport type: `sse`, `streamable-http`, `stdio` | `sse` |
| `MIKROTIK_MCP__HOST` | HTTP server listen address | `0.0.0.0` |
| `MIKROTIK_MCP__PORT` | HTTP server listen port | `8000` |
