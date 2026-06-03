# Windows Application

WinImagePrep is a native .NET 8 WPF desktop application for preparing Windows 11 installation media with injected drivers. It is the primary and actively maintained tool in the Win11ImagePrep repository.

**Current version:** 4.0.4  
**Download:** [Releases page](https://github.com/andy-kemp/Win11ImagePrep/releases) — `WinImagePrep_full.exe` (72 MB, includes .NET 8 runtime)

## Installation

No installation required. Download `WinImagePrep_full.exe` and run it directly. The executable is self-contained and includes the .NET 8 runtime — it works on any Windows 10/11 x64 machine.

Always run as Administrator — the application requires elevated privileges for DISM operations, disk partitioning, and WIM mounting. It will prompt for elevation automatically if launched without it.

## Interface Overview

The main window is divided into three sections:

**Input selection (top)**

| Field | Purpose |
|-------|---------|
| 1. Select Windows ISO | Path to your Windows 11 ISO file |
| 2. Select Driver MSI | Path to your hardware driver MSI package |
| 3. Select USB Drive | Target USB drive for bootable media creation |

**Action buttons (middle)**

| Button | Action |
|--------|--------|
| Prepare Image with Drivers | Full driver injection workflow |
| Create from Saved Image | Use a previously prepared image |
| Create USB from ISO | Direct USB creation without driver injection |
| Refresh | Re-scan for connected USB drives |
| Repair/Cleanup | Force-unmount stuck WIMs and clean temp files |

**Operation Log (bottom)**  
Real-time timestamped log of all actions performed. Persists for the duration of the session.

## Operation Modes

### Prepare Image with Drivers

The primary workflow. Injects drivers from an MSI package into all components of a Windows 11 ISO and creates a bootable USB.

**What it does:**

1. Mounts the ISO and copies all files to `C:\WinImagePrep\Windows11\`
2. Extracts drivers from the MSI to `C:\WinImagePrep\Drivers\`
3. Validates driver files (checks for `.inf` and `.cat` signature files)
4. Injects drivers into WinPE — `boot.wim` index 1
5. Injects drivers into Windows Setup — `boot.wim` index 2
6. For each selected Windows edition in `install.wim`:
   - Mounts the edition and injects drivers
   - Locates and injects drivers into `winre.wim` (Recovery Environment)
   - Unmounts and commits changes
7. Splits `install.wim` into `.swm` files if it exceeds 4GB (FAT32 file size limit)
8. Formats USB as FAT32 (14GB partition, MBR), copies all files, preserves ISO volume label
9. Optionally saves the prepared image to `C:\WinImagePrep\SavedImages\`

**Expected duration:** 45–90 minutes, depending on the number of editions selected and hardware speed.

!!! warning "Application may appear frozen"
    DISM driver injection takes 10–15 minutes per operation. The UI does not update during these periods — the application is working normally. Do not close it.

### Create from Saved Image

Uses a previously prepared image from `C:\WinImagePrep\SavedImages\` to create a new bootable USB without repeating the full injection process.

1. Select a saved image from the dropdown (image name and ISO label are shown)
2. Select your USB drive and click **Refresh** to detect it
3. Click **Create USB** — the USB is formatted and files copied (~5–10 minutes)

This is significantly faster than the full preparation workflow and is the recommended approach when creating multiple USB drives from the same image.

### Create USB from ISO

Creates a standard bootable USB directly from an unmodified Windows 11 ISO — no driver injection. Equivalent to a basic Rufus operation.

1. Browse to an ISO file
2. Select your USB drive
3. Click **Create USB from ISO**

If `install.wim` exceeds 4GB, it is automatically split into `.swm` files for FAT32 compatibility.

### Repair / Cleanup

Recovers from a failed or interrupted operation.

- Force-dismounts all mounted WIM images (`Dismount-WindowsImage -Discard`)
- Unmounts any mounted ISO images
- Clears the `C:\WinImagePrep\Temp\` and `C:\WinImagePrep\Mount\` directories

Use this if a previous run failed, if DISM reports mount errors, or if the application crashed and left images in a mounted state.

## Advanced Options

### Edition Selection

Click **Select Specific Windows Editions** after selecting an ISO. The application mounts the ISO, reads available editions from `install.wim`, and presents a selection list.

By default all editions are selected and processed. Deselecting editions you don't need (e.g. keeping only Pro and Enterprise) reduces processing time by 10–15 minutes per excluded edition.

Selected editions are remembered for the current session.

### Windows App Removal

Click **Apps** to open the app removal dialog. Select which provisioned Windows apps to remove from the image before deployment:

- Xbox
- OneDrive
- Cortana
- Microsoft Teams
- Other built-in provisioned packages

A live counter shows how many apps are selected. Use **Select All** / **None** to toggle quickly. Removal is performed via DISM (`Remove-AppxProvisionedPackage`) during the image preparation phase.

## Directory Structure

| Path | Contents | Persistent? |
|------|----------|-------------|
| `C:\WinImagePrep\SavedImages\` | Saved prepared images | Yes |
| `C:\WinImagePrep\Logs\` | Application logs | Yes |
| `C:\WinImagePrep\Windows11\` | Extracted ISO contents (working) | No — cleaned after use |
| `C:\WinImagePrep\Drivers\` | Extracted MSI drivers (working) | No — cleaned after use |
| `C:\WinImagePrep\Mount\` | WIM mount points (working) | No — cleaned after use |
| `C:\WinImagePrep\Temp\` | Temporary files | No — cleaned after use |
| `C:\WinImagePrep\Config\` | ISO label and config files | No — cleaned after use |

## Building from Source

If you prefer to build rather than use the pre-built release:

**Prerequisites:** Visual Studio 2022+ or .NET 8 SDK, Windows 10/11

```powershell
git clone https://github.com/andy-kemp/Win11ImagePrep.git
cd Win11ImagePrep/WinImagePrep
dotnet build --configuration Release
```

Output: `WinImagePrep\bin\Release\net8.0-windows\WinImagePrep.exe`

**Self-contained single-file build (no runtime needed on target):**

```powershell
dotnet publish WinImagePrep/WinImagePrep.csproj `
  -c Release -r win-x64 --self-contained true `
  -p:PublishSingleFile=true `
  -p:IncludeNativeLibrariesForSelfExtract=true `
  -p:EnableCompressionInSingleFile=true `
  -p:DebugType=embedded `
  -o "WinImagePrep/bin/Publish"
```

## Operation Close Protection

v4.0.4 added a warning dialog when you attempt to close the application during an active operation. If you confirm, the application cancels the current operation, terminates any running DISM processes, and cleans up before exiting. This prevents leaving WIM images in a mounted state.

## Performance Tips

- **Select only the editions you need** — each edition takes 10–15 minutes to process
- **Use an SSD** for `C:\WinImagePrep\` working directory — DISM is I/O intensive
- **Use USB 3.0+ drives** — significantly faster file copy during USB creation
- **Close other applications** — DISM operations are memory-intensive (8GB+ RAM recommended)
- **Temporarily disable real-time antivirus scanning** — AV scanning slows DISM considerably
