# Windows MCP Complete Tool Reference

Detailed parameter documentation and usage examples for all 16 tools provided by the CursorTouch Windows-MCP server.

---

## Click

Click at screen coordinates.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `x` | int | Yes | — | X coordinate on screen |
| `y` | int | Yes | — | Y coordinate on screen |
| `button` | string | No | "left" | Mouse button: "left", "right", "middle" |
| `double_click` | bool | No | false | Perform double-click |

**Examples:**
- Left-click a button: `Click(x=500, y=300)`
- Right-click for context menu: `Click(x=500, y=300, button="right")`
- Double-click to open file: `Click(x=500, y=300, double_click=true)`

---

## Type

Input text into the currently focused element.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `text` | string | Yes | — | Text to type |
| `clear` | bool | No | false | Clear existing content before typing |

**Examples:**
- Type into focused field: `Type(text="Hello World")`
- Replace existing text: `Type(text="New Value", clear=true)`

---

## Scroll

Scroll vertically or horizontally.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `direction` | string | Yes | — | "up", "down", "left", "right" |
| `amount` | int | No | — | Number of scroll units |
| `x` | int | No | — | X coordinate to scroll at |
| `y` | int | No | — | Y coordinate to scroll at |

**Examples:**
- Scroll down in current window: `Scroll(direction="down", amount=3)`
- Scroll within a specific panel: `Scroll(direction="down", amount=5, x=400, y=300)`

---

## Move

Move mouse pointer with optional drag.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `x` | int | Yes | — | Target X coordinate |
| `y` | int | Yes | — | Target Y coordinate |
| `drag` | bool | No | false | Hold mouse button during move |

**Examples:**
- Hover over element: `Move(x=500, y=300)`
- Drag item to new position: `Move(x=800, y=400, drag=true)`

---

## Shortcut

Execute keyboard shortcuts.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `keys` | string | Yes | — | Key combination (e.g., "ctrl+c") |

**Comprehensive shortcut reference:**

### General
| Keys | Action |
|------|--------|
| `ctrl+c` | Copy |
| `ctrl+x` | Cut |
| `ctrl+v` | Paste |
| `ctrl+z` | Undo |
| `ctrl+y` | Redo |
| `ctrl+s` | Save |
| `ctrl+a` | Select all |
| `ctrl+f` | Find |
| `ctrl+h` | Find and replace |
| `ctrl+p` | Print |
| `ctrl+n` | New |
| `ctrl+o` | Open |
| `ctrl+w` | Close tab/document |

### Windows Management
| Keys | Action |
|------|--------|
| `alt+tab` | Switch window |
| `alt+f4` | Close window |
| `win+d` | Show/hide desktop |
| `win+e` | Open File Explorer |
| `win+r` | Open Run dialog |
| `win+l` | Lock screen |
| `win+left` | Snap window left |
| `win+right` | Snap window right |
| `win+up` | Maximize window |
| `win+down` | Minimize/restore window |
| `ctrl+shift+esc` | Open Task Manager |

### Browser-Specific
| Keys | Action |
|------|--------|
| `ctrl+t` | New tab |
| `ctrl+w` | Close tab |
| `ctrl+shift+t` | Reopen closed tab |
| `ctrl+l` | Focus address bar |
| `ctrl+tab` | Next tab |
| `ctrl+shift+tab` | Previous tab |
| `f5` | Refresh |
| `ctrl+shift+i` | Developer tools |

### Text Editing
| Keys | Action |
|------|--------|
| `ctrl+shift+left` | Select word left |
| `ctrl+shift+right` | Select word right |
| `home` | Start of line |
| `end` | End of line |
| `ctrl+home` | Start of document |
| `ctrl+end` | End of document |
| `ctrl+backspace` | Delete word left |
| `ctrl+delete` | Delete word right |

---

## Wait

Pause execution.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `duration` | float | Yes | — | Seconds to wait |

**Recommended durations:**
- Rapid UI action gap: 0.2-0.5s
- Menu/dialog response: 0.3-0.5s
- Window switch settle: 0.5-1s
- App launch warm-up: 1-3s
- Heavy app load (Office, IDE): 3-5s

---

## Screenshot

Fast desktop capture.

**Parameters:** None

**Returns:**
- Desktop image
- Cursor position (x, y)
- Active window title and bounds

---

## Snapshot

Full state capture with element identification.

**Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `use_dom` | bool | No | false | Extract DOM content from web browsers |

**Returns:**
- Desktop image
- Interactive elements (ID, coordinates, type, label)
- Scrollable regions
- DOM content (when `use_dom=True`)

---

## App

Application and window management.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | Yes | "launch", "switch", "resize", "move", "minimize", "maximize", "close" |
| `name` | string | Conditional | Application/window name (for all actions) |
| `path` | string | Conditional | Executable path (for launch) |
| `width` | int | Conditional | Window width (for resize) |
| `height` | int | Conditional | Window height (for resize) |
| `x` | int | Conditional | Window X position (for move) |
| `y` | int | Conditional | Window Y position (for move) |

---

## Shell

Execute PowerShell commands.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `command` | string | Yes | PowerShell command to execute |

**Returns:** stdout, stderr, exit code

---

## Scrape

Extract webpage information.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | Yes | URL to scrape |

**Returns:** Extracted page content as text

---

## MultiSelect

Select multiple items.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `items` | array | Yes | List of items to select (coordinates or identifiers) |

---

## MultiEdit

Input text into multiple fields.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `fields` | array | Yes | List of field identifiers with values |

---

## Clipboard

Read or write clipboard content.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | Yes | "read" or "write" |
| `text` | string | Conditional | Text to write (required for "write") |

---

## Process

Manage running processes.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | Yes | "list" or "kill" |
| `name` | string | Conditional | Process name (for kill) |
| `pid` | int | Conditional | Process ID (for kill) |

---

## Notification

Send Windows toast notifications.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | Yes | Notification title |
| `message` | string | Yes | Notification body text |

---

## Registry

Windows Registry operations.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | Yes | "read", "write", "delete", "list" |
| `path` | string | Yes | Registry key path |
| `name` | string | Conditional | Value name |
| `value` | string | Conditional | Value data (for write) |
| `type` | string | Conditional | Value type: REG_SZ, REG_DWORD, REG_QWORD, REG_EXPAND_SZ, REG_MULTI_SZ |
