# Data Operations Patterns

Advanced scraping strategies, clipboard automation chains, and bulk operation patterns for Windows MCP.

---

## Web Scraping Strategies

### Basic Page Scrape
```
Scrape(url="https://example.com/data")
-> parse extracted content
-> use data for next steps
```

### Multi-Page Scrape
```
For each page URL:
  Scrape(url=page_url)
  -> extract relevant data
  -> accumulate results
-> process combined results
```

### Scrape vs. Browser DOM Extraction

**Use Scrape when:**
- The page is publicly accessible (no login required)
- Static content that does not require JavaScript rendering
- Only the text content is needed
- The URL is known in advance

**Use Snapshot(use_dom=True) when:**
- The page requires authentication (already logged in via browser)
- Content is dynamically loaded via JavaScript
- Need to interact with the page after reading content
- The page is already open in the browser

### Handling Authenticated Pages
```
# Cannot use Scrape for authenticated pages
# Instead, use browser automation:

App(launch, name="Chrome") -> Wait(2)
# Navigate to login page
Snapshot -> find URL bar -> Click -> Type(url)
Shortcut("enter") -> Wait(2)

# Log in
Snapshot -> find login fields
Type(username) -> Click(password_field) -> Type(password)
Click(login_button) -> Wait(2)

# Now extract content
Snapshot(use_dom=True) -> read DOM content
```

---

## Clipboard Automation Chains

### Text Processing Pipeline
```
# Extract from app
App(switch to source) -> Wait(0.5)
Shortcut("ctrl+a") -> Shortcut("ctrl+c")
raw_text = Clipboard(read)

# Process the text (e.g., transformation handled by Claude)
processed_text = transform(raw_text)

# Deliver to destination
Clipboard(write, text=processed_text)
App(switch to destination) -> Wait(0.5)
Click(target_field) -> Shortcut("ctrl+v")
```

### Multi-Source Data Collection
```
# Source 1
App(switch to App1) -> select content -> Shortcut("ctrl+c")
data1 = Clipboard(read)

# Source 2
App(switch to App2) -> select content -> Shortcut("ctrl+c")
data2 = Clipboard(read)

# Combine and deliver
combined = format(data1, data2)
Clipboard(write, text=combined)
App(switch to target) -> Click(field) -> Shortcut("ctrl+v")
```

### Clipboard Safety
- **Read immediately after copy** — Other actions may overwrite the clipboard
- **Write before switching** — Write to clipboard while still in the current context
- **Text only** — Clipboard tool handles text. For images/files, use GUI automation
- **Large text** — No practical limit, but very large clipboard content may slow paste operations

---

## MultiSelect Patterns

### File Selection in Explorer
```
# Open File Explorer and navigate
App(launch, name="File Explorer") -> Wait(2)
Click(address_bar) -> Type(text="C:\\Documents", clear=true) -> Shortcut("enter")
Wait(1)

# Identify and select files
Snapshot -> identify target file positions
MultiSelect(items=[file1_coords, file2_coords, file3_coords])

# Perform action on selected files
Shortcut("ctrl+c")  # Copy
Shortcut("delete")   # Delete
Click(right-click) -> choose action from context menu
```

### Checkbox Batch Selection
```
Snapshot -> identify all checkboxes
# Filter for desired checkboxes based on labels
MultiSelect(items=[checkbox1, checkbox2, checkbox5])
Screenshot -> verify selections
```

### List Item Selection
```
Snapshot -> identify list items
MultiSelect(items=[item1, item3, item7])
# Now operate on selected items
```

### Select All Then Deselect Some
```
Shortcut("ctrl+a")  # Select all items
# Deselect specific items by Ctrl+clicking
Click(item_to_deselect_x, item_to_deselect_y)  # with Ctrl held via Shortcut
```

---

## MultiEdit Patterns

### Complete Form Fill
```
Snapshot -> identify all form fields by label and position

MultiEdit(fields=[
  {field: name_field, value: "John Doe"},
  {field: email_field, value: "john@example.com"},
  {field: phone_field, value: "555-0123"},
  {field: address_field, value: "123 Main St"}
])

Screenshot -> verify all fields filled correctly
Click(submit_button) -> Wait(1) -> Screenshot -> verify submission
```

### Repeated Form Entry
```
For each data record:
  # Navigate to empty form
  Click(new_entry_button) -> Wait(0.5)
  Snapshot -> identify fields

  MultiEdit(fields=[record_field_values])

  Click(save_button) -> Wait(0.5)
  Screenshot -> verify saved
```

### Partial Form Update
```
Snapshot -> identify fields that need updating

MultiEdit(fields=[
  {field: changed_field_1, value: "new_value_1"},
  {field: changed_field_2, value: "new_value_2"}
])

Click(update_button)
```

---

## Combined Data Workflows

### Web-to-Spreadsheet Pipeline
```
# Extract from web
Scrape(url="https://example.com/data-table")
-> parse into rows and columns
-> format as tab-separated values

# Paste into spreadsheet
Clipboard(write, text=tab_separated_data)
App(switch to Excel) -> Wait(0.5)
Click(target_cell_A1)
Shortcut("ctrl+v")
Wait(0.5) -> Screenshot -> verify data pasted
```

### Cross-Application Report
```
# Collect from multiple sources
App(switch to App1) -> extract data via GUI or Shell
App(switch to App2) -> extract data via GUI or Shell
Scrape(url) -> extract web data

# Compile into document
App(switch to Word) -> Wait(0.5)
Type(text=compiled_report)
Shortcut("ctrl+s") -> save
```

### Data Validation Flow
```
# Read existing data
Snapshot -> identify data cells/fields
Shortcut("ctrl+a") -> Shortcut("ctrl+c")
current_data = Clipboard(read)

# Validate (processed by Claude)
validated_data = validate(current_data)

# Update invalid entries
For each invalid entry:
  Click(cell/field) -> Type(text=corrected_value, clear=true)

Screenshot -> verify corrections
```

---

## Performance Tips

### Batch Over Sequential
```
# Slow: individual operations
Click(field1) -> Type("value1")
Click(field2) -> Type("value2")
Click(field3) -> Type("value3")
# 6 tool calls

# Fast: batch operation
MultiEdit(fields=[{f1: "value1"}, {f2: "value2"}, {f3: "value3"}])
# 1 tool call
```

### Minimize Captures
```
# Take ONE Snapshot, extract ALL needed info
Snapshot -> identify:
  - All form fields
  - Submit button
  - Any dropdowns or checkboxes
# Then act on all identified elements without re-capturing
```

### Use Shell for Data Processing
```
# Shell is faster than GUI for data transformation
Shell("Get-Content data.csv | ConvertFrom-Csv | Where-Object Status -eq 'Active' | ConvertTo-Csv -NoTypeInformation | Set-Content filtered.csv")
```
