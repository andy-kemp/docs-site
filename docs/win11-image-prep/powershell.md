# PowerShell Script

`WinImagePrep_V3.ps1` is a self-contained PowerShell WPF application that provides the same driver injection and USB creation workflow as the Windows Application, using PowerShell and .NET WPF controls. It is the predecessor to the native WPF app and is suited to environments where running a known script is preferred over a compiled executable.

!!! note "Recommendation"
    For most deployments the [Windows Application](windows-app.md) (`WinImagePrep_full.exe`) is recommended. It is actively developed, has more features (v4.0.4 vs v3), and requires no execution policy changes.

## When to Use the Script

- You want to review exactly what runs before executing it
- Your environment restricts unsigned executables but permits scripts
- You are already in a PowerShell workflow and want to launch directly

## Prerequisites

- Windows 10 or Windows 11
- PowerShell 5.1 or later (built into Windows)
- Administrator privileges
- 25GB+ free space on `C:`

No additional modules or runtime installations are required — the script uses only built-in Windows APIs.

## Running the Script

### Option 1 — Right-click and Run as Administrator

1. Right-click `WinImagePrep_V3.ps1`
2. Select **Run with PowerShell**
3. If prompted for execution policy, see below

### Option 2 — From a PowerShell prompt

```powershell
# Allow script execution for this session only
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass

# Run as Administrator
.\WinImagePrep_V3.ps1
```

!!! warning "Execution Policy"
    Windows blocks PowerShell scripts by default. The `-Scope Process` flag bypasses this for the current session only and does not permanently change your system policy.

### Option 3 — Using the launch helper

The repo includes `Run-WinImagePrep.ps1`, which launches the compiled .NET application rather than the script. To launch the script specifically, use the direct methods above.

## Interface

The script presents an identical graphical interface to the Windows Application (v3 UI):

- ISO file selection with Verify button
- Driver MSI selection
- Edition selection dialog
- Action buttons: Prepare Image, Create from Saved Image, Create USB from ISO
- USB drive selection with Refresh and real-time drive info panel
- Real-time Operation Log (black console-style pane, green text)
- About dialog (Help > ?)
- Repair/Cleanup button

## Operation Modes

The script supports the same core operations as the Windows Application:

| Mode | Button |
|------|--------|
| Full driver injection + USB creation | **Prepare Image with Drivers** |
| USB from a previously saved image | **Create from Saved Image** |
| Direct USB from ISO without injection | **Create USB from ISO** |
| Force-unmount and cleanup | **Repair/Cleanup** |

See [Windows Application](windows-app.md) for a detailed walkthrough of each mode — the workflow is identical.

## What the Script Does

The script is fully self-contained. It does not call external helper files. Key capabilities:

- **Silent command execution** — DISM and msiexec run with no console windows or popups
- **Progress dialogs** — WPF-based progress windows with a cancel button for each long operation
- **Edition selection dialog** — mounts ISO, reads editions from `install.wim`, presents a multi-select list
- **Saved image management** — browse and reuse previously prepared images from `C:\WinImagePrep\SavedImages\`
- **USB from ISO dialog** — standalone dialog for direct USB creation without driver injection
- **Driver validation** — checks MSI contents for `.inf` and `.cat` files before proceeding
- **Disk space check** — validates 25GB free on `C:` before starting
- **ISO integrity check** — verifies `boot.wim` and `install.wim` are present in the ISO
- **WIM splitting** — automatically splits `install.wim` >4GB into `.swm` files for FAT32
- **Cleanup on error** — a global error trap calls `Invoke-Cleanup` to dismount all images if anything fails

## Directory Structure

Same as the Windows Application:

| Path | Contents |
|------|----------|
| `C:\WinImagePrep\SavedImages\` | Persistent saved prepared images |
| `C:\WinImagePrep\Logs\` | Logs |
| `C:\WinImagePrep\Windows11\` | Temporary extracted ISO (cleaned after use) |
| `C:\WinImagePrep\Drivers\` | Temporary extracted drivers (cleaned after use) |
| `C:\WinImagePrep\Mount\` | Temporary WIM mount points (cleaned after use) |

## Script Security

The script does not connect to the internet, send any data externally, or install anything permanently. All operations are local file and disk operations performed under the user's administrator session.

To review the full source before running:

```powershell
# Read script without executing
Get-Content .\WinImagePrep_V3.ps1 | more
```

Some antivirus products flag PowerShell scripts that use WMI, DISM, disk partitioning, or WPF GUI components as suspicious. This is a false positive. If blocked, add an exclusion for the script file or use the compiled Windows Application instead.

## Limitations vs Windows Application

| Feature | PowerShell Script (V3) | Windows Application (V4) |
|---------|----------------------|-------------------------|
| App removal dialog | No | Yes |
| Operation close protection | No | Yes |
| Menu bar | No | Yes |
| Active development | No | Yes |
| Requires execution policy change | Sometimes | No |
| Source code reviewable | Yes | Yes (open source) |
