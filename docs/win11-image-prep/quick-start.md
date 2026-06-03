# Quick Start

Get from download to bootable USB in the shortest path possible.

## What You Need

Before starting, have these ready:

- A **Windows 11 ISO** — download from [Microsoft](https://www.microsoft.com/software-download/windows11) (Business or Consumer editions both work)
- A **Driver MSI package** — e.g. Surface drivers from [Microsoft Surface Support](https://support.microsoft.com/en-us/surface/download-drivers-and-firmware-for-surface)
- A **USB drive** — 14GB minimum, 32GB+ recommended, all data will be erased
- **25GB+ free space** on `C:` drive for working files
- A Windows 10 or 11 machine with administrator access

## Step 1 — Download the Application

Download `WinImagePrep_full.exe` from the [Releases page](https://github.com/andy-kemp/Win11ImagePrep/releases).

This is a single self-contained executable — no installation, no .NET runtime required. Just download and run.

## Step 2 — Run as Administrator

Right-click `WinImagePrep_full.exe` and select **Run as administrator**.

!!! note
    The application will detect if it is not running with elevated privileges and prompt you to restart with elevation automatically.

## Step 3 — Select Your ISO

Click **Browse...** next to **1. Select Windows ISO** and choose your Windows 11 ISO file.

Optionally, click **Verify** to confirm the ISO contains valid `boot.wim` and `install.wim` files and that you have enough disk space.

## Step 4 — Select Your Driver MSI

Click **Browse...** next to **2. Select Driver MSI** and select your hardware driver MSI file (e.g. a Surface driver package).

## Step 5 — Configure Options (Optional)

Click **Select Specific Windows Editions** to choose which editions to include in the prepared image. By default all editions are processed — selecting only the editions you need (e.g. Pro only) significantly reduces processing time.

To remove built-in Windows apps from the image, click the **Apps** button. When you enable app removal, the application will offer to scan the ISO automatically and load the full list of provisioned apps present in that image. You can also click the green **Load Apps from ISO** button at any time to trigger the scan manually.

## Step 6 — Prepare the Image

Click **Prepare Image with Drivers**.

Read the time warning — this process typically takes **45–90 minutes** depending on the number of editions selected and your hardware. The application will appear unresponsive during DISM driver injection operations — this is normal.

!!! warning
    Do **not** close the application during processing. DISM operations are running in the background even when the UI appears frozen.

The Operation Log at the bottom of the window shows real-time progress at each stage.

## Step 7 — Create the Bootable USB

Once image preparation completes:

1. Insert your USB drive (14GB+ required)
2. Click **Refresh** to detect it
3. Select your USB drive from the **3. Select USB Drive** dropdown
4. Confirm the data erasure warning
5. Wait for the USB to be formatted and files copied (~5–10 minutes)

!!! danger
    All existing data on the selected USB drive will be permanently erased. Double-check the drive selection before confirming.

## Step 8 — Done

Your bootable Windows 11 USB is ready. Boot the target device from the USB in UEFI mode to start the installation with all drivers pre-loaded.

---

## Save for Later

After the USB is created, the application asks if you want to save the prepared image. Saved images are stored in `C:\Win11ImagePrep\SavedImages\` (the default working root, configurable via **Tools > Options**) and can be used to create additional USB drives much faster — skipping the full ISO extraction and driver injection process.

---

## Next Steps

- [Windows Application](windows-app.md) — Full feature reference and all operation modes
- [Requirements](requirements.md) — Detailed hardware and software requirements
- [Troubleshooting](troubleshooting.md) — If something goes wrong
