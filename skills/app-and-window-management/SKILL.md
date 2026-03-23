---
name: App and Window Management
description: This skill should be used when the user asks to "launch an application", "open a program", "resize a window", "move a window", "switch between windows", "arrange windows", "minimize or maximize a window", "close an application", "manage desktop layout", or needs to control application windows on Windows using the App tool.
version: 1.0.0
---

# App and Window Management

This skill covers the App tool for launching, arranging, switching, and managing application windows on Windows.

## App Tool Actions

The App tool supports these actions: **launch**, **switch**, **resize**, **move**, **minimize**, **maximize**, **close**.

### Launch

Start an application by name or path.

**Parameters:** `action: "launch"`, plus `name` (app name) or `path` (full executable path)

**By name (common apps):**
```
App(action="launch", name="Notepad")
App(action="launch", name="Chrome")
App(action="launch", name="File Explorer")
App(action="launch", name="Calculator")
```

**By path (specific executables):**
```
App(action="launch", path="C:\\Program Files\\MyApp\\app.exe")
```

**After launching:** Always use `Wait(1-3)` — applications take time to initialize before they are interactive.

### Switch

Bring an existing window to the foreground.

**Parameters:** `action: "switch"`, `name` (window title or partial match)

```
App(action="switch", name="Document.docx - Word")
App(action="switch", name="Chrome")
```

**Tip:** Use partial window titles — the tool matches substrings. For precise targeting, include enough of the title to be unique.

**Alternative:** `Shortcut("alt+tab")` for quick cycling between the two most recent windows.

### Resize

Change window dimensions.

**Parameters:** `action: "resize"`, `name`, `width`, `height`

```
App(action="resize", name="Notepad", width=800, height=600)
```

### Move

Reposition a window on the screen.

**Parameters:** `action: "move"`, `name`, `x`, `y`

```
App(action="move", name="Notepad", x=100, y=100)
```

### Minimize / Maximize / Close

**Parameters:** `action` + `name`

```
App(action="minimize", name="Notepad")
App(action="maximize", name="Chrome")
App(action="close", name="Calculator")
```

**Alternative for close:** `Shortcut("alt+f4")` closes the currently focused window.

## Common Workflows

### Open App and Interact
```
App(launch "Notepad") -> Wait(2)
Snapshot -> verify Notepad is open and focused
Type(text="Hello World")
Shortcut("ctrl+s") -> Wait(0.5) -> handle save dialog
```

### Side-by-Side Window Layout
```
App(resize "App1", width=960, height=1080)
App(move "App1", x=0, y=0)
App(resize "App2", width=960, height=1080)
App(move "App2", x=960, y=0)
```

**Alternative:** Use Windows Snap shortcuts:
```
App(switch "App1") -> Shortcut("win+left")
App(switch "App2") -> Shortcut("win+right")
```

### Multi-App Data Transfer
```
App(switch to source app) -> Wait(0.5)
Select content -> Shortcut("ctrl+c")
App(switch to target app) -> Wait(0.5)
Click(target_field) -> Shortcut("ctrl+v")
```

### Clean Up Desktop
```
App(minimize "App1")
App(minimize "App2")
App(close "TempApp")
```

## Window Management Best Practices

1. **Wait after launch** — Always `Wait(1-3)` after `App(launch)` before interacting
2. **Verify focus** — After `App(switch)`, take a `Screenshot` to confirm the correct window is foreground
3. **Use Snap shortcuts** — `win+left`, `win+right`, `win+up` (maximize) are reliable for tiling
4. **Handle "not found"** — If `App(switch)` fails, the window may have a different title. Use `Snapshot` to check visible windows or `Process(list)` to verify the app is running
5. **Prefer Shortcut for switch** — `alt+tab` is fast for toggling between two apps
6. **Full-screen apps** — Some apps (games, media) may not respond to `resize`/`move`. Use `Shortcut("alt+enter")` to toggle full-screen first

## Application Launch Reference

| Application | Launch Name | Notes |
|-------------|-------------|-------|
| Notepad | `Notepad` | Built-in text editor |
| File Explorer | `File Explorer` or `explorer` | File browser |
| Calculator | `Calculator` | Built-in calculator |
| Chrome | `Chrome` | Google Chrome browser |
| Edge | `Edge` | Microsoft Edge browser |
| Firefox | `Firefox` | Mozilla Firefox browser |
| Command Prompt | `cmd` | Use Shell tool instead when possible |
| PowerShell | `PowerShell` | Use Shell tool instead when possible |
| Task Manager | — | Use `Shortcut("ctrl+shift+esc")` |

## When to Use App vs. Other Tools

- **Need to run a command?** -> Use `Shell` directly, not App to open PowerShell
- **Need to open a file?** -> Use `Shell` with `Start-Process "file.txt"` or `App(launch)` the associated program
- **Need to check if an app is running?** -> Use `Process(list)` instead of visual checks
- **Need to switch quickly?** -> `Shortcut("alt+tab")` is faster than `App(switch)` for recent windows

## Additional Resources

### Reference Files

For advanced window management patterns and multi-monitor workflows:
- **`references/workflows.md`** — Complex multi-window workflows, multi-monitor strategies, and automation patterns
