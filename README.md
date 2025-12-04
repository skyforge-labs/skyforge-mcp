# Skyforge MCP Server

> ⚠️ **ALPHA RELEASE** - This is an early alpha version. Expect bugs and breaking changes.
>
> 🚫 **NOT FOR PRODUCTION** - This is a development/experimental version. For a production implementation, please contact [james@skyforge-labs.com](mailto:james@skyforge-labs.com)
>
> 🔓 **NO AUTHENTICATION** - This server has no built-in authentication. CORS is wide open (`allow_origins=["*"]`). Use at your own risk and secure your deployment appropriately.

A Model Context Protocol (MCP) server that connects AI assistants to SkySpark and Haxall building automation systems. Dynamically exposes your SkySpark Axon functions as MCP tools.

## Features

- **Dynamic Axon Tools** - Fetches tool definitions from SkySpark at runtime
- **Prompt Support** - Expose templated prompts from SkySpark
- **Dual Transport** - Supports stdio (Claude Desktop) and HTTP/SSE (web clients)
- **Type Safety** - Full Haystack type system with automatic JSON Schema conversion
- **Docker Ready** - Simple Docker deployment included

## How It Works

The server fetches tools from SkySpark on each `list_tools` request. This means:

- Add new tools by creating Axon functions in SkySpark
- No server restart needed for schema changes
- SkySpark is your single source of truth

## Quick Start

### Prerequisites

