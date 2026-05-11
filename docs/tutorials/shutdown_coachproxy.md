# Shutdown CoachProxy safely

Performing a proper software shutdown prevents SD card corruption and reduces the risk of boot problems. This guide lists safe shutdown methods and what to do in emergency situations.

## Recommended: use the CoachProxy UI (preferred)

If the CoachProxy web UI is available, use its System → **Shutdown** or **Reboot** option. This performs a controlled shutdown that is safest for the device and storage. Use the UI first when possible; the steps below remain available if the UI is unreachable.

## Command line shutdown (fallback)

If the UI is unavailable, connect locally or via Tailscale SSH and run:

```bash
sudo shutdown -h now
```

Wait for the system activity LED (the green "ACT" light on Raspberry Pi models) to stop blinking and turn off before removing power.


## Physically removing power

- After a confirmed software shutdown (UI or CLI) and when LEDs indicate the device is powered down, you may unplug the power connector.
- Use gentle, steady force when disconnecting any clip-style connectors.


## Emergency / unresponsive device

If the system is completely unresponsive and you cannot access the UI or SSH, pulling the power may be unavoidable. After an emergency unplug:

- Inspect the storage (SD card) for filesystem errors on the next boot and restore from backups if needed.
- Consider running a filesystem check on another machine or mounting the card to verify integrity.


## Tips

- Always keep recent backups or a master image handy for quick recovery.
- Consider using an external UPS if the device is in a location prone to power interruptions.
