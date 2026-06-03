# Windows Application

WinImagePrep is a native .NET 8 WPF desktop application for preparing Windows 11 installation media with injected drivers. It is the primary and actively maintained tool in the Win11ImagePrep repository.

**Current version:** 4.4.1  
**Download:** [Releases page](https://github.com/andy-kemp/Win11ImagePrep/releases) — `WinImagePrep_full.exe` (72 MB, includes .NET 8 runtime)

## Installation

No installation required. Download `WinImagePrep_full.exe` and run it directly. The executable is self-contained and includes the .NET 8 runtime — it works on any Windows 10/11 x64 machine.

Always run as Administrator — the application requires elevated privileges for DISM operations, disk partitioning, and WIM mounting. It will prompt for elevation automatically if launched without it.

## Interface Overview

The main window has a menu bar and three main sections:

**Menu bar**  
Provides access to **File** (exit), **Tools** (Options and Check for Updates), and **Help** (About dialog with version information).

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
| Load Apps from ISO | Scan the selected ISO to discover all provisioned apps |
| Refresh | Re-scan for connected USB drives |
| Repair/Cleanup | Force-unmount stuck WIMs and clean temp files |

**Operation Log (bottom)**  
Real-time timestamped log of all actions performed. Persists for the duration of the session.

## Operation Modes

### Prepare Image with Drivers

The primary workflow. Injects drivers from an MSI package into all components of a Windows 11 ISO and creates a bootable USB.

**What it does:**

1. Mounts the ISO and copies all files to `C:\Win11ImagePrep\Temp\ExtractedISO\`
2. Extracts drivers from the MSI to `C:\Win11ImagePrep\Temp\Drivers\`
3. Validates driver files (checks for `.inf` and `.cat` signature files)
4. Injects drivers into WinPE — `boot.wim` index 1
5. Injects drivers into Windows Setup — `boot.wim` index 2
6. For each selected Windows edition in `install.wim`:
   - Mounts the edition and injects drivers
   - Locates and injects drivers into `winre.wim` (Recovery Environment)
   - Unmounts and commits changes
7. Splits `install.wim` into `.swm` files if it exceeds 4GB (FAT32 file size limit)
8. Formats USB as FAT32 (14GB partition, MBR), copies all files, preserves ISO volume label
9. Optionally saves the prepared image to `C:\Win11ImagePrep\SavedImages\`

**Expected duration:** 45–90 minutes, depending on the number of editions selected and hardware speed.

!!! warning "Application may appear frozen"
    DISM driver injection takes 10–15 minutes per operation. The UI does not update during these periods — the application is working normally. Do not close it.

### Create from Saved Image

Uses a previously prepared image from `C:\Win11ImagePrep\SavedImages\` to create a new bootable USB without repeating the full injection process.

1. Select a saved image from the dropdown — each entry shows the image folder name and ISO label
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
- Clears the `C:\Win11ImagePrep\Temp\` directory

Use this if a previous run failed, if DISM reports mount errors, or if the application crashed and left images in a mounted state.

## Advanced Options

### Edition Selection

Click **Select Specific Windows Editions** after selecting an ISO. The application mounts the ISO, reads available editions from `install.wim`, and presents a selection list.

By default all editions are selected and processed. Deselecting editions you don't need (e.g. keeping only Pro and Enterprise) reduces processing time by 10–15 minutes per excluded edition.

Selected editions are remembered for the current session.

### Windows App Removal

The app removal system uses a two-stage discovery process to identify the correct packages for your specific ISO:

**Dynamic discovery (recommended):** Click the green **Load Apps from ISO** button to scan `install.wim` inside the selected ISO and discover all provisioned apps actually present in that image — typically ~47 packages. This resolves the exact package name for each app (handling variants like `MSTeams` vs `MicrosoftTeams`), ensuring reliable removal. When you enable the "Remove Windows apps" option, the application automatically offers to run this scan.

**GitHub-hosted app list:** If no scan is run, the app list is loaded at runtime from the [Win11ImagePrep GitHub repository](https://github.com/andy-kemp/Win11ImagePrep), covering 79 packages across x64 and ARM64 architectures. A local cache is used as fallback when offline.

The app selection dialog shows all discovered or listed apps with checkboxes. A live counter shows how many are selected — use **Select All** / **None** to toggle quickly. Removal is performed via DISM (`Remove-AppxProvisionedPackage`) during the image preparation phase. Only packages actually present in the current image are removed (smart removal).

Apps that can be removed include Xbox, OneDrive, Cortana, Microsoft Teams, and many other built-in provisioned packages.

## Directory Structure

The application uses two root locations:

- **Settings root** (`C:\ProgramData\Win11ImagePrep\`) — always fixed; stores `settings.json` and temporary internal files
- **Working root** (`C:\Win11ImagePrep\` by default) — configurable via **Tools > Options**; all ISO processing happens here

| Path | Contents | Persistent? |
|------|----------|-------------|
| `C:\ProgramData\Win11ImagePrep\settings.json` | Application settings (WorkingRoot, log level, temp cleanup preference) | Yes |
| `{WorkingRoot}\SavedImages\` | Named subfolders, one per saved image, each containing `boot\`, `efi\`, `sources\`, and `iso-label.txt` | Yes |
| `{WorkingRoot}\Logs\` | Application logs | Yes |
| `{WorkingRoot}\Temp\ExtractedISO\` | Extracted ISO contents (working) | No — cleaned after use |
| `{WorkingRoot}\Temp\Drivers\` | Extracted MSI drivers (working) | No — cleaned after use |
| `{WorkingRoot}\Temp\Mount\` | WIM mount points (working) | No — cleaned after use |
| `{WorkingRoot}\Temp\` | General temporary files, including ISO extraction for Create USB from ISO | No — cleaned after use |

## Auto-Update

From **Tools > Check for Updates**, the application checks the Win11ImagePrep GitHub repository for a newer version. If one is available, it is downloaded and installed automatically. The updater runs elevated, creates a backup of the current executable, and rolls back automatically on failure.

Version tracking is handled via `version.json` in the application directory.

## Settings

Open **Tools > Options** to configure the application.

| Setting | Default | Description |
|---------|---------|-------------|
| Working Root | `C:\Win11ImagePrep` | Root folder for all ISO processing, driver extraction, and temporary files. Must be on a local fixed drive with 25 GB+ free space. Cannot be a network path, removable drive, or OneDrive folder. |
| Delete temp files on exit | Enabled | Removes `Temp\` contents when the application closes |
| Log level | Information | Controls verbosity of the operation log (Minimal / Information / Verbose) |

Click **Reset to Defaults** to restore all settings to their original values. Changing the Working Root takes effect on the next application start.

Settings are stored at `C:\ProgramData\Win11ImagePrep\settings.json` separately from the working folder, so they persist even if you relocate or reset the working root.

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

A warning dialog is shown when you attempt to close the application during an active operation. If you confirm, the application cancels the current operation, terminates any running DISM processes, and cleans up before exiting. This prevents leaving WIM images in a mounted state.

## Performance

### Typical Timing

| Operation | Typical Duration | Notes |
|-----------|-----------------|-------|
| ISO Validation | 30–60 seconds | |
| ISO Extraction | 2–5 minutes | ~5 GB of files |
| MSI Extraction | 1–2 minutes | Varies by driver package size |
| WinPE Injection | 5–10 minutes | First DISM operation |
| WinSetup Injection | 5–10 minutes | Second DISM operation |
| Edition Injection | 10–15 min each | Multiplied by number of editions selected (typically 5–10 editions) |
| WinRE Injection | 3–5 min each | Per edition |
| WIM Splitting | 3–5 minutes | Only if `install.wim` exceeds 4 GB |
| USB Creation | 5–10 minutes | ~6 GB file copy |
| **Total (full workflow)** | **45–90 minutes** | All editions selected |

### Performance Tips

- **Select only the editions you need** — each edition takes 10–15 minutes to process
- **Use an SSD** for the working root (`C:\Win11ImagePrep\` by default) — DISM is I/O intensive
- **Use USB 3.0+ drives** — significantly faster file copy during USB creation
- **Close other applications** — DISM operations are memory-intensive (8GB+ RAM recommended)
- **Temporarily disable real-time antivirus scanning** — AV scanning slows DISM considerably
