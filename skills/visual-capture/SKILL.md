---
name: Visual Capture and Analysis
description: This skill should be used when the user asks to "take a screenshot", "capture the screen", "identify elements on screen", "get a snapshot", "extract DOM content", "find interactive elements", "analyze screen state", "what is on the screen", or needs to observe and analyze the current Windows desktop state using Screenshot or Snapshot tools.
version: 1.0.0
---

# Visual Capture and Analysis

This skill covers the two observation tools — Screenshot and Snapshot — which are the foundation of every automation workflow. Before acting on the screen, always observe it first.

## Screenshot vs. Snapshot: When to Use Which

| Criteria | Screenshot | Snapshot |
|----------|-----------|----------|
| Speed | Fast | Slower |
| Visual image | Yes | Yes |
| Cursor position | Yes | Yes |
| Active window info | Yes | Yes |
| Interactive element IDs | No | Yes |
| Element coordinates | No | Yes |
| Scrollable regions | No | Yes |
| DOM extraction | No | Yes (with `use_dom`) |

**Decision rule:**
- Need to verify a result or get a quick look? -> `Screenshot`
- Need to identify elements before clicking/typing? -> `Snapshot`
- Need web page content/structure? -> `Snapshot` with `use_dom=True`

## Screenshot

Fast desktop capture returning a visual image, cursor position, and active window information.

**Takes no parameters** — captures the entire desktop.

**When to use:**
- Verifying the result of an action (did the click work?)
- Checking the general state of the desktop
- Quick visual confirmation between steps
- Monitoring progress of a long operation

**Output includes:**
- Full desktop image
- Current cursor coordinates
- Active (foreground) window title and bounds

## Snapshot

Full state capture with interactive element identification.

**Key parameter:** `use_dom` (boolean, default: false)

**When to use:**
- Before any `Click`, `Type`, or interaction — to locate targets
- When needing to enumerate all interactive elements in a window
- When needing to identify scrollable regions
- When automating web browsers and needing page DOM content

**Output includes:**
- Full desktop image
- List of interactive elements, each with:
  - Unique element ID
  - Screen coordinates (x, y)
  - Element type (button, text field, link, etc.)
  - Element label/text
- Scrollable regions with boundaries
- DOM content (only when `use_dom=True`)

### DOM Extraction (`use_dom=True`)

Enable DOM extraction when automating web browsers. This extracts the page's HTML structure, enabling:
- Reading page content without scraping
- Identifying elements by their HTML attributes
- Understanding page layout and structure
- Finding hidden or off-screen elements

**Performance note:** DOM extraction adds latency. Only enable when specifically needed for web automation.

## Effective Analysis Patterns

### Identify a Button to Click
```
Snapshot -> scan element list for target button
  -> note its (x, y) coordinates
  -> Click(x, y)
```

### Find a Text Field to Fill
```
Snapshot -> scan element list for input/text field elements
  -> identify the field by its label or position
  -> Click(field_x, field_y) -> Type(text="value")
```

### Verify an Action Succeeded
```
Perform action -> Screenshot
  -> check if expected visual change occurred
  -> if not, retry or take a different approach
```

### Analyze a Complex Window
```
Snapshot -> enumerate all elements
  -> categorize by type (buttons, fields, labels)
  -> identify the target interaction area
  -> plan the sequence of actions
```

### Extract Web Page Data
```
Snapshot(use_dom=True) -> read DOM content
  -> extract text, links, or structured data
  -> process the extracted information
```

## Best Practices

1. **Start every automation with Snapshot** — Establish the current state before acting
2. **Use Screenshot for verification** — Cheaper than Snapshot when element IDs are not needed
3. **Re-snapshot after major changes** — Window content may have shifted (new elements, dialogs, navigation)
4. **Do not cache old snapshots** — Screen state changes constantly; always use fresh captures
5. **Minimize DOM extraction** — Only use `use_dom=True` when web content analysis is specifically required
6. **Read element labels carefully** — Match elements by their label text, not just position, to handle UI layout variations

## Handling Common Scenarios

**Multiple monitors:** Screenshot and Snapshot capture the primary display. To interact with secondary monitors, specify coordinates in that monitor's range.

**Overlapping windows:** The Snapshot returns elements for all visible windows. Focus the target window first using `App(switch)` before taking a snapshot.

**Dynamic content:** Elements that load asynchronously may not appear immediately. Use `Wait(0.5-1s)` before Snapshot if content is still loading.

**High DPI displays:** Coordinates are in screen pixels. The tools handle DPI scaling automatically.

## Additional Resources

### Reference Files

For advanced capture techniques and analysis workflows:
- **`references/analysis-guide.md`** — Advanced screen analysis, multi-window strategies, and element identification patterns
