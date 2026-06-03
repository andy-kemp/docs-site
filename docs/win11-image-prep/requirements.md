# Requirements

## Operating System

- Windows 10 (20H2 or later)
- Windows 11 (any version)
- x64 architecture only

Both tools require an existing Windows machine to run — they cannot run inside WSL or a Windows container.

## Privileges

Administrator privileges are required. The application and script will prompt for elevation if not already running as admin. All DISM operations, disk partitioning, ISO mounting, and WIM file access require full administrator rights.

## Disk Space

A minimum of **25GB free space on the C: drive** is required for working files during image preparation.

| Working directory | Approximate size |
|------------------|-----------------|
| `C:\WinImagePrep\Windows11\` — extracted ISO | ~5 GB |
| `C:\WinImagePrep\Drivers\` — extracted MSI drivers | ~2 GB |
| `C:\WinImagePrep\Mount\` — mounted WIM images | ~15 GB |
| Buffer for temporary operations | ~3 GB |

Working directories are cleaned automatically after a successful run or on the next startup. Saved images in `C:\WinImagePrep\SavedImages\` persist and are not cleaned automatically — manage these manually as needed.

## RAM

- **Minimum:** 4 GB
- **Recommended:** 8 GB or more

DISM driver injection is memory-intensive. Insufficient RAM will cause operations to be slower or may trigger paging to disk, increasing total processing time.

## USB Drive

- **Minimum capacity:** 14 GB
- **Recommended capacity:** 32 GB or larger
- FAT32 formatted by the tool (any existing data will be erased)
- USB 3.0 or faster strongly recommended for reasonable file copy times
- UEFI-compatible — the created USB is not designed for legacy BIOS boot

## Processor

A modern multi-core CPU is recommended. DISM driver injection operations are CPU-intensive and are performed sequentially. More cores improve responsiveness but DISM does not parallelise individual injection operations.

## Software Dependencies

### Windows Application (`WinImagePrep_full.exe`)

The pre-built `WinImagePrep_full.exe` is self-contained and includes the .NET 8 runtime. No additional software installation is required.

If building from source:

- .NET 8.0 SDK or Visual Studio 2022+
- Windows 10/11 SDK

### PowerShell Script (`WinImagePrep_V3.ps1`)

- PowerShell 5.1 or later (built into Windows — no installation required)
- No external modules required

### Built into Windows (both tools rely on these)

| Component | Purpose |
|-----------|---------|
| **DISM** (`dism.exe`) | Driver injection, WIM mounting, app removal |
| **PowerShell** | ISO mounting via `Mount-DiskImage` |
| **robocopy** | Fast file copying of ISO and USB contents |
| **msiexec** | MSI driver package extraction |

All of the above are present in any standard Windows 10/11 installation.

## Input Files

### Windows 11 ISO

- Official ISO from [Microsoft](https://www.microsoft.com/software-download/windows11)
- Business or Consumer editions both supported
- Must contain `Sources\boot.wim` and `Sources\install.wim`
- ISOs using `install.esd` require manual conversion first — see [Troubleshooting](troubleshooting.md)

### Driver MSI Package

- MSI-packaged hardware drivers (e.g. [Surface drivers from Microsoft](https://support.microsoft.com/en-us/surface/download-drivers-and-firmware-for-surface))
- Must contain `.inf` driver files when extracted
- Signed drivers (`.cat` files present) are preferred; unsigned drivers are flagged but still injected
- Only local file paths are supported — UNC network paths are not currently supported
