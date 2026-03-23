# Advanced Window Management Workflows

Complex multi-window workflows, tiling strategies, and multi-monitor patterns for Windows MCP automation.

---

## Window Tiling Layouts

### Two-Window Side by Side
```
App(switch, name="App1") -> Shortcut("win+left")
App(switch, name="App2") -> Shortcut("win+right")
```

### Three-Window Layout (1 left, 2 stacked right)
```
App(switch, name="MainApp") -> Shortcut("win+left")
App(switch, name="TopRight")
App(resize, name="TopRight", width=960, height=540)
App(move, name="TopRight", x=960, y=0)
App(switch, name="BottomRight")
App(resize, name="BottomRight", width=960, height=540)
App(move, name="BottomRight", x=960, y=540)
```

### Four-Window Grid (Quadrants)
```
App(switch, name="App1") -> Shortcut("win+left") -> Shortcut("win+up")
App(switch, name="App2") -> Shortcut("win+right") -> Shortcut("win+up")
App(switch, name="App3") -> Shortcut("win+left") -> Shortcut("win+down")
App(switch, name="App4") -> Shortcut("win+right") -> Shortcut("win+down")
```

### Full Screen Focus
```
App(switch, name="FocusApp") -> App(maximize, name="FocusApp")
```

---

## Multi-Application Workflows

### Compare Two Documents
```
# Open both documents
App(launch, path="document1.docx") -> Wait(3)
App(launch, path="document2.docx") -> Wait(3)

# Tile side by side
App(switch, name="document1") -> Shortcut("win+left")
App(switch, name="document2") -> Shortcut("win+right")

# Now interact with each as needed
App(switch, name="document1") -> interact
App(switch, name="document2") -> interact
```

### Data Entry from Source to Target
```
# Arrange windows
App(switch, name="Source Spreadsheet") -> Shortcut("win+left")
App(switch, name="Target Form") -> Shortcut("win+right")

# Read from source
App(switch, name="Source Spreadsheet")
Snapshot -> identify data cells
Click(cell) -> Shortcut("ctrl+c")
data = Clipboard(read)

# Write to target
App(switch, name="Target Form")
Snapshot -> identify target field
Click(field) -> Type(text=data)
```

### Browser + IDE Workflow
```
# Arrange windows
App(switch, name="VS Code") -> Shortcut("win+left")
App(switch, name="Chrome") -> Shortcut("win+right")

# Work cycle
App(switch, name="VS Code") -> edit code -> Shortcut("ctrl+s")
App(switch, name="Chrome") -> Shortcut("f5") -> verify changes
```

---

## Application Lifecycle Patterns

### Launch and Configure
```
App(launch, name="Application") -> Wait(3)
Screenshot -> verify launched
Snapshot -> find configuration options
# Set up preferences, workspace, etc.
```

### Graceful Shutdown
```
App(switch, name="Application")
Shortcut("ctrl+s") -> Wait(0.5) -> save work first
App(close, name="Application") -> Wait(0.5)
Screenshot -> check for "save changes?" dialog
If dialog: Click(Save/Don't Save as appropriate)
```

### Restart Application
```
App(close, name="Application") -> Wait(1)
# Verify closed
Process(list) -> check if process still running
If still running: Process(kill, name="application")
Wait(1)
App(launch, name="Application") -> Wait(3)
Screenshot -> verify restarted
```

---

## Window Discovery

### Find Window by Title
When the exact window title is unknown:
```
Snapshot -> check visible windows in the capture
App(switch, name="partial title match")
```

### List All Windows
```
Shell("Get-Process | Where-Object {$_.MainWindowTitle -ne ''} | Select-Object ProcessName, MainWindowTitle")
```
This lists all processes with visible windows and their titles.

### Find Hidden/Minimized Windows
```
Shell("Get-Process | Where-Object {$_.MainWindowHandle -ne 0} | Select-Object ProcessName, MainWindowTitle")
```

---

## Window State Management

### Save Window Positions (for later restoration)
```
Shell("Get-Process | Where-Object {$_.MainWindowTitle -ne ''} | ForEach-Object { Add-Type -AssemblyName System.Windows.Forms; $rect = [System.Windows.Forms.Screen]::FromHandle($_.MainWindowHandle).Bounds; \"$($_.ProcessName): $rect\" }")
```

### Check if Window Exists
```
Shell("Get-Process -Name 'appname' -ErrorAction SilentlyContinue | Where-Object {$_.MainWindowTitle -ne ''}")
```
Returns nothing if the app is not running or has no visible window.

---

## Handling Special Window Types

### Dialog Windows
Dialogs are child windows of their parent app. They typically:
- Appear on top of the parent window
- Block interaction with the parent until dismissed
- Have a limited set of buttons (OK, Cancel, Yes, No, etc.)

```
Snapshot -> identify dialog elements
Click(appropriate_button)
Wait(0.3) -> Screenshot -> verify dialog dismissed
```

### System Tray Applications
Some apps minimize to the system tray instead of the taskbar:
```
Click(system_tray_area) -> Wait(0.3) -> look for hidden icons arrow
Snapshot -> identify tray icon
Click(icon) or Click(icon, button="right") for menu
```

### Full-Screen Applications
```
# Exit full-screen first
Shortcut("f11") or Shortcut("alt+enter") or Shortcut("escape")
Wait(0.5) -> Screenshot -> verify windowed mode
# Now resize/move as needed
```

### UAC (User Account Control) Prompts
UAC prompts may block automation. If detected:
- Inform the user that a UAC prompt needs manual approval
- Avoid automation that triggers UAC when possible
- Prefer operations that do not require elevation

---

## Performance Tips

### Minimize Window Switches
Each `App(switch)` takes 0.3-0.5s. Batch operations in one window before switching:
```
# Good: batch operations per window
App(switch, name="App1") -> action1 -> action2 -> action3
App(switch, name="App2") -> action4 -> action5

# Avoid: frequent switching
App(switch, name="App1") -> action1
App(switch, name="App2") -> action4
App(switch, name="App1") -> action2  # unnecessary switch
```

### Use Shortcuts for Speed
```
# Fast window management
Shortcut("alt+tab")      # faster than App(switch) for recent window
Shortcut("win+d")        # show desktop instantly
Shortcut("win+left/right") # snap without needing coordinates
```
