# Firmware Updates

There are two kinds of firmware on a BridgeBox setup: the BridgeBox's own, and each AMS unit's. Both are updated from
Klipper over the link the BridgeBox already uses, with no Bambu printer needed.

## Updating the BridgeBox

BridgeBoxes ship already flashed, so this is only for updates. The update goes over USB or WiFi, whichever the
BridgeBox is on.

### What you need

Two files, which always travel together. Both are in `firmwares/BridgeBox/` in the AFC-Klipper-Add-On folder:

| Image | Manifest |
|-------|----------|
| `signed_s3.bin` | `signed_s3.bin.manifest.json` |

The BridgeBox runs only signed images, and the manifest is the signature. Without it, the BridgeBox refuses the update.

`firmwares/BridgeBox/bridgebox_flash.cfg` adds three short commands for the update. Upload it to your config folder and
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

Each AMS unit's own firmware is updated through the BridgeBox too: no Bambu printer and no cable to the AMS. You pick
the unit and the version from a menu, the image comes straight from Bambu, and Klipper keeps running.

Why update: an AMS 2 Pro only loads a tray while its dryer runs once it has Bambu's newer AMS firmware, and an AMS HT on
05.00.22.19 keeps heating while printing with a lane loaded.

### One-time setup

The updater is a script that runs beside Klipper. Set it up once:

1. **Check for `gcode_shell_command`.** Kalico has it built in. On Klipper, install the `gcode_shell_command` extra,
   as KIAUH does.
2. **Copy the updater into your config:**

    ```bash
    mkdir -p ~/printer_data/config/ams_firmware
    cd ~/AFC-Klipper-Add-On/firmwares/Bambu_AMS
    cp ams_flash.py ams_flash.cfg ams_firmware.json ~/printer_data/config/ams_firmware/
    ```

3. **Check the path in `ams_flash.cfg`.** Its `command:` line must hold the full path to `ams_flash.py`, because
   Klipper does not expand `~`. It is set for the user `pi`:

    ```ini
    command: /usr/bin/python3 /home/pi/printer_data/config/ams_firmware/ams_flash.py
    ```

    If your user is not `pi`, change `/home/pi` to your home folder. `echo $HOME` prints it.

4. **Include it** by adding this line to `printer.cfg`:

    ```ini
    [include ams_firmware/ams_flash.cfg]
    ```

5. **Let an update outlive a Klipper restart.** Run this once:

    ```bash
    sudo loginctl enable-linger $USER
    ```

6. **Restart Klipper** with `FIRMWARE_RESTART`.

The full guide is `firmwares/Bambu_AMS/AMS_FLASH_HOWTO.md` in the AFC-Klipper-Add-On folder.

### Update a unit

Open the menu one of three ways:

- on the [BridgeBox display](display.md#flash-ams): **Settings > Flash AMS**
- on the display's update page: the **AMS** card
- in the console: `AFC_BAMBU_FIRMWARE`

1. **Pick the unit.** Each one shows the firmware it runs and whether a newer one is out.
2. **Pick the version.** The menu lists the newest two from Bambu for that model, and any image you put in the
   folder yourself.
3. **Confirm.** The update takes a few minutes. The units on that BridgeBox read offline until it is done; the
   other units can stay connected. At the end a popup says how it went, for example
   `Bambu_AMS_1 now runs 05.00.22.22 (was 04.00.21.87).`, and the unit comes back online on its new firmware.

It refuses while a print is running or paused, and while a unit on the same BridgeBox is drying. The whole run is
logged in `~/printer_data/logs/ams_flash.log`. Without the menu:

```
AFC_BAMBU_FIRMWARE UNIT=Bambu_AMS_1 VERSION=05.00.22.22 CONFIRM=1
```

!!! tip "Offered updates"

    The first time the BridgeBox sees a unit, AFC reads its firmware. If a newer one is out, a popup offers it:
    **Update to ...** or **Not now**. Each unit is asked once, and again after `AFC_BRIDGEBOX_FORGET`. During a print
    the popup waits until the print ends.

!!! note "Without internet"

    Put Bambu's image (`n3f_rev5-firmware-v05.00.22.22-..._product.bin.sig`) or a Bambu printer's offline update
    package (`offline-ota-....zip`) in the folder beside `ams_flash.py`. The menu lists it with the others.

!!! warning "If an update stops partway"

    The unit waits in its bootloader. It is not bricked, but **keep it powered**: it has no firmware to boot until an
    update completes. Open the menu again and press **Finish** for that unit.

## Updating the display

See [Touchscreen Display](display.md#updating).

[Next step: Touchscreen Display](display.md){ .md-button }
