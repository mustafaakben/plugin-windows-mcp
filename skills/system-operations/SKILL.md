---
name: Windows System Operations
description: This skill should be used when the user asks to "run a PowerShell command", "execute a shell command", "list processes", "kill a process", "modify the registry", "send a notification", "check system info", "manage Windows services", "automate system tasks", or needs to perform system-level operations on Windows using Shell, Process, Registry, or Notification tools.
version: 1.0.0
---

# Windows System Operations

This skill covers four system-level tools: Shell (PowerShell execution), Process (process management), Registry (Windows Registry operations), and Notification (toast notifications).

## Shell

Execute PowerShell commands directly on the system.

**Key parameter:** `command` (string — the PowerShell command to run)

**Returns:** stdout, stderr, and exit code.

### When to Use Shell vs. GUI Tools

**Prefer Shell when:**
- The task can be accomplished via command line
- Reliability matters more than visual feedback
- Performing file operations, system queries, or batch tasks
- Running scripts or installers

**Use GUI tools when:**
- The task requires visual interaction (e.g., drawing, drag-and-drop)
- No CLI equivalent exists
- The user specifically needs to see the visual result

### Common Shell Patterns

**File operations:**
```powershell
Get-ChildItem -Path "C:\Users" -Recurse -Filter "*.txt"
Copy-Item "source.txt" "destination.txt"
New-Item -ItemType Directory -Path "C:\NewFolder"
Remove-Item "file.txt" -Force
```

**System information:**
```powershell
Get-ComputerInfo | Select-Object WindowsProductName, OsVersion
Get-WmiObject -Class Win32_Processor | Select-Object Name, NumberOfCores
Get-CimInstance -ClassName Win32_LogicalDisk | Select-Object DeviceID, FreeSpace, Size
[System.Environment]::OSVersion
```

**Network:**
```powershell
Test-NetConnection -ComputerName "google.com" -Port 443
Get-NetIPAddress | Where-Object AddressFamily -eq "IPv4"
Invoke-WebRequest -Uri "https://example.com" -OutFile "page.html"
```

**Software management:**
```powershell
Get-Package | Where-Object Name -like "*Python*"
winget list
winget install "App.Name"
```

**Services:**
```powershell
Get-Service | Where-Object Status -eq "Running"
Restart-Service -Name "ServiceName"
Get-Service -Name "wuauserv" | Select-Object Status, StartType
```

### Shell Best Practices

1. **Avoid interactive commands** — Do not run commands requiring user input (e.g., Read-Host, confirmations). Use `-Force` or `-Confirm:$false` flags
2. **Handle errors** — Check exit codes and stderr output
3. **Use full paths** — Avoid relying on PATH for critical scripts
4. **Escape special characters** — PowerShell uses backtick for escaping
5. **Limit output** — Use `Select-Object` and `Where-Object` to filter large result sets

## Process

List running processes or terminate them by name or PID.

### List Processes
```
Process(action="list")
```

Returns all running processes with their PID, name, CPU, and memory usage.

**Use cases:**
- Check if an application is running before launching it
- Find PIDs for processes to terminate
- Monitor resource consumption

### Kill Processes
```
Process(action="kill", name="notepad")
Process(action="kill", pid=1234)
```

**Best practices:**
- Prefer `name` for well-known single-instance apps
- Use `pid` when multiple instances exist and only one should be terminated
- Verify with `Process(list)` after killing to confirm termination
- Be cautious with system processes — killing critical processes can cause instability

**Common termination targets:**
- Hung applications that are not responding
- Background processes consuming excessive resources
- Applications that need to be restarted

## Registry

Read, write, delete, or list Windows Registry keys and values.

### Read a Value
```
Registry(action="read", path="HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer", name="ShellState")
```

### Write a Value
```
Registry(action="write", path="HKCU\\Software\\MyApp", name="Setting", value="enabled", type="REG_SZ")
```

### List Keys
```
Registry(action="list", path="HKCU\\Software\\Microsoft")
```

### Delete a Value
```
Registry(action="delete", path="HKCU\\Software\\MyApp", name="OldSetting")
```

### Registry Safety Rules

1. **Always read before writing** — Verify the current value before overwriting
2. **Prefer HKCU over HKLM** — Current-user changes are safer than machine-wide changes
3. **Never delete keys blindly** — List contents first to understand what will be removed
4. **Back up before modifying** — Export the key with `reg export` via Shell before changes
5. **Use correct types** — REG_SZ (string), REG_DWORD (32-bit integer), REG_QWORD (64-bit integer), REG_EXPAND_SZ (expandable string), REG_MULTI_SZ (multi-string)

### Common Registry Paths

| Path | Purpose |
|------|---------|
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` | User startup programs |
| `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` | System startup programs |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer` | Explorer settings |
| `HKCU\Control Panel\Desktop` | Desktop settings (wallpaper, etc.) |
| `HKLM\SYSTEM\CurrentControlSet\Services` | Windows services |

## Notification

Send Windows toast notifications to alert the user.

**Parameters:** `title` (required), `message` (required)

```
Notification(title="Task Complete", message="The file export has finished successfully.")
```

**When to use:**
- Long-running automation tasks have completed
- An error or warning needs user attention
- Status updates during background operations
- Confirmation that a requested action was performed

**Best practices:**
- Keep titles short (under 50 characters)
- Make messages actionable and informative
- Do not spam — reserve notifications for meaningful events

## Combined System Workflows

### Check and Restart a Service
```
Shell("Get-Service -Name 'wuauserv'") -> check status
Shell("Restart-Service -Name 'wuauserv' -Force") -> restart
Shell("Get-Service -Name 'wuauserv'") -> verify running
Notification(title="Service Restarted", message="Windows Update service has been restarted.")
```

### Kill Hung App and Relaunch
```
Process(list) -> find the hung process
Process(kill, name="myapp")
Wait(1)
App(launch "MyApp") -> Wait(2)
Screenshot -> verify app is running
```

### System Health Check
```
Shell("Get-CimInstance Win32_Processor | Select-Object LoadPercentage")
Shell("Get-CimInstance Win32_LogicalDisk | Select-Object DeviceID, @{N='FreeGB';E={[math]::Round($_.FreeSpace/1GB,2)}}")
Shell("Get-Process | Sort-Object WorkingSet64 -Descending | Select-Object -First 5 Name, @{N='MemMB';E={[math]::Round($_.WorkingSet64/1MB,1)}}")
```

## Additional Resources

### Reference Files

For advanced PowerShell patterns and system administration techniques:
- **`references/powershell-patterns.md`** — Common PowerShell one-liners, service management, and system administration patterns
