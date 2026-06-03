# Win11 Image Prep

Create customised Windows 11 installation media with injected drivers and pre-removed apps — ready to deploy, no post-install cleanup required.

## What It Does

Win11 Image Prep solves the problem of deploying Windows 11 to hardware (particularly Surface devices) that requires drivers to be present during setup — before Windows is even installed. Rather than installing Windows and then layering drivers on top, this tool injects drivers directly into the ISO image so they are available from the first boot.

The result is a bootable USB drive with:

- Drivers pre-injected into WinPE, Windows Setup, all Windows editions, and the Recovery Environment
- Unwanted built-in apps (Xbox, Cortana, OneDrive, Teams, etc.) removed before deployment
- Only the Windows editions you want included in the installer
- A UEFI-compatible FAT32 bootable USB ready to use

## Two Tools, One Repo

The [Win11ImagePrep repository](https://github.com/andy-kemp/Win11ImagePrep) contains two tools that do the same job — choose the one that fits your environment:

| Tool | Description | Best For |
|------|-------------|----------|
| **Windows Application** | Native .NET 8 WPF desktop app (`WinImagePrep_full.exe`) | Most users — no dependencies, just download and run |
| **PowerShell Script** | Self-contained PowerShell WPF script (`WinImagePrep_V3.ps1`) | Environments where running a script is preferred over an unsigned exe |

Both tools provide a graphical interface, real-time operation logs, and the same core driver injection workflow. The Windows Application (v4) is the actively developed version and is recommended for all new use.

## How It Works

```
Select Windows 11 ISO + Driver MSI
        ↓
Mount ISO → Extract files
        ↓
Extract drivers from MSI
        ↓
Inject drivers into WinPE (boot.wim index 1)
        ↓
Inject drivers into Windows Setup (boot.wim index 2)
        ↓
Inject drivers into each Windows edition (install.wim)
        ↓
Inject drivers into Recovery Environment (winre.wim)
        ↓
Split install.wim if >4GB (FAT32 compatibility)
        ↓
Format USB as FAT32 → Copy all files → Bootable USB ready
```

## Four Operation Modes

| Mode | What It Does |
|------|-------------|
| **Prepare Image with Drivers** | Full workflow — inject drivers, remove apps, then create USB |
| **Create USB from ISO** | Create a standard bootable USB from an unmodified ISO (no driver injection) |
| **Create USB from Saved Image** | Use a previously prepared image to create a new USB quickly |
| **Repair / Cleanup** | Force-unmount stuck WIM images and clean temporary files |

## Key Features

- :material-usb: **Bootable USB creation** — UEFI-compatible FAT32, MBR partitioned, 14GB+ partition
- :material-screwdriver: **Driver injection** — WinPE, Setup, all editions, and WinRE in one pass
- :material-package-variant-remove: **App removal** — remove provisioned Windows apps before deployment; dynamic discovery scans the ISO to find all ~47 packages present in your specific image
- :material-format-list-checks: **Edition control** — select which Windows editions to include, suppress others
- :material-content-save: **Saved images** — save prepared images to `C:\Win11ImagePrep\SavedImages\` for fast reuse
- :material-shield-check: **Driver validation** — checks for `.inf` and `.cat` signature files before injection
- :material-progress-clock: **Real-time logging** — live operation log throughout the entire process
- :material-broom: **Auto-cleanup** — temporary files cleaned automatically; saved images preserved
- :material-update: **Auto-update** — check for and install updates directly from the Tools menu

## Quick Links

- [Quick Start](quick-start.md) — Get up and running with the Windows Application
- [Windows Application](windows-app.md) — Detailed guide for WinImagePrep v4
- [PowerShell Script](powershell.md) — Using the PowerShell-based tool
- [Requirements](requirements.md) — System and hardware requirements
- [Troubleshooting](troubleshooting.md) — Common issues and fixes
