# Firmware Updates

There are two kinds of firmware on a BridgeBox setup: the BridgeBox's own, and each AMS unit's. Both are updated from
Klipper over the link the BridgeBox already uses, with no buttons to hold and no Bambu printer.

## Updating the BridgeBox

BridgeBoxes ship already flashed, so this is only for updates. The update goes over USB or WiFi, whichever the
BridgeBox is on.

### What you need

Two files, which always travel together. Both are in `Firmwares/BridgeBox/` in the AFC-Klipper-Add-On folder:

| Image | Manifest |
|-------|----------|
| `signed_s3.bin` | `signed_s3.bin.manifest.json` |

The BridgeBox runs only signed images, and the manifest is the signature. Without it, the BridgeBox refuses the update.

`Firmwares/BridgeBox/bridgebox_flash.cfg` adds three short commands for the update. Upload it to your config folder and
add `[include bridgebox_flash.cfg]` to `printer.cfg`, then `RESTART`:

| Command | Does |
|---------|------|
| `BRIDGEBOX_FLASH` | Check: transfer and verify the image, then throw it away. |
| `BRIDGEBOX_FLASH MODE=go` | Update: write the image and reboot. |
| `BRIDGEBOX_FLASH_ABORT` | Clear a stuck transfer. |

### Steps

1. **Make sure the printer is idle.** The update refuses to run during a print.
2. **Note the current version** with `AFC_BAMBU_UIDS`. The firmware line reads something like `AFC-2.90`.
3. **Upload both files** to the top level of your Klipper config folder, next to `printer.cfg`.
4. **Dry run.** This transfers and checks the image, then discards it. The BridgeBox keeps running what it had.

    === "With bridgebox_flash.cfg"
        ```
        BRIDGEBOX_FLASH UNIT=Bambu_AMS_1
        ```
    === "Without it"
        ```
        AFC_BAMBU_FLASH UNIT=Bambu_AMS_1 FILE=signed_s3.bin APPLY=0
        ```

    `UNIT=` can be any unit on that BridgeBox; it picks the BridgeBox, not the AMS. Look for
    `signed manifest vNNN accepted` and `the image transferred and verified, and was then discarded`.

5. **Apply.**

    === "With bridgebox_flash.cfg"
        ```
        BRIDGEBOX_FLASH UNIT=Bambu_AMS_1 MODE=go
        ```
    === "Without it"
        ```
        AFC_BAMBU_FLASH UNIT=Bambu_AMS_1 FILE=signed_s3.bin
        ```

    The BridgeBox writes its own flash and reboots. The link drops for about ten seconds and the AMS units go offline,
    then come back by themselves. The last message is usually
    `apply sent; the bridge stopped answering without confirming`. That is normal: the BridgeBox reboots before its
    reply gets out.

6. **Confirm** with `AFC_BAMBU_UIDS` once the units are back. The firmware line should show the new version.

!!! tip

    Always pass `FILE=` to `AFC_BAMBU_FLASH`. Without it the command looks for a different file name.

### What can go wrong

Nothing on the BridgeBox is erased until the whole image has arrived and checked out, so almost every failure leaves it
running the old firmware. A power cut during the apply is safe too: the new image goes into a spare slot, and the old
one keeps running until the new one has been checked.

| Message | Meaning |
|---------|---------|
| `the bridge rejected the manifest` | The signature does not match, or the image is older than what the BridgeBox runs. It will not go backwards. Nothing changed. |
| `the bridge refused the image` | The BridgeBox turned it down before writing anything. The message says why. |
| `no .manifest.json beside the image` | Upload the manifest next to the image, with its name unchanged. |
| `the link died mid-transfer` / `the image did not verify` | The transfer was interrupted. Nothing changed; run it again. |
| `busy: a transfer is already running` | Clear it with `BRIDGEBOX_FLASH_ABORT`, then run it again. |
| The version did not change | The apply did not take. Run step 5 again. |

## Updating the AMS units

The AMS units' own firmware can be updated over the bus from Klipper, with no Bambu printer. There is one command per
model, and each only sends its own model's file:

| Model | Command | File |
|-------|---------|------|
| AMS 2 Pro | `AFC_BAMBU_AMS_UPDATE` | `ams2_artifact.json` |
| AMS HT | `AFC_BAMBU_HT_UPDATE` | `ht_artifact.json` |
| AMS 1 | `AFC_BAMBU_AMS_ONE_UPDATE` | not available yet |

Why update: an AMS 2 Pro only loads a tray while its dryer runs once it has Bambu's updated firmware, and an AMS HT on
older firmware refuses to dry with a lane loaded.

!!! danger "One unit at a time"

    Only the unit being updated may be connected to the BridgeBox. Unplug every other AMS first. The command checks
    this in Klipper and again on the bus, and refuses if it finds a second unit.

The update needs a one-time setup (Klipper's `gcode_shell_command` and the updater files). Follow
`Firmwares/Bambu_AMS/AMS_FLASH_HOWTO.md` in the AFC-Klipper-Add-On folder. In short:

1. Connect only that unit and wait until it shows online.
2. `AFC_BAMBU_AMS_UPDATE MODE=check` runs read-only checks and ends with `READ-ONLY CHECKS PASSED`.
3. `AFC_BAMBU_AMS_UPDATE MODE=go` stops Klipper, rewrites the unit's firmware and restarts Klipper. It takes a few
   minutes; follow it in `~/printer_data/logs/ams_flash.log`. Success is
   `FLASH CONFIRMED: the loader verified the image and reset into the new firmware.`
4. Reconnect your other units and restart Klipper.

!!! warning

    If an update stops partway, the unit waits in its bootloader. **Do not power it off.** Run the same `MODE=go`
    command again and it picks the unit up from there.

## Updating the display

See [Touchscreen Display](display.md#updating).

[Next step: Touchscreen Display](display.md){ .md-button }
