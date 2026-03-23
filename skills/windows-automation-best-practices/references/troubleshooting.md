# Windows MCP Troubleshooting Guide

Comprehensive troubleshooting, error recovery, and environment verification for Windows MCP automation.

---

## Environment Setup Verification

### Check Prerequisites
```powershell
# Python version (must be 3.13+)
python --version

# UV installed
uv --version

# Windows MCP available
uvx windows-mcp --help

# Windows version
[System.Environment]::OSVersion.Version
```

### First-Run Notes
- First installation may take 1-2 minutes for dependency setup
- Windows display language should be English for full App tool support
- If using non-English Windows, the App tool may have limited functionality

### Verify MCP Server Running
If tools are not responding, verify the server:
```powershell
# Check if windows-mcp process exists
Get-Process | Where-Object { $_.ProcessName -like "*windows*mcp*" -or $_.CommandLine -like "*windows-mcp*" }
```

---

## Common Errors and Solutions

### "Element not found" After Snapshot

**Symptoms:** Snapshot returns elements, but clicking coordinates does nothing or clicks wrong target.

**Causes and fixes:**
1. **Window moved/resized since Snapshot** — Re-Snapshot to get fresh coordinates
2. **Overlay or popup appeared** — Screenshot to check for overlapping windows, dismiss any popups
3. **Element is off-screen** — Scroll to reveal the element, then re-Snapshot
4. **DPI scaling issue** — Rare, but restart MCP server if coordinates are consistently offset

### "Window not found" with App Tool

**Symptoms:** `App(switch, name="...")` fails to find the window.

**Causes and fixes:**
1. **Wrong window title** — Titles may differ from app name. Use Shell to list actual titles:
   ```powershell
   Get-Process | Where-Object { $_.MainWindowTitle -ne '' } | Select-Object ProcessName, MainWindowTitle
   ```
2. **App minimized to tray** — Some apps (Slack, Discord, etc.) minimize to system tray, not taskbar
3. **App not running** — Verify with `Process(list)`, then `App(launch)` if needed
4. **Multiple windows** — Be more specific with the title substring

### Shell Command Fails

**Symptoms:** Shell returns error or unexpected output.

**Causes and fixes:**
1. **Syntax error** — PowerShell uses backtick for escaping, not backslash
2. **Missing module** — Install with `Install-Module ModuleName`
3. **Insufficient permissions** — Some commands require admin elevation
4. **Command not found** — Check with `Get-Command "cmdname"` or verify PATH
5. **Interactive prompt blocking** — Add `-Force` or `-Confirm:$false` flags

### Click Does Not Register

**Symptoms:** Click appears to execute but nothing happens on screen.

**Causes and fixes:**
1. **Wrong coordinates** — Re-Snapshot and verify target position
2. **Element is disabled/grayed out** — Check element state in Snapshot
3. **Click intercepted** — A transparent overlay or tooltip may be catching the click. Move mouse away first, then click
4. **Timing issue** — Add `Wait(0.3)` before the click for the UI to settle

### Type Outputs Wrong Text

**Symptoms:** Typed text is garbled, missing characters, or appears in wrong location.

**Causes and fixes:**
1. **Wrong field has focus** — Click the target field explicitly before typing
2. **Keyboard layout mismatch** — Windows MCP assumes English keyboard layout
3. **Special characters** — Use `Clipboard(write)` + `Shortcut("ctrl+v")` for special characters
4. **Input too fast** — Rare; add `Wait(0.1)` if characters are dropped

### Screenshot/Snapshot Returns Black Screen

**Symptoms:** Capture shows black or blank image.

**Causes and fixes:**
1. **Screen locked** — Cannot automate while screen is locked
2. **Screen saver active** — Move mouse or press a key to wake
3. **Secure desktop (UAC)** — UAC prompts run on a separate desktop that cannot be captured
4. **Full-screen exclusive app** — Some games use exclusive full-screen mode. Switch to windowed mode

---

