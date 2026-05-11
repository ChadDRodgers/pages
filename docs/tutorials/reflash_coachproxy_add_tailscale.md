tailscale ip -4
tailscale status
# CoachProxy — Reflash & Tailscale Setup Guide

This guide walks through flashing a CoachProxy image to a microSD card, preparing legacy Debian Buster repositories (if needed), installing Tailscale, and verifying connectivity.


## 1. Flash the image

- Target: 32GB microSD card
- Recommended software: balenaEtcher or Raspberry Pi Imager
- Process: flash your CoachProxy `.img` to the card, insert it into the Pi, and wait ~2 minutes for the device to complete its initial boot and heartbeat.


## 2. SSH into the Raspberry Pi (local)

Before reflashing or installing software, connect to the Raspberry Pi from a machine on the same local network. Replace the placeholders below with your values — do not use real credentials here.

```bash
# Basic SSH (replace <username> and <hostname-or-ip> with your values)
ssh <username>@<hostname-or-ip>

# If using an identity file:
ssh -i /path/to/private_key <username>@<hostname-or-ip>

# If the SSH server uses a non-standard port:
ssh -p <port> <username>@<hostname-or-ip>
```

When connecting for the first time you may be prompted to accept the host key fingerprint. Authenticate using your account password or your SSH key as configured on the Pi. Ensure you're on a trusted local network when entering credentials.


## 3. Update legacy repositories (Debian Buster)

If the system shows "No Release file" errors, point apt to the legacy archive servers.

Update source lists:

```bash
sudo sed -i 's#raspbian.raspberrypi.org#legacy.raspbian.org#g' /etc/apt/sources.list
sudo sed -i 's#archive.raspberrypi.org#archive.raspberrypi.org#g' /etc/apt/sources.list.d/raspi.list
```

Confirm the changes:

```bash
cat /etc/apt/sources.list
cat /etc/apt/sources.list.d/raspi.list
```

Refresh the package index:

```bash
sudo apt update
```

## 4. Install Tailscale

Add the GPG key and repository, then install:

```bash
curl -fsSL https://tailscale.com | sudo apt-key add -
curl -fsSL https://tailscale.com | sudo tee /etc/apt/sources.list.d/tailscale.list
sudo apt update && sudo apt install -y tailscale
```

## 5. Authenticate and enable SSH

Start Tailscale and authenticate the device:

```bash
sudo tailscale up
```

When prompted, copy the URL printed to the terminal, open it in a browser, and sign in to authorize the Pi.

Enable Tailscale SSH (optional):

```bash
sudo tailscale set --ssh
```

## 6. Tailscale admin console and sharing

- Open the Tailscale Admin Console to view devices.
- Find your Pi (often `raspberrypi` or `coachproxy`) and confirm the status dot is green.
- Verify that `SSH` appears under the device Services.

If a device shows "Expired" or "Needs Re-authentication", use the three-dot menu next to the device in the console and choose **Disable Key Expiry** to avoid future lockouts.

To share access with another Tailscale user:

1. Click **Share...** on the device row.
2. Generate a share link and send it to the recipient. They can then access the CoachProxy Web UI via the device's Tailscale IP while the link is active.

## 7. Local verification

Run the following on the Pi to view the assigned Tailscale IP and connection status:

```bash
tailscale ip -4
tailscale status
```

## Notes

- Commands in this guide require `sudo` and network access. Use caution when running commands that modify system files.
- This guide focuses on Debian Buster / legacy setups; adapt repository lines as needed for other distributions or releases.