- SkySpark or Haxall server with API access
- Docker (recommended) OR Python 3.12+ with [uv](https://docs.astral.sh/uv/)

### Install from PyPI

Once published, you can install directly:

```bash
pip install skyforge-mcp

# Run stdio entry (console script)
skyforge-mcp-stdio

# Or run package via module (also stdio)
python -m skyforge_mcp
```

Environment variables (required by all modes):

Create a file named `.env` in the project folder, then paste:

```
SKYSPARK_URI=http://host.docker.internal:8082/api/demo
SKYSPARK_USERNAME=su
SKYSPARK_PASSWORD=su
```

Done. (Works on Windows, macOS, and Linux.)

### Quick Setup with Example Tools

For immediate testing, import the included `setup.zinc` file into your SkySpark project. This provides example MCP tools and the required `fetchMcpTools()` function.

### Docker Setup (Easiest)

1. **Clone and configure**

   ```bash
   git clone https://github.com/yourusername/skyforge-mcp.git
   cd skyforge-mcp
   ```

   Create a file named `.env` in the project folder, then paste:

   ```
   SKYSPARK_URI=http://host.docker.internal:8082/api/demo
   SKYSPARK_USERNAME=su
   SKYSPARK_PASSWORD=su
   ```

   Done. (Works on Windows, macOS, and Linux.)

2. **Start server**

   ```bash
   docker-compose up --build
   ```

   Server runs on `http://localhost:8000/mcp`

3. **Test with MCP Inspector**
   ```bash
   npx @modelcontextprotocol/inspector docker exec -it skyspark-mcp-server uv run main.py
   ```

### Local Setup (Development)

1. **Install and run**

   ```bash
   # Install uv package manager
   curl -LsSf https://astral.sh/uv/install.sh | sh

   # Clone and setup
   git clone https://github.com/yourusername/skyforge-mcp.git
   cd skyforge-mcp
   uv sync
   ```

   Create a file named `.env` in the project folder, then paste:

   ```
   SKYSPARK_URI=http://localhost:8082/api/demo
   SKYSPARK_USERNAME=su
   SKYSPARK_PASSWORD=su
   ```

   Done. (Works on Windows, macOS, and Linux.)

   ```bash
   # Run HTTP/SSE mode (for web clients)
   uv run main.py

   # Or stdio mode directly (same as pip/console-script behavior)
   uv run python -m skyforge_mcp
   ```

## Claude Desktop Integration (stdio)

This server is designed to run as an MCP stdio server when used with Claude Desktop. You can run it through Docker Compose.

Edit your Claude Desktop config:

- Windows: `%APPDATA%\Claude\claude_desktop_config.json`
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`

Pick one of the options below.

### Option A — Use cwd (project working directory)

```json
{
  "mcpServers": {
    "skyforge-mcp": {
      "type": "stdio",
      "command": "docker",
      "args": [
        "compose",
        "run",
        "--rm",
        "skyforge-mcp",
        "uv",
        "run",
        "skyforge-mcp-stdio"
      ],
      "cwd": "C:\\\\Users\\\\YOUR_USER\\\\Documents\\\\SkyForge labs\\\\GitHub\\\\skyforge-mcp",
      "env": {
        "SKYSPARK_URI": "http://host.docker.internal:8082/api/demo",
        "SKYSPARK_USERNAME": "su",
        "SKYSPARK_PASSWORD": "su"
      }
    }
  }
}
```

### Option B — No cwd: use --project-directory (no absolute file path)

```json
{
  "mcpServers": {
    "skyforge-mcp": {
      "type": "stdio",
      "command": "docker",
      "args": [
        "compose",
        "--project-directory",
        "C:\\\\path\\\\to\\\\skyforge-mcp",
        "run",
        "--rm",
        "skyforge-mcp",
        "uv",
        "run",
        "skyforge-mcp-stdio"
      ],
      "env": {
        "SKYSPARK_URI": "http://host.docker.internal:8082/api/demo",
        "SKYSPARK_USERNAME": "su",
        "SKYSPARK_PASSWORD": "su"
      }
    }
  }
}
```

Notes:

- Replace the project directory with the path to your cloned repo (Windows shown; macOS/Linux use `/path/to/skyforge-mcp`).
- On Windows JSON, backslashes must be escaped (`\\`).
- Restart Claude Desktop after saving the config.

### Cursor MCP (stdio) configuration

Add to Cursor settings (MCP servers). This uses the PyPI package if installed:

```json
{
  "mcpServers": {
    "skyforge-mcp": {
      "command": "python",
      "args": ["-m", "skyforge_mcp"],
      "env": {
        "SKYSPARK_URI": "http://host.docker.internal:8082/api/demo",
        "SKYSPARK_USERNAME": "su",
        "SKYSPARK_PASSWORD": "su"
      }
    }
  }
}
```

## Cursor MCP Integration

Add this to your Cursor MCP configuration to run via stdio:

```json
{
  "mcpServers": {
    "skyforge-mcp": {
      "command": "python",
      "args": ["-m", "skyforge_mcp.stdio"],
      "env": {
        "SKYSPARK_URI": "http://host.docker.internal:8082/api/demo",
        "SKYSPARK_USERNAME": "su",
        "SKYSPARK_PASSWORD": "su"
      }
    }
  }
}
```

Alternatively, after `pip install skyforge-mcp`, you can use the console script:

```json
{
  "mcpServers": {
    "skyforge-mcp": {
      "command": "skyforge-mcp-stdio",
      "env": {
        "SKYSPARK_URI": "http://host.docker.internal:8082/api/demo",
        "SKYSPARK_USERNAME": "su",
        "SKYSPARK_PASSWORD": "su"
      }
    }
  }
}
```

## Creating SkySpark Tools

In SkySpark, implement `fetchMcpTools()` to return tool definitions as a grid. Each row should have:

- `name` - Tool identifier (Str)
- `dis` - Display name (Str)
- `help` - Description (Str)
- `params` - Parameter schema (Dict or List)

**Example in SkySpark:**

```axon
// Return MCP tools grid
fetchMcpTools: () => [
  {
    name: "getSiteEquips",
    dis: "Get Site Equipment",
    help: "Returns all equipment for a site",
    params: {
      kind: "Dict",
      params: {
        siteId: {
          kind: "Ref",
          help: "Site reference ID",
          required: marker()
        }
      }
    }
  }
].toGrid

// Tool implementation (called via `call()`)
getSiteEquips: (dict) => readAll(equip and siteRef == dict->siteId)
```

Import the included `setup.zinc` file into your SkySpark project for example tools and the required `fetchMcpTools()` function.

The server fetches tools automatically when clients call `list_tools`.

## Configuration

Create `.env` file (works on Windows/macOS/Linux):

```
# For Docker on host machine:
SKYSPARK_URI=http://host.docker.internal:8082/api/demo

# Or for local development:
# SKYSPARK_URI=http://localhost:8082/api/demo

SKYSPARK_USERNAME=su
SKYSPARK_PASSWORD=su
```

All three variables are required.

## Project Structure

```
skyforge-mcp/
├── app/
│   ├── skyspark/         # SkySpark integration
│   │   ├── client.py     # Phable-based API client
│   │   ├── converters.py # Haystack ↔ JSON Schema conversion
│   │   ├── grid.py       # HGrid wrapper for dual format output
│   │   └── types.py      # Extended Haystack types
│   └── tools/
│       └── axon_tools.py # Hardcoded tool examples
├── main.py              # MCP server entry point
├── docker-compose.yml   # Docker setup
└── Dockerfile           # Container definition
```

## Troubleshooting

**Connection errors:**

- **Docker**: Use `host.docker.internal` instead of `localhost` in SKYSPARK_URI
- Verify SkySpark URI is accessible: `curl http://your-server:8080/api/demo`
- Check `.env` credentials
- Ensure SkySpark API is enabled

**No tools appearing:**

- Verify `fetchMcpTools()` function exists in SkySpark
- Check server logs: `docker-compose logs` or `uv run main.py`
- Test with MCP Inspector

**Docker issues:**

```bash
docker-compose logs              # View logs
docker-compose restart           # Restart
docker-compose up --build        # Rebuild
```

## Security Notes

⚠️ **Important:**

- **This is NOT for production use** - if you are interested in a production implementation, contact [james@skyforge-labs.com](mailto:james@skyforge-labs.com)
- No built-in authentication - secure your network/deployment
- CORS allows all origins - intended for local development
- Store credentials securely (`.env` files, environment variables)
- For production, add authentication middleware or use VPN/firewall

## Credits & License

**Built with:**

- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk) - Model Context Protocol implementation
- [Phable](https://github.com/rick-jennings/phable) - Haystack/SkySpark client library by Rick Jennings
- [Project Haystack](https://project-haystack.org/) - Building automation data standard

**License:** MIT - see LICENSE file

## Contributing

Issues and PRs welcome! This is an alpha release - feedback appreciated.

**Repository:** [GitHub](https://github.com/skyforge-labs/skyforge-mcp)

## Operations

The most common commands and URLs in one place.

### Environment

- Copy `.env.example` to `.env` and set:
  - `SKYSPARK_URI` (API path, not `/ui`; no trailing slash)
  - `SKYSPARK_USERNAME`
  - `SKYSPARK_PASSWORD`

Example:

```bash
SKYSPARK_URI=http://host.docker.internal:8082/api/demo
SKYSPARK_USERNAME=su
SKYSPARK_PASSWORD=su
```

### Run (PyPI install)

```bash
pip install -U skyforge-mcp

# STDIO server
skyforge-mcp-stdio
# or
python -m skyforge_mcp

# HTTP/SSE server on :8000 (/mcp and /mcp/messages)
skyforge-mcp
```

### Run (Docker)

```bash
docker compose up --build
# HTTP/SSE available at http://localhost:8000/mcp
```

### Run (uv local dev)

```bash
uv sync
uv run main.py             # HTTP/SSE on :8000
python -m skyforge_mcp     # STDIO
```

### MCP Inspector

- Streamable HTTP URL: `http://localhost:8000/mcp`
- STDIO:
  - Command: path to your `python.exe`
  - Arguments: `-m skyforge_mcp`

### Notes on dependencies

- The `phable` library is pinned to `0.1.20` (known‑good) for reproducible behavior. If you upgrade and see API changes, revert to `0.1.20` or test thoroughly.
