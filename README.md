# ChromeOS Factory Shim (Recovery USB)

This shim builds a **bootable recovery USB image** from the official ChromeOS recovery media and (optionally) drops your factory payload into the stateful partition. You then boot a device into recovery mode and insert the stick—no virtualization or host ChromeOS runtime is involved, and write protect stays untouched because everything happens from the external media.

## What the shim does
- Downloads the **official recovery image** for a target board (or a custom URL you provide).
- Optionally injects a factory payload directory into the stateful partition so it is available when recovery boots.
- Writes the prepared recovery image to a USB block device so you can boot straight into recovery mode.

## Prerequisites (run on the host that will write the USB)
- Shell access with `sudo` available (writing the stick and injecting payloads require root privileges).
- Tools: `curl`, `unzip`, `dd`, `losetup`, `mount`, `umount`, and `rsync` (these are present on ChromeOS in Dev Mode and most Linux hosts).
- Enough free space for the recovery download (a few GB) under `~/.local/share/chromeos-factory-shim` by default.

## Quickstart (build a recovery-mode factory stick)
```bash
# Download the official recovery image for your board (defaults to eve)
./shim init --board <board>

# (Optional) inject your factory payload into the stateful partition of the image
./shim inject --payload-dir /path/to/factory/files

# Write the recovery image to a USB block device (this erases the device)
sudo ./shim write --device /dev/sdX  # replace sdX with the USB device node
```

Boot the ChromeOS target into recovery mode (Esc+Refresh+Power or the equivalent key combo), insert the USB stick, and it will boot the official recovery system with your payload available under `/mnt/stateful_partition/factory-shim/`.

### Step-by-step (physical recovery flow, no host runtime)
1. **Copy or clone this repo** onto any Linux/ChromeOS host with `sudo` available.
2. **Download the recovery image** for your board:
   ```bash
   ./shim init --board <board>
   ```
3. **Inject a factory payload (optional):**
   ```bash
   ./shim inject --payload-dir /path/to/factory/files
   ```
   The payload is copied into the image’s stateful partition so it is present on first boot of recovery.
4. **Write the recovery USB:**
   ```bash
   sudo ./shim write --device /dev/sdX
   ```
   Replace `/dev/sdX` with the raw USB device (not a numbered partition). This overwrites the stick with the prepared recovery image.
5. **Boot the device into recovery mode** and insert the stick. Everything runs from the recovery image—no changes to the internal OS or firmware, and write protect stays intact.

## Do I need anything on the host after writing the stick?
No. The host only builds and flashes the recovery USB. Once written, the USB stick is the shim. If you later need to update the payload or board image, rerun the steps above and rewrite the stick.

## Payload notes
- Files are placed under `/mnt/stateful_partition/factory-shim/` in recovery so your scripts or binaries can be launched from there.
- To keep state across re-flashes, keep your source payload outside the repo and point `--payload-dir` at it each time.
- If you skip `--payload-dir`, the stick will boot a vanilla recovery image (still useful for board diagnostics).

## Clean up
Remove downloaded artifacts (the recovery ZIP and unpacked image):
```bash
./shim clean
```
