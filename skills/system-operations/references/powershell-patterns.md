# PowerShell Patterns for Windows MCP

Common PowerShell one-liners, system administration commands, and automation patterns for use with the Shell tool.

---

## File System Operations

### List Files
```powershell
# List all files in a directory
Get-ChildItem -Path "C:\Users" -Recurse -File

# List by extension
Get-ChildItem -Path "C:\Projects" -Filter "*.py" -Recurse

# List files modified in the last 24 hours
Get-ChildItem -Path "C:\" -Recurse -File | Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-1) }

# List large files (>100MB)
Get-ChildItem -Path "C:\" -Recurse -File | Where-Object { $_.Length -gt 100MB } | Sort-Object Length -Descending | Select-Object FullName, @{N='SizeMB';E={[math]::Round($_.Length/1MB,2)}}
```

### Copy, Move, Delete
```powershell
# Copy with overwrite
Copy-Item "source.txt" "dest.txt" -Force

# Copy directory recursively
Copy-Item "C:\Source" "C:\Dest" -Recurse -Force

# Move/rename
Move-Item "old-name.txt" "new-name.txt"

# Delete file
Remove-Item "file.txt" -Force

# Delete directory recursively
Remove-Item "C:\TempFolder" -Recurse -Force

# Create directory
New-Item -ItemType Directory -Path "C:\NewFolder" -Force
```

### File Content
```powershell
# Read file content
Get-Content "file.txt"

# Read first/last N lines
Get-Content "file.txt" -Head 10
Get-Content "file.txt" -Tail 20

# Search file content
Select-String -Path "*.log" -Pattern "ERROR"

# Replace text in file
(Get-Content "file.txt") -replace "old-text", "new-text" | Set-Content "file.txt"

# Write to file
"content" | Set-Content "file.txt"
"append this" | Add-Content "file.txt"
```

---

## System Information

### Computer Info
```powershell
# OS version
[System.Environment]::OSVersion

# Full computer info
Get-ComputerInfo | Select-Object WindowsProductName, OsVersion, CsName, CsProcessors, CsTotalPhysicalMemory

# Hostname
$env:COMPUTERNAME

# Current user
$env:USERNAME

# Uptime
(Get-Date) - (Get-CimInstance Win32_OperatingSystem).LastBootUpTime
```

### Hardware
```powershell
# CPU info
Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores, NumberOfLogicalProcessors, MaxClockSpeed

# Memory
Get-CimInstance Win32_PhysicalMemory | Measure-Object Capacity -Sum | Select-Object @{N='TotalGB';E={[math]::Round($_.Sum/1GB,2)}}

# Disk space
Get-CimInstance Win32_LogicalDisk | Select-Object DeviceID, @{N='FreeGB';E={[math]::Round($_.FreeSpace/1GB,2)}}, @{N='TotalGB';E={[math]::Round($_.Size/1GB,2)}}, @{N='UsedPercent';E={[math]::Round(($_.Size-$_.FreeSpace)/$_.Size*100,1)}}

# GPU
Get-CimInstance Win32_VideoController | Select-Object Name, DriverVersion, AdapterRAM
```

### Network
```powershell
# IP addresses
Get-NetIPAddress | Where-Object AddressFamily -eq "IPv4" | Select-Object InterfaceAlias, IPAddress

# Test connectivity
Test-NetConnection -ComputerName "google.com" -Port 443

# DNS lookup
Resolve-DnsName "example.com"

# Active connections
Get-NetTCPConnection | Where-Object State -eq "Established" | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort

# WiFi info
netsh wlan show interfaces
```

---

## Process Management

### List and Filter
```powershell
# All processes sorted by memory
Get-Process | Sort-Object WorkingSet64 -Descending | Select-Object -First 10 Name, @{N='MemMB';E={[math]::Round($_.WorkingSet64/1MB,1)}}, CPU

# Find specific process
Get-Process -Name "chrome" -ErrorAction SilentlyContinue

# Processes with windows
Get-Process | Where-Object { $_.MainWindowTitle -ne '' } | Select-Object ProcessName, MainWindowTitle

# CPU-intensive processes
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 Name, CPU, @{N='MemMB';E={[math]::Round($_.WorkingSet64/1MB,1)}}
```

### Start and Stop
```powershell
# Start a process
Start-Process "notepad.exe"
Start-Process "C:\Program Files\App\app.exe" -ArgumentList "--flag"

# Open a file with default app
Start-Process "document.pdf"

# Open URL in default browser
Start-Process "https://example.com"

# Stop process
Stop-Process -Name "notepad" -Force
Stop-Process -Id 1234 -Force
```

---

## Service Management

```powershell
# List all services
Get-Service | Sort-Object Status | Select-Object Name, DisplayName, Status, StartType

# Running services
Get-Service | Where-Object Status -eq "Running"

# Check specific service
Get-Service -Name "wuauserv" | Select-Object Name, DisplayName, Status, StartType

# Start/Stop/Restart (may require elevation)
Start-Service -Name "ServiceName"
Stop-Service -Name "ServiceName" -Force
Restart-Service -Name "ServiceName" -Force
```

---

## Software Management

```powershell
# List installed software
Get-Package | Select-Object Name, Version, ProviderName | Sort-Object Name

# Search for software
Get-Package | Where-Object Name -like "*Python*"

# Winget operations
winget list
winget search "app name"
winget install "App.Publisher.Name"
winget upgrade --all

# Check if command exists
Get-Command "python" -ErrorAction SilentlyContinue
```

---

## Environment Variables

```powershell
# List all
Get-ChildItem Env:

# Get specific
$env:PATH
$env:USERPROFILE
$env:APPDATA

# Set for current session
$env:MY_VAR = "value"

# Set persistently for user
[System.Environment]::SetEnvironmentVariable("MY_VAR", "value", "User")

# Add to PATH (user level)
$path = [System.Environment]::GetEnvironmentVariable("PATH", "User")
[System.Environment]::SetEnvironmentVariable("PATH", "$path;C:\NewPath", "User")
```

---

## Scheduled Tasks

```powershell
# List scheduled tasks
Get-ScheduledTask | Select-Object TaskName, State, TaskPath

# Get task details
Get-ScheduledTask -TaskName "TaskName" | Get-ScheduledTaskInfo

# Run task immediately
Start-ScheduledTask -TaskName "TaskName"

# Disable/Enable task
Disable-ScheduledTask -TaskName "TaskName"
Enable-ScheduledTask -TaskName "TaskName"
```

---

## Date and Time

```powershell
# Current date/time
Get-Date
Get-Date -Format "yyyy-MM-dd HH:mm:ss"

# Timezone
Get-TimeZone

# Calculate time differences
(Get-Date) - (Get-Date "2024-01-01")

# File age
(Get-Date) - (Get-Item "file.txt").LastWriteTime
```

---

## Output Formatting

```powershell
# Export to CSV
Get-Process | Export-Csv "processes.csv" -NoTypeInformation

# Export to JSON
Get-Process | ConvertTo-Json | Set-Content "processes.json"

# Format as table
Get-Service | Format-Table Name, Status, StartType -AutoSize

# Format as list (detailed)
Get-Service -Name "wuauserv" | Format-List *
```

---

## Error Handling

```powershell
# Try/Catch
try {
    Get-Item "nonexistent.txt" -ErrorAction Stop
} catch {
    Write-Output "Error: $_"
}

# Suppress errors
Get-Process -Name "nonexistent" -ErrorAction SilentlyContinue

# Check last command result
$LASTEXITCODE  # for native commands
$?             # for PowerShell cmdlets
```
