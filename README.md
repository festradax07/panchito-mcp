# panchito-mcp

Minimal, read-only MCP server for low-resource Linux hosts (Raspberry Pi, small VMs).

*Barato, rápido y siempre a mano, como un pancho de Constitución.*

> **Status:** work in progress — v1 under development.

## What it is

`panchito-mcp` is a [Model Context Protocol](https://modelcontextprotocol.io) server that lets an AI assistant (Claude or any MCP-compatible client) **observe** a Linux host: system health, failed services, bounded logs and local service stats.

It is designed around three constraints:

- **Small footprint.** Built to run comfortably on a Raspberry Pi 3 B+ (1 GB RAM). Resource usage is measured and documented, not assumed.
- **Read-only by design.** No tool can modify the host. Observing is safe; acting is out of scope for v1.
- **Few, well-designed tools.** A small, predictable tool surface instead of dozens of loosely defined ones.

## What it is not

- Not a remote administration tool. It cannot restart services, run commands or change configuration.
- Not a monitoring stack replacement. It complements Prometheus/Grafana-style tooling; it doesn't replace it.

## Planned tools (v1)

| Tool | Description |
| --- | --- |
| `get_system_status` | CPU, memory, disk, load average, uptime, temperature |
| `list_failed_services` | systemd units in a failed state |
| `tail_journal` | Last N lines of a whitelisted systemd unit's journal |
| `get_pihole_stats` | Summary stats from a local Pi-hole instance (optional) |

All tools validate their inputs and truncate their outputs.

## Requirements

- Linux with systemd
- Python 3.11+

## Quick start

```bash
git clone https://github.com/<your-user>/panchito-mcp.git
cd panchito-mcp
python3 -m venv .venv && source .venv/bin/activate
pip install -e .
cp config.example.yaml config.yaml   # edit with your own settings
```

### Connecting to an MCP client (stdio)

```json
{
  "mcpServers": {
    "panchito": {
      "command": "/path/to/panchito-mcp/.venv/bin/panchito-mcp",
      "args": ["--config", "/path/to/config.yaml"]
    }
  }
}
```

## Security model

- Runs as a dedicated, unprivileged user.
- Every tool is read-only.
- Tool arguments are validated against explicit allowlists (e.g. which systemd units can be read).
- Output is bounded to avoid flooding the model's context.
- Tool output is treated as data. Logs may contain untrusted text, which is a prompt-injection vector; the server never interprets it.

See [`docs/security.md`](docs/security.md) (coming soon) for the full threat model.

## Resource usage

To be measured on a Raspberry Pi 3 B+ and documented here: idle RSS, RSS under load and latency per tool.

## Roadmap

- [ ] **v1** — stdio transport, read-only tools, config file, tests
- [ ] **v2** — Streamable HTTP transport, systemd unit with hardening, auth behind a reverse proxy
- [ ] **v3** — Optional Go implementation and footprint comparison

## Design decisions

Architecture decisions are recorded in [`docs/adr/`](docs/adr/).

## License

MIT
