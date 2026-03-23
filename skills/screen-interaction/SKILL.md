---
name: Screen Interaction with Windows MCP
description: This skill should be used when the user asks to "click a button", "type text", "scroll a page", "move the mouse", "press a keyboard shortcut", "interact with GUI elements", "automate mouse clicks", "fill in a text field", "use keyboard shortcuts on Windows", or needs to perform any direct screen interaction using Windows MCP tools.
version: 1.0.0
---

# Screen Interaction with Windows MCP

This skill covers the six tools for direct GUI interaction: Click, Type, Scroll, Move, Shortcut, and Wait. These tools form the foundation of all visual Windows automation.

## Golden Rule: Observe Before Acting

Never click, type, or scroll blindly. Always capture the screen state first:

1. Use `Snapshot` to get interactive element IDs and coordinates
2. Identify the target element from the snapshot results
3. Perform the interaction using the element's coordinates
4. Verify the result with `Screenshot` or another `Snapshot`

## Click

Perform mouse clicks at screen coordinates.

**Key parameters:** `x`, `y` (required), `button` (left/right/middle), `double_click` (boolean)

**Patterns:**
- Single left-click: activate buttons, select items, set focus
- Double-click: open files, select words in text
- Right-click: open context menus

**Workflow:**
```
Snapshot -> identify element coordinates -> Click(x, y) -> Screenshot to verify
```

## Type

Input text into the currently focused element.

**Key parameters:** `text` (required), `clear` (boolean — clears field before typing)

**Patterns:**
- New input: `Click` on field, then `Type(text="value")`
- Replace existing text: `Click` on field, then `Type(text="new value", clear=true)`
- Multi-line input: include newline characters in the text string

**Important:** Always ensure the target field has focus (via `Click`) before calling `Type`. Without focus, text may be typed into the wrong element.

## Scroll

Scroll vertically or horizontally within a window or region.

**Key parameters:** `direction` (up/down/left/right), `amount`, `x`/`y` (coordinates)

**Patterns:**
- Scroll within a specific panel: provide `x`/`y` coordinates inside that panel
- Page through a document: use larger `amount` values
- Scroll to reveal hidden content: moderate `amount`, then verify with `Screenshot`

**Best practice:** Specify coordinates to target a particular scrollable area. Without coordinates, scrolling applies to the window under the cursor.

## Move

Move the mouse pointer, with optional drag support.

**Key parameters:** `x`, `y` (required), `drag` (boolean)

**Patterns:**
- Hover: `Move(x, y)` — triggers tooltips, hover menus, highlights
- Drag and drop: `Click` at source, then `Move(x, y, drag=true)` to destination
- Slider adjustment: `Click` on slider handle, `Move` with `drag=true`

## Shortcut

Execute keyboard shortcuts — often more reliable than GUI clicks.

**Key parameters:** `keys` (required — e.g., "ctrl+c", "alt+tab")

**Essential Windows shortcuts:**
| Shortcut | Action |
|----------|--------|
| `ctrl+c` / `ctrl+v` | Copy / Paste |
| `ctrl+z` / `ctrl+y` | Undo / Redo |
| `ctrl+s` | Save |
| `ctrl+a` | Select all |
| `alt+tab` | Switch window |
| `alt+f4` | Close window |
| `win+d` | Show desktop |
| `win+e` | Open File Explorer |
| `ctrl+shift+esc` | Task Manager |
| `win+r` | Run dialog |
| `ctrl+w` | Close tab |
| `ctrl+t` | New tab (browsers) |

**When to prefer Shortcut over Click:**
- Menu actions (File > Save -> prefer `ctrl+s`)
- Tab/window switching (prefer `alt+tab`)
- Text operations (prefer `ctrl+c`, `ctrl+v`, `ctrl+a`)
- Any action with a well-known keyboard shortcut

## Wait

Pause execution to allow the UI to catch up.

**Key parameter:** `duration` (seconds)

**Recommended wait times:**
| Scenario | Duration |
|----------|----------|
| Between rapid UI actions | 0.2-0.5s |
| After clicking a menu item | 0.3-0.5s |
| After switching tabs/windows | 0.5-1s |
| After launching an application | 1-3s |
| After triggering a dialog/popup | 0.5-1s |
| After saving a large file | 1-2s |

**Prefer verification over long waits.** Instead of `Wait(5)`, use `Wait(1)` + `Screenshot` to check if the UI has updated, then proceed or wait more.

## Common Interaction Patterns

### Fill a Form
```
Snapshot -> identify fields
Click(field1_x, field1_y) -> Type(text="value1")
Click(field2_x, field2_y) -> Type(text="value2")
Click(submit_x, submit_y)
Screenshot -> verify submission
```

### Navigate a Menu
```
Click(menu_x, menu_y) -> Wait(0.3)
Snapshot -> identify menu items
Click(item_x, item_y)
```

### Copy Text from One App to Another
```
App(switch to source) -> Wait(0.5)
Shortcut("ctrl+a") -> Shortcut("ctrl+c")
App(switch to target) -> Wait(0.5)
Click(target_field_x, target_field_y)
Shortcut("ctrl+v")
```

### Drag and Drop
```
Snapshot -> identify source and destination
Click(source_x, source_y)
Move(dest_x, dest_y, drag=true)
Screenshot -> verify result
```

## Error Recovery

- **Clicked wrong element:** Use `Shortcut("ctrl+z")` to undo if possible, then retry
- **Typed in wrong field:** Use `Shortcut("ctrl+z")`, click the correct field, retry
- **Menu disappeared:** Click the menu trigger again, use `Wait(0.3)` before clicking items
- **Dialog appeared unexpectedly:** Use `Snapshot` to identify dialog buttons, dismiss or respond

## Additional Resources

### Reference Files

For detailed interaction patterns and advanced techniques:
- **`references/patterns.md`** — Common multi-step interaction patterns, edge cases, and troubleshooting
