---
name: Windows MCP Overview
description: This skill should be used when the user asks "what Windows MCP tools are available", "how to automate Windows", "which Windows tool should I use", "Windows MCP overview", "list Windows MCP capabilities", "what can Windows MCP do", "get started with Windows automation", or needs guidance on selecting the right Windows-MCP tool for a task.
version: 1.0.0
---

# Windows MCP Overview

The Windows-MCP server (by CursorTouch) provides 16 tools for comprehensive Windows desktop automation. This skill covers tool selection, capabilities, and high-level usage patterns.

## Prerequisites

- Python 3.13+ installed
- UV package manager (`pip install uv`)
- Windows 7/8/8.1/10/11
- English as the default Windows display language (or disable the App tool for other languages)

## Tool Categories at a Glance

### Screen Interaction
| Tool | Purpose |
|------|---------|
| **Click** | Click at screen coordinates (left/right/double) |
| **Type** | Input text into focused elements with optional clear |
| **Scroll** | Vertical/horizontal scrolling at specific coordinates |
| **Move** | Mouse movement with optional drag support |
| **Shortcut** | Execute keyboard shortcuts (Ctrl+C, Alt+Tab, etc.) |
| **Wait** | Pause execution for a specified duration |

### Visual Capture
| Tool | Purpose |
|------|---------|
| **Screenshot** | Fast desktop capture with cursor and active window info |
| **Snapshot** | Full state capture with interactive element IDs and DOM extraction |

### Application Management
| Tool | Purpose |
|------|---------|
| **App** | Launch, resize, move, switch, minimize, maximize, close windows |

### System Operations
| Tool | Purpose |
|------|---------|
| **Shell** | Execute PowerShell commands |
| **Process** | List running processes or terminate by PID/name |
| **Registry** | Read, write, delete, or list Windows Registry keys/values |
| **Notification** | Send Windows toast notifications |

### Data Operations
| Tool | Purpose |
|------|---------|
| **Scrape** | Extract information from web pages |
| **Clipboard** | Read or write Windows clipboard content |
| **MultiSelect** | Select multiple items (files, folders, checkboxes) |
| **MultiEdit** | Input text into multiple form fields simultaneously |

## Tool Selection Decision Tree

**Need to see the screen?**
- Quick visual check -> `Screenshot`
- Need interactive element IDs -> `Snapshot`
- Need web page DOM content -> `Snapshot` with `use_dom=True`

**Need to interact with GUI elements?**
- Click something -> `Snapshot` first to locate, then `Click`
- Enter text -> `Type` (with `clear: true` to replace existing text)
- Scroll a page/list -> `Scroll` with target coordinates
- Use a keyboard combo -> `Shortcut`
- Drag an item -> `Move` with `drag: true`

**Need to manage applications?**
- Open a program -> `App` (launch)
- Arrange windows -> `App` (resize/move)
- Switch focus -> `App` (switch) or `Shortcut` (Alt+Tab)
- Close a program -> `App` (close) or `Shortcut` (Alt+F4)

**Need system-level operations?**
- Run a command -> `Shell`
- Monitor/kill processes -> `Process`
- Modify system settings -> `Registry`
- Alert the user -> `Notification`

**Need data operations?**
- Extract web content -> `Scrape`
- Transfer data via clipboard -> `Clipboard`
- Select multiple items -> `MultiSelect`
- Fill multiple fields -> `MultiEdit`

## Core Automation Workflow

Most Windows automation follows this cycle:

1. **Observe** — Capture screen state with `Screenshot` or `Snapshot`
2. **Identify** — Locate target elements using `Snapshot` element IDs or coordinates
3. **Act** — Perform the action (`Click`, `Type`, `Shortcut`, etc.)
4. **Verify** — Capture screen again to confirm the action succeeded
5. **Wait** — Use `Wait` between steps if the UI needs time to update

## Performance Notes

- Typical action latency: 0.2-0.9 seconds
- `Screenshot` is significantly faster than `Snapshot`
- `Snapshot` with `use_dom=True` is the slowest but provides the most data
- Batch tools (`MultiSelect`, `MultiEdit`) outperform sequential individual actions
- Prefer `Shell` over GUI interaction for any task that can be scripted

## Additional Resources

### Reference Files

For complete parameter documentation and examples for every tool:
- **`references/tool-reference.md`** — Detailed parameter reference for all 16 Windows MCP tools with usage examples and tips
