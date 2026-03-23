# Advanced Screen Analysis Guide

Strategies for effective screen capture, element identification, and multi-window analysis with Windows MCP.

---

## Element Identification Strategies

### By Element Type
Snapshot results categorize elements by type. Common types:
- **button** — Clickable buttons, toolbar items
- **text_input** / **text_field** — Editable text fields
- **link** — Hyperlinks (in browsers and some apps)
- **checkbox** — Toggle checkboxes
- **radio** — Radio button options
- **dropdown** / **combobox** — Selection dropdowns
- **tab** — Tab controls
- **menu_item** — Menu entries
- **list_item** — Items in lists or trees
- **scrollbar** — Scrollable regions

### By Label Text
Match elements by their visible label to find the correct target:
- Button: look for text matching the action ("Save", "Cancel", "Submit")
- Text field: look for adjacent label text ("Name:", "Email:", "Search")
- Checkbox: look for the option text ("Remember me", "Agree to terms")

### By Position
When labels are ambiguous, use spatial reasoning:
- Form fields are typically arranged top-to-bottom or left-to-right
- Buttons are often at the bottom of forms or dialogs
- Navigation is typically at the top or left side
- Status indicators are often in the bottom-right

---

## Multi-Window Analysis

### Identifying the Active Window
Screenshot and Snapshot both report the active (foreground) window. Check this first to ensure interactions target the correct application.

### Working with Overlapping Windows
```
Scenario: Target element is behind another window
1. Snapshot -> identify the foreground window
2. App(switch, name="target window") -> bring target forward
3. Wait(0.3) -> Snapshot -> now target elements are accessible
```

### Multi-Monitor Awareness
- Primary monitor: coordinates start at (0, 0)
- Secondary monitors: coordinates extend beyond primary bounds
  - Monitor to the right: x values > primary width
  - Monitor to the left: negative x values
  - Monitor above: negative y values

To interact with secondary monitors, use coordinates in the appropriate range.

---

## Web Page Analysis with DOM

### When to Enable DOM Extraction
Enable `Snapshot(use_dom=True)` when:
- Needing to read text content from a web page
- Identifying page structure and element hierarchy
- Finding elements that are not visually interactive but contain data
- Working with dynamically loaded content

### DOM Content Interpretation
The DOM extraction provides:
- HTML element hierarchy
- Text content of elements
- Element attributes (classes, IDs, data attributes)
- Form field values
- Link URLs and destinations

### Combining Visual and DOM Analysis
```
1. Snapshot(use_dom=True) -> get both visual and DOM data
2. Cross-reference DOM content with visual element positions
3. Use DOM text to confirm which visual element is the correct target
4. Act on the element using its visual coordinates
```

---

## Handling Dynamic Content

### Asynchronous Loading
```
Problem: Elements not yet loaded when Snapshot is taken
Solution:
  Wait(1) -> Snapshot -> check if elements present
  If not: Wait(1) -> Snapshot again
  Max 3 attempts before reporting failure
```

### Content That Changes Over Time
```
Problem: Dashboard with auto-refreshing data
Solution:
  Snapshot -> capture current state
  Act immediately on captured coordinates
  Do not re-Snapshot unnecessarily between rapid actions
```

### Animations and Transitions
```
Problem: Elements moving due to animation
Solution:
  Wait(0.5-1) for animation to complete
  Then Snapshot for stable coordinates
```

---

## Efficient Capture Strategies

### Minimize Snapshot Calls
Each Snapshot has a cost. Optimize by:
1. Taking one Snapshot and extracting all needed information
2. Planning multiple actions from a single Snapshot before re-capturing
3. Using Screenshot for verification steps (cheaper than Snapshot)

### Progressive Detail
```
1. Screenshot -> quick visual assessment (is the right window open?)
2. Snapshot -> element identification (where exactly to click?)
3. Snapshot(use_dom=True) -> DOM analysis (what does the page content say?)
```

Only progress to the next level when the previous level's information is insufficient.

### Capture After State Changes
Always re-capture after:
- Clicking a button (may open dialog, navigate, or change content)
- Submitting a form (page may reload)
- Switching windows (different window content)
- Scrolling (new elements may be visible)
- Resizing a window (layout may reflow)

---

## Troubleshooting Capture Issues

### Snapshot Returns No Interactive Elements
**Causes:**
- Window is still loading (Wait and retry)
- Application uses custom rendering (e.g., games, some Java apps)
- Window is minimized or hidden

**Solutions:**
- Add `Wait(1-2)` and retry
- Ensure the window is maximized and focused
- Try `Snapshot(use_dom=True)` for web-based apps
- Fall back to coordinate-based interaction using Screenshot visual inspection

### Screenshot Shows Black or Blank Screen
**Causes:**
- Screen saver is active
- System is locked
- Full-screen app is blocking capture

**Solutions:**
- Use `Shortcut("escape")` or `Move` to wake the screen
- Wait for any full-screen transition to complete

### Coordinates Do Not Match Visual Position
**Causes:**
- DPI scaling mismatch
- Multiple monitors with different resolutions
- Window was moved or resized between Snapshot and action

**Solutions:**
- Always use fresh Snapshot coordinates
- Take a new Snapshot after any window change
- Windows MCP handles DPI scaling automatically — report bugs if coordinates consistently mismatch
