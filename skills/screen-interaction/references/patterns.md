# Screen Interaction Patterns

Advanced multi-step interaction patterns, edge cases, and troubleshooting for GUI automation with Windows MCP.

---

## Multi-Step Form Filling

### Standard Form
```
Snapshot -> identify all form fields and their coordinates
For each field:
  Click(field_x, field_y) -> Wait(0.2) -> Type(text="value", clear=true)
Click(submit_button_x, submit_button_y)
Wait(1) -> Screenshot -> verify submission
```

### Form with Dropdowns
```
Snapshot -> identify dropdown element
Click(dropdown_x, dropdown_y) -> Wait(0.3)
Snapshot -> identify dropdown options
Click(option_x, option_y)
Screenshot -> verify selection
```

### Form with Checkboxes/Radio Buttons
```
Snapshot -> identify checkbox/radio positions
Click(checkbox_x, checkbox_y) -> verify state change with Screenshot
```

### Form with File Upload
```
Snapshot -> identify upload button
Click(upload_button_x, upload_button_y) -> Wait(1)
  -> File dialog opens
Type(text="C:\\path\\to\\file.txt") -> Wait(0.3)
Shortcut("enter") -> Wait(1)
Screenshot -> verify file attached
```

---

## Menu Navigation

### Dropdown Menu
```
Click(menu_trigger_x, menu_trigger_y) -> Wait(0.3)
Snapshot -> identify menu items
Click(item_x, item_y)
```

### Nested Sub-Menu
```
Click(menu_x, menu_y) -> Wait(0.3)
Move(submenu_trigger_x, submenu_trigger_y) -> Wait(0.3)
Snapshot -> identify sub-menu items
Click(subitem_x, subitem_y)
```

### Right-Click Context Menu
```
Click(target_x, target_y, button="right") -> Wait(0.3)
Snapshot -> identify context menu items
Click(item_x, item_y)
```

**Tip:** Menus dismiss on any click outside. If a menu disappears unexpectedly, re-trigger it.

---

## Dialog Handling

### Save Dialog
```
Shortcut("ctrl+s") -> Wait(0.5)
Snapshot -> check for save dialog
If first save (Save As dialog):
  Type(text="filename", clear=true)
  Click(save_button_x, save_button_y) -> Wait(1)
Screenshot -> verify save completed
```

### Confirmation Dialog
```
Action that triggers confirmation -> Wait(0.5)
Snapshot -> identify dialog buttons (OK, Cancel, Yes, No)
Click(appropriate_button_x, appropriate_button_y)
```

### Error Dialog
```
Screenshot -> check for error dialog
If error present:
  Snapshot -> read error message text
  Click(OK_button) or Shortcut("enter") to dismiss
  -> decide on recovery action
```

---

## Text Selection Patterns

### Select All Text
```
Click(text_area_x, text_area_y) -> set focus
Shortcut("ctrl+a")
```

### Select Specific Word
```
Double-click on the word: Click(word_x, word_y, double_click=true)
```

### Select a Range
```
Click(start_x, start_y) -> set cursor at start
Move(end_x, end_y, drag=true) -> drag to select range
```

### Select Line
```
Click at line start: Click(line_start_x, line_y)
Shortcut("home") -> go to line start
Shortcut("shift+end") -> select to end of line
```

---

## Scrolling Strategies

### Scroll to Find an Element
```
Snapshot -> check if target element is visible
If not visible:
  Scroll(direction="down", amount=3, x=window_center_x, y=window_center_y)
  Wait(0.3) -> Snapshot -> check again
Repeat until found or reached end
```

### Scroll to Top/Bottom
```
Shortcut("ctrl+home") -> scroll to top
Shortcut("ctrl+end") -> scroll to bottom
```

### Horizontal Scroll
```
Scroll(direction="right", amount=3, x=scrollable_x, y=scrollable_y)
```

---

## Tab and Window Navigation

### Switch Browser Tabs
```
Shortcut("ctrl+tab") -> next tab
Shortcut("ctrl+shift+tab") -> previous tab
Shortcut("ctrl+1") through Shortcut("ctrl+9") -> specific tab by position
```

### Navigate Browser History
```
Shortcut("alt+left") -> back
Shortcut("alt+right") -> forward
```

### Multiple Windows of Same App
```
Snapshot -> identify available windows by title
App(switch, name="Specific Window Title")
```

---

## Edge Cases and Recovery

### Element Not Found After Snapshot
- The element may be off-screen: `Scroll` to reveal it
- The element may be behind a dialog: dismiss the dialog first
- The element may be loading: `Wait(1)` and re-Snapshot
- The window may have lost focus: `App(switch)` to refocus

### Click Registers on Wrong Element
- Re-Snapshot immediately — UI may have shifted
- Check for overlapping windows or popups
- Try clicking with more precise coordinates (center of the element)

### Type Produces Wrong Characters
- Check keyboard layout (English required for some tools)
- Use `Clipboard(write)` + `Shortcut("ctrl+v")` as fallback for special characters
- Verify the correct field has focus with `Snapshot`

### Shortcut Does Not Work
- Verify the correct window has focus
- Some apps override standard shortcuts — check app-specific bindings
- Try the menu-based alternative if the shortcut is unresponsive

### Drag and Drop Fails
- Increase the `Wait` time between click and move
- Verify both source and destination coordinates with `Snapshot`
- Some UI frameworks require clicking and holding before dragging — use `Move` with `drag=true`
