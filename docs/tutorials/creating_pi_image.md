
# Creating a Pi master image (macOS)

This guide shows several ways to create a master Raspberry Pi image on macOS for cloning or backups. Use caution — imaging tools can overwrite drives.

## Recommended: ApplePi‑Baker (GUI)

- Download and install ApplePi‑Baker v2 (or similar Pi imaging utility).
- Insert the source microSD/SSD and open the app.
- Select the source drive and choose **Create Backup** to produce a `.img` (or compressed) copy.
- To restore, select the target drive and use **Restore Backup**, choosing the saved image file.

ApplePi‑Baker provides a simple graphical workflow and verifies operations for you.

## Native macOS method (Terminal — dd)

Warning: `dd` is powerful and destructive if given the wrong device. Double-check device IDs before running.

1. Identify disks:

```bash
diskutil list
```

2. Unmount the source disk (replace `<N>` with the disk number, e.g. `2`):

```bash
diskutil unmountDisk /dev/disk<N>
```

3. Create the image (use `rdisk` for raw speed; replace `<N>`):

```bash
sudo dd if=/dev/rdisk<N> of=~/Desktop/CoachProxy_Master.img bs=1m status=progress
```

Notes:
- `if=` is the source device (the Pi drive).
- `of=` is the output file path where the image will be saved.
- `bs=1m` sets a larger block size for faster transfers.

## Alternative: Disk Utility

- Open Disk Utility, select the device in the sidebar.
- Choose **File → New Image → Image from [Device]** and set format to **DVD/CD master** to capture all sectors.

Disk Utility may produce `.dmg` files; convert if you need a raw `.img`.

## Restoring the image to a new drive

- Using ApplePi‑Baker: choose the saved image and **Restore** to the target drive.
- Using `dd` (reverse `if`/`of`):

```bash
sudo dd if=~/Desktop/CoachProxy_Master.img of=/dev/rdisk<TARGET_N> bs=1m status=progress
```

## Tailscale and identity considerations

When cloning a device that has Tailscale configured, clear the old device identity on the first boot of the restored system to avoid duplicate nodes:

```bash
sudo tailscale logout
```

Also consider changing the hostname and SSH keys on the cloned device to avoid conflicts:

```bash
# Change hostname (example)
sudo hostnamectl set-hostname <new-hostname>

# Regenerate SSH host keys (Debian-based example)
sudo rm /etc/ssh/ssh_host_* && sudo dpkg-reconfigure openssh-server
```

## Verification

- Mount the image or boot the device and confirm system services start as expected.
- Verify network and Tailscale connectivity after re-authenticating.

## Safety tips

- Always confirm device IDs with `diskutil list` before running destructive commands.
- Keep at least one verified backup copy off the device you are imaging.
- Inspect downloaded tools (like ApplePi‑Baker) from their official sources.
