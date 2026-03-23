---
name: Windows Automation Best Practices
description: This skill should be used when the user asks about "Windows automation best practices", "reliable automation", "automation performance", "error handling in automation", "automation workflow design", "Windows MCP tips", "troubleshooting automation", "automation strategy", or needs guidance on designing robust, efficient, and maintainable Windows automation workflows.
version: 1.0.0
---

# Windows Automation Best Practices

This skill provides principles and patterns for building reliable, efficient, and maintainable Windows automation workflows using Windows MCP tools.

## Core Principles

### 1. Prefer CLI Over GUI

When a task can be accomplished via command line, use the `Shell` tool instead of GUI automation.

**Why:** Shell commands are faster, more reliable, and less affected by UI changes.

| Task | GUI Approach (Fragile) | CLI Approach (Preferred) |
|------|----------------------|------------------------|
| Create a folder | File Explorer + right-click + New Folder | `Shell("New-Item -ItemType Directory -Path 'C:\\Folder'")` |
| Copy files | File Explorer drag-and-drop | `Shell("Copy-Item 'source' 'dest'")` |
| Check disk space | Open This PC, read labels | `Shell("Get-CimInstance Win32_LogicalDisk")` |
| Install software | Download + click installer | `Shell("winget install AppName")` |

**Use GUI only when:** no CLI alternative exists, visual verification is essential, or the task inherently requires visual interaction.

### 2. Observe-Act-Verify

Every GUI interaction should follow this cycle:

1. **Observe:** `Snapshot` to identify the current state and target elements
2. **Act:** Perform the interaction (Click, Type, Shortcut, etc.)
3. **Verify:** `Screenshot` to confirm the action produced the expected result

Never chain multiple actions without verifying intermediate results, especially for critical operations.

### 3. Minimize Captures

Use the cheapest observation tool that provides sufficient information:

- `Screenshot` when only a visual check is needed (fast)
- `Snapshot` when element IDs or coordinates are needed (slower)
- `Snapshot(use_dom=True)` only when DOM content is specifically required (slowest)

### 4. Use Smart Waits

Avoid long, arbitrary `Wait()` calls. Instead:

```
# Bad: arbitrary long wait
App(launch "Word") -> Wait(5)

# Good: short wait + verification loop
App(launch "Word") -> Wait(1.5) -> Screenshot -> check if loaded
```

**Recommended wait times:**
- Between rapid UI actions: 0.2-0.5s
- After menu/dialog actions: 0.3-0.5s
- After window switching: 0.5-1s
- After app launch: 1-3s (then verify)

### 5. Handle Failure Gracefully

When an action does not produce the expected result:
1. Take a `Snapshot` to understand the current state
2. Check for unexpected dialogs, popups, or error messages
3. Attempt recovery (dismiss dialog, retry action)
4. If recovery fails, report the issue with a screenshot

## Workflow Design Patterns

### Sequential with Verification
```
Action1 -> Verify1 -> Action2 -> Verify2 -> Action3 -> Verify3
```
Best for critical workflows where each step must succeed.

### Batch Operations
```
Snapshot -> identify all targets
MultiSelect or MultiEdit -> batch action
Verify result
```
Best for repetitive tasks on multiple items.

### Data Pipeline
```
Source: Scrape/Clipboard/Snapshot -> extract data
Transform: process data
Output: Clipboard/Type/Shell -> deliver results
```
Best for data extraction and transfer workflows.

### App Coordination
```
Setup: Launch/switch apps -> arrange windows
Work: interact with each app in sequence
Cleanup: close temporary apps, save results
```
Best for multi-application workflows.

## Performance Optimization

1. **Batch over sequential** — `MultiSelect`/`MultiEdit` over repeated Click/Type
2. **Shell over GUI** — PowerShell commands execute faster than navigating GUI
3. **Screenshot over Snapshot** — Use `Screenshot` for verification, `Snapshot` only when element IDs are needed
4. **Shortcuts over menus** — `Shortcut("ctrl+s")` is faster than navigating File > Save
5. **Skip unnecessary waits** — Verify readiness with `Screenshot` instead of over-waiting
6. **Minimize DOM extraction** — `Snapshot(use_dom=True)` is the most expensive operation

## Common Pitfalls and Solutions

| Pitfall | Solution |
|---------|----------|
| Clicking without Snapshot first | Always Snapshot to get current coordinates |
| Typing into wrong field | Click the target field first to ensure focus |
| Stale coordinates after scroll/resize | Re-Snapshot after any content change |
| Long waits "just in case" | Use short waits + verification |
| Ignoring failed actions | Always verify with Screenshot after actions |
| Using GUI when CLI works | Check if Shell can accomplish the task first |
| Not handling dialogs | Snapshot to detect unexpected popups |
| Hardcoding coordinates | Use Snapshot element IDs when possible |

## Security Considerations

1. **Registry changes** — Always read before write, back up before modifying, prefer HKCU over HKLM
2. **Shell commands** — Avoid running untrusted commands, be cautious with `-Force` flags
3. **Process termination** — Never kill system-critical processes (csrss, lsass, winlogon, svchost)
4. **Sensitive data** — Be aware that Clipboard contents, screenshots, and Shell output may contain passwords, tokens, or personal information
5. **Telemetry** — The MCP server supports `ANONYMIZED_TELEMETRY=false` to disable data collection

## Troubleshooting

**Tool returns no response:**
- Check if Windows MCP server is running (`uvx windows-mcp`)
- Verify Python 3.13+ and UV are installed
- Restart the MCP server

**Click hits wrong target:**
- Re-Snapshot — screen content may have changed
- Check for overlapping windows
- Verify coordinates match the intended element

**App does not launch:**
- Check application name spelling
- Try using the full executable path
- Verify the app is installed with `Shell("where appname")`

**Shell command fails:**
- Check PowerShell syntax (backtick escaping, quotes)
- Verify required permissions (admin vs. user)
- Check if the command requires a specific working directory

**Snapshot shows no elements:**
- The window may still be loading — add `Wait(1)` and retry
- Some custom UI frameworks may not expose accessible elements
- Try `use_dom=True` for web-based applications

## Additional Resources

### Reference Files

For detailed troubleshooting and advanced automation patterns:
- **`references/troubleshooting.md`** — Comprehensive troubleshooting guide, error recovery patterns, and environment setup verification
