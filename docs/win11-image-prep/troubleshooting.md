# Troubleshooting

## Application appears frozen / stuck at a percentage

**Expected behaviour.** DISM driver injection operations run silently in the background and take 10–15 minutes each. The UI does not update during these periods.

- Do not close the application
- Watch the Operation Log — it updates between major steps
- Total expected time is 45–90 minutes for a full run with multiple editions
- If you selected all editions (typically 5–10), multiply per-edition time accordingly

---

## "Access Denied" or administrator errors

**Cause:** The application is not running with elevated privileges.

**Fix:** Right-click the executable or script and select **Run as administrator**. The application will also prompt automatically to restart with elevation.

To verify you are running as admin in PowerShell:

```powershell
([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```

This should return `True`.

---

## "Execution Policy" error (PowerShell script only)

**Cause:** Windows is blocking script execution.

**Fix:** Bypass the execution policy for the current session only:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\WinImagePrep_V3.ps1
```

This does not change the system-wide policy.

---

## "Error 0x800F0823" — Access denied during DISM

**Cause:** A WIM image is already mounted from a previous run or failed operation.

**Fix:** Click **Repair/Cleanup** in the application. This force-dismounts all mounted images and clears the working directories.

Alternatively, run manually in an elevated PowerShell prompt:

```powershell
# List mounted images
Get-WindowsImage -Mounted

# Force unmount all
Get-WindowsImage -Mounted | ForEach-Object {
    Dismount-WindowsImage -Path $_.Path -Discard
}

# Clean working directories
Remove-Item -Path C:\WinImagePrep\Mount\* -Recurse -Force
Remove-Item -Path C:\WinImagePrep\Windows11\* -Recurse -Force
Remove-Item -Path C:\WinImagePrep\Drivers\* -Recurse -Force
```

---

## "Not enough disk space"

**Cause:** Less than 25GB is available on `C:`.

**Fix:** Free up space before retrying:

- Delete `C:\Windows\Temp\` contents
- Run **Disk Cleanup → Clean up system files** to remove Windows Update files
- Check `C:\WinImagePrep\` for leftover working files from previous runs and delete them

---

## "No USB drives found"

**Cause:** The USB drive is not being detected.

**Fix:**

- Ensure the USB drive is physically connected
- Try a different USB port
- Click the **Refresh** button in the application
- Open **Disk Management** (`diskmgmt.msc`) to confirm the drive appears in Windows
- Try a different USB drive

---

## USB shows "Not formatted" after creation completes

**Cause:** Windows sometimes shows this briefly while the FAT32 volume is being registered.

**Fix:**

- Safely eject and reinsert the USB
- If prompted to format, click **Cancel** — do not reformat
- Verify files exist on the drive in File Explorer before assuming it failed
- Test by booting a device from the USB in UEFI mode

---

## "The file install.wim was not found"

**Cause:** Some Windows ISOs use `install.esd` instead of `install.wim`. The tool requires `install.wim`.

**Fix:** Convert the ESD to WIM first using DISM in an elevated PowerShell prompt:

```powershell
# Check available edition indexes
dism /Get-WimInfo /WimFile:install.esd

# Export each edition to install.wim (repeat for each index needed)
dism /Export-Image /SourceImageFile:install.esd /SourceIndex:1 /DestinationImageFile:install.wim /Compress:max /CheckIntegrity
```

Run the export command for each edition index, then use the resulting `install.wim` alongside the other ISO files.

---

## Antivirus blocking the script or application

**Cause:** Security software flagging PowerShell scripts or executables that use DISM, disk partitioning, or WPF controls.

**Fix:** This is a false positive. Options:

- Add an exclusion for the file in your antivirus settings
- Use the Windows Application instead of the script (compiled binaries are less commonly flagged)
- Review the [open source code](https://github.com/andy-kemp/Win11ImagePrep) to verify the tool before adding an exclusion

---

## Operation fails mid-way / application crashes

If the application crashes or is forcibly closed during an operation, WIM images may be left in a mounted state. On the next launch, use **Repair/Cleanup** to recover.

For manual cleanup:

```powershell
# Unmount all stuck WIM images
Get-WindowsImage -Mounted | ForEach-Object {
    Dismount-WindowsImage -Path $_.Path -Discard
}

# Dismount any ISO images
Get-DiskImage | Where-Object Attached | ForEach-Object {
    Dismount-DiskImage -ImagePath $_.ImagePath
}

# Clean working directories
Remove-Item -Path C:\WinImagePrep\Windows11\* -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item -Path C:\WinImagePrep\Drivers\* -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item -Path C:\WinImagePrep\Mount\* -Recurse -Force -ErrorAction SilentlyContinue
```

---

## Performance — operations are very slow

- **Use an SSD** for `C:\WinImagePrep\` — the working directory is heavily I/O intensive during WIM mounting and driver injection
- **Disable real-time antivirus scanning** during the operation — scanning every file written by DISM significantly increases time
- **Select fewer editions** — each edition adds 10–15 minutes; choose only the editions you plan to deploy
- **Close other applications** to free RAM for DISM operations

---

## Reporting Bugs

Open an issue on the [GitHub repository](https://github.com/andy-kemp/Win11ImagePrep/issues) and include:

- Windows version and build number
- Tool version (Windows Application version or script file name)
- Full contents of the Operation Log from the session
- The exact error message received
