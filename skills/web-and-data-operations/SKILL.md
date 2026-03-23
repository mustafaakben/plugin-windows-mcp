---
name: Web and Data Operations
description: This skill should be used when the user asks to "scrape a webpage", "extract web content", "read the clipboard", "write to clipboard", "copy and paste", "select multiple files", "batch select items", "fill multiple form fields", "multi-edit", or needs to extract data from web pages, manage clipboard content, or perform bulk selection and input operations on Windows.
version: 1.0.0
---

# Web and Data Operations

This skill covers four data-oriented tools: Scrape (web content extraction), Clipboard (read/write clipboard), MultiSelect (batch item selection), and MultiEdit (bulk form input).

## Scrape

Extract information from web pages by URL.

**Key parameter:** `url` (required — the page to scrape)

**Returns:** Extracted page content as text.

### When to Use Scrape vs. Snapshot DOM

| Scenario | Use Scrape | Use Snapshot (DOM) |
|----------|-----------|-------------------|
| Page is already open in browser | No | Yes |
| Need data from a URL not yet visited | Yes | No |
| Need to interact with page elements | No | Yes |
| Just need the text content | Yes | Either |
| Need to fill forms or click links | No | Yes |

### Scrape Patterns

**Basic content extraction:**
```
Scrape(url="https://example.com/data-page")
-> process extracted text
```

**Sequential multi-page extraction:**
```
Scrape(url="https://example.com/page/1") -> extract data
Scrape(url="https://example.com/page/2") -> extract data
-> combine results
```

**Best practices:**
- Use for publicly accessible pages
- Combine with `Clipboard(write)` to store extracted data
- For authenticated pages, use browser GUI automation instead (navigate, log in, then use `Snapshot` with `use_dom=True`)

## Clipboard

Read from or write to the Windows system clipboard.

### Read Clipboard
```
Clipboard(action="read")
```
Returns the current clipboard text content.

### Write Clipboard
```
Clipboard(action="write", text="Data to paste")
```
Writes text to the clipboard for pasting.

### Clipboard Workflows

**Extract selected text from any app:**
```
Shortcut("ctrl+a") -> select all
Shortcut("ctrl+c") -> copy to clipboard
Clipboard(read) -> retrieve the text
```

**Paste specific content into any app:**
```
Clipboard(write, text="Prepared content")
Click(target_field_x, target_field_y)
Shortcut("ctrl+v") -> paste
```

**Transfer data between applications:**
```
App(switch to source) -> Wait(0.5)
Shortcut("ctrl+a") -> Shortcut("ctrl+c")
text = Clipboard(read)
-> process or transform the text
Clipboard(write, text=processed_text)
App(switch to destination) -> Wait(0.5)
Click(target_field) -> Shortcut("ctrl+v")
```

**Best practices:**
- Read clipboard immediately after `ctrl+c` — other actions may overwrite it
- Write to clipboard before switching to the target app
- Clipboard only supports text — for images or files, use GUI automation

## MultiSelect

Select multiple items simultaneously (files, folders, checkboxes, list items).

**Key parameter:** `items` (array — list of items to select, by coordinates or identifiers)

### MultiSelect Patterns

**Select multiple files in File Explorer:**
```
Snapshot -> identify file positions
MultiSelect(items=[file1_coords, file2_coords, file3_coords])
```

**Select checkboxes in a form:**
```
Snapshot -> identify checkbox positions
MultiSelect(items=[checkbox1, checkbox2, checkbox3])
```

**Advantages over individual clicks:**
- Faster than sequential `Click` operations
- Atomically selects all items (no risk of losing selection)
- Handles the Ctrl+Click pattern automatically

**Best practices:**
- Always `Snapshot` first to identify item positions
- Verify selection with `Screenshot` after MultiSelect
- For file operations, follow MultiSelect with the desired action (copy, delete, move)

## MultiEdit

Input text into multiple form fields simultaneously.

**Key parameter:** `fields` (array — field identifiers with their values)

### MultiEdit Patterns

**Fill a complete form:**
```
Snapshot -> identify all form fields
MultiEdit(fields=[
  {field: field1_id, value: "First Name"},
  {field: field2_id, value: "Last Name"},
  {field: field3_id, value: "email@example.com"}
])
```

**Advantages over sequential Type:**
- Significantly faster than Click + Type for each field
- Reduces errors from misclicked fields
- Handles focus management automatically

**Best practices:**
- Always `Snapshot` first to identify field IDs
- Verify filled values with `Screenshot` after MultiEdit
- For dropdowns or non-text fields, use `Click` and GUI interaction instead

## Combined Data Workflows

### Web Data to Spreadsheet
```
Scrape(url) -> extract data
-> format as tab-separated values
Clipboard(write, text=formatted_data)
App(switch to Excel) -> Wait(0.5)
Click(target_cell) -> Shortcut("ctrl+v")
```

### Batch File Operations
```
App(launch "File Explorer") -> Wait(2)
Navigate to folder via address bar
Snapshot -> identify files
MultiSelect(items=[target_files])
Shortcut("ctrl+c") -> copy files
Navigate to destination folder
Shortcut("ctrl+v") -> paste files
```

### Form Automation
```
App(switch to browser) -> Wait(0.5)
Snapshot -> identify all form fields
MultiEdit(fields=[all_field_values])
Snapshot -> identify submit button
Click(submit_x, submit_y)
Wait(1) -> Screenshot -> verify submission
```

## Error Handling

- **Scrape returns empty:** The page may require authentication or JavaScript rendering. Use browser GUI + Snapshot(use_dom=True) instead
- **Clipboard read returns old data:** Ensure `ctrl+c` completed before reading. Add `Wait(0.3)` between copy and read
- **MultiSelect misses items:** Re-Snapshot to get updated positions (window may have scrolled or resized)
- **MultiEdit skips fields:** Some fields may be disabled or read-only. Use `Snapshot` to verify field types

## Additional Resources

### Reference Files

For advanced data extraction and manipulation patterns:
- **`references/data-patterns.md`** — Advanced scraping strategies, clipboard automation chains, and bulk operation patterns