## Error Recovery Patterns

### Undo Last Action
```
Shortcut("ctrl+z") -> Screenshot -> check if undone successfully
```

### Dismiss Unexpected Dialog
```
Snapshot -> identify dialog elements
If "OK" button: Click(ok_button)
If "Cancel" button: Click(cancel_button)
If "X" close button: Click(close_button)
Fallback: Shortcut("escape")
```

### Recover From Wrong Window Focus
```
Snapshot -> identify which window is active
App(switch, name="intended target") -> Wait(0.3)
Screenshot -> verify correct window is now active
```

### Restart Automation After Failure
```
# Assess current state
Screenshot -> understand where things went wrong
Snapshot -> identify available actions

# Clean up partial state
If form partially filled: consider Shortcut("ctrl+z") or navigate away
If wrong app open: App(close) or App(switch)
If dialog blocking: dismiss it

# Restart from a known good state
Navigate to starting point
Begin automation sequence again
```

---

## Performance Troubleshooting

### Automation Running Slowly

**Possible causes:**
1. **Too many Snapshots** — Use Screenshot for verification, Snapshot only for element discovery
2. **Unnecessary DOM extraction** — Only use `use_dom=True` when web content is needed
3. **Excessive waits** — Reduce wait times, use verification instead
4. **System resource contention** — Check CPU/memory with `Process(list)` or Shell

**Optimization checklist:**
- Replace `Snapshot` with `Screenshot` where element IDs are not needed
- Remove `use_dom=True` unless specifically needed
- Reduce `Wait` durations to minimum necessary
- Use `Shell` instead of GUI for scriptable tasks
- Use `MultiSelect`/`MultiEdit` instead of sequential operations

### High Memory Usage

```powershell
# Check top memory consumers
Get-Process | Sort-Object WorkingSet64 -Descending | Select-Object -First 10 Name, @{N='MemMB';E={[math]::Round($_.WorkingSet64/1MB,1)}}
```

If the MCP server itself is using excessive memory, restart it.

---

## System-Critical Process Safety

**Never terminate these processes:**
| Process | Role |
|---------|------|
| `csrss` | Client/Server Runtime |
| `lsass` | Local Security Authority |
| `winlogon` | Windows Logon |
| `svchost` | Service Host (many instances) |
| `smss` | Session Manager |
| `wininit` | Windows Initialization |
| `services` | Service Control Manager |
| `dwm` | Desktop Window Manager |
| `explorer` | Windows Shell (killing restarts the taskbar/desktop) |

**Safe to terminate:**
- User applications (browsers, editors, media players)
- Non-essential background apps
- Hung or frozen applications

---

## Registry Safety Checklist

Before any Registry modification:

1. **Read current value** — `Registry(read, path, name)` to know what will be changed
2. **Back up the key** — `Shell("reg export 'HKCU\\Software\\MyKey' backup.reg")`
3. **Verify the path** — Typos in registry paths can create junk keys
4. **Use correct type** — REG_SZ for strings, REG_DWORD for integers
5. **Verify after write** — `Registry(read)` to confirm the change applied correctly
6. **Prefer HKCU** — Changes under HKCU affect only the current user, HKLM affects all users and requires elevation

---

## Transport Mode Troubleshooting

### stdio (Default)
- Standard mode, no additional configuration needed
- If not working, verify `uvx` is in PATH

### SSE Mode
```bash
uvx windows-mcp --transport sse --host localhost --port 8000
```
- Verify port is not in use: `Shell("netstat -ano | findstr :8000")`
- Check firewall rules if connecting from another machine

### Streamable HTTP Mode
```bash
uvx windows-mcp --transport streamable-http --host localhost --port 8000
```
- Recommended for production use
- Same port/firewall considerations as SSE

---

## Getting Help

- **GitHub Issues:** https://github.com/CursorTouch/Windows-MCP/issues
- **Discord Community:** discord.gg/Aue9Yj2VzS
- **Twitter:** @CursorTouch
