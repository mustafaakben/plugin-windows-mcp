# plugin-windows-mcp

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A Claude Code plugin that integrates the [Windows-MCP](https://github.com/CursorTouch/Windows-MCP) server and provides skills for automating Windows desktops effectively.

## What This Plugin Provides

- **MCP Integration** — Automatically starts the Windows-MCP server, giving Claude Code access to 16 Windows automation tools
- **7 Skills** — Best practices, workflows, and tool guidance that teach Claude how to use those tools effectively

## Prerequisites

- **Python 3.13+**
- **UV package manager** — `pip install uv`
- **Windows 7/8/8.1/10/11**
- **English** as the default Windows display language (or disable the App tool for other languages)

## Installation

**Step 1** — Add the marketplace:

```bash
claude plugin marketplace add mustafaakben/plugin-windows-mcp
```

**Step 2** — Install the plugin:

```bash
claude plugin install plugin-windows-mcp
```

Or from within Claude Code, use the slash commands:

```
/plugin marketplace add mustafaakben/plugin-windows-mcp
/plugin install plugin-windows-mcp
```

## Available Tools (via MCP)

| Tool | Description |
|------|-------------|
| **Click** | Click at screen coordinates (left/right/double) |
| **Type** | Input text with optional clear |
| **Scroll** | Vertical/horizontal scrolling |
| **Move** | Mouse movement with drag support |
| **Shortcut** | Keyboard shortcuts (Ctrl+C, Alt+Tab, etc.) |
| **Wait** | Pause execution |
| **Screenshot** | Fast desktop capture |
| **Snapshot** | Full state capture with element IDs and DOM extraction |
| **App** | Launch, resize, move, switch, close applications |
| **Shell** | Execute PowerShell commands |
| **Process** | List or kill processes |
| **Registry** | Read/write/delete Windows Registry |
| **Notification** | Windows toast notifications |
| **Scrape** | Extract webpage content |
| **Clipboard** | Read/write clipboard |
| **MultiSelect** | Batch item selection |
| **MultiEdit** | Bulk form input |

## Skills

| Skill | Description |
|-------|-------------|
| **Windows MCP Overview** | Tool reference, selection guide, and decision tree |
| **Screen Interaction** | Click, Type, Scroll, Move, Shortcut, Wait patterns |
| **Visual Capture** | Screenshot vs Snapshot, DOM extraction, screen analysis |
| **App & Window Management** | Launch, tile, switch, and arrange application windows |
| **System Operations** | PowerShell, processes, registry, and notifications |
| **Web & Data Operations** | Scraping, clipboard chains, MultiSelect, MultiEdit |
| **Best Practices** | Performance, error handling, security, and workflow design |

## How It Works

When this plugin is installed:

1. The Windows-MCP server starts automatically via `uvx windows-mcp`
2. All 16 tools become available to Claude Code
3. Skills activate automatically based on what you ask Claude to do — no manual invocation needed

For example, asking Claude to "click the save button" will trigger the **Screen Interaction** skill, which teaches Claude to Snapshot first, identify the button coordinates, then Click.

## Configuration

The MCP server runs with telemetry disabled by default. To customize, edit `.mcp.json`:

```json
{
  "mcpServers": {
    "windows-mcp": {
      "command": "uvx",
      "args": ["windows-mcp"],
      "env": {
        "ANONYMIZED_TELEMETRY": "false"
      }
    }
  }
}
```

### Remote Mode

To connect to a remote Windows VM instead of the local machine, add these environment variables:

```json
"env": {
  "MODE": "remote",
  "SANDBOX_ID": "your-sandbox-id",
  "API_KEY": "your-api-key"
}
```

## Versioning

This project follows [Semantic Versioning](https://semver.org/). See [CHANGELOG.md](CHANGELOG.md) for release history.

## License

MIT
