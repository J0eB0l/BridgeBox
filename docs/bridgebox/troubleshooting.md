# Troubleshooting

Start with `AFC_BAMBU_UIDS`. It lists every unit the BridgeBox sees on the bus and the BridgeBox firmware version, so it
tells you straight away whether the problem is between Klipper and the BridgeBox, or between the BridgeBox and the AMS.

!!! Question

    `AFC_BAMBU_UIDS` says the bridge is not connected.

??? Answer

    Klipper can't reach the BridgeBox.

    - On USB, check the cable and that the BridgeBox has power. With `serial_port: auto`, the chain keeps the board it
      saved; if you swapped boards, run `AFC_BRIDGEBOX_FORGET_BOARD` and `RESTART`.
    - On a second chain with `serial_port: auto`, make sure each BridgeBox is plugged in. Each chain takes a board no
      other chain uses.
    - On WiFi, check the address hasn't changed. Give the BridgeBox a fixed DHCP lease in your router.
    - Never list the BridgeBox in an `[mcu]` section. It is not a Klipper MCU.

!!! Question

    The BridgeBox is connected, but a unit is missing.

??? Answer

    - Check the unit has power. The units are powered by the BridgeBox's 24 V supply.
    - Check the cable from the BridgeBox to the buffer, and between units, is seated. A lit AMS does not prove the bus
      cable is connected.
    - A unit only claims a bay once AFC's PREP has run after a restart. Give it a few seconds.
    - Run `AFC_BRIDGEBOX_BAYS`. If every bay of the unit's kind is taken, it waits for one; see
      [No free bay](bays.md#no-free-bay).

!!! Question

    My Bambu lanes moved to other lane numbers.

??? Answer

    Lanes move when `pool_ams`, `ams_names`, `ht_names` or `lane_base` change, or when another unit takes one of their
    lane numbers. Settle those before the first start. See [Lane layout](bays.md#lane-layout).

!!! Question

    A load feeds but never finishes.

??? Answer

    Each AMS feeds until the extruder's toolhead sensor sees filament. Check that the `[AFC_extruder]` the chain feeds
    has a working `pin_tool_start`, or `pin_tool_start: buffer` with `buffer_type: bambu` on the chain. See
    [Extruder](configuration.md#extruder).

!!! Question

    A spool's tag doesn't read.

??? Answer

    The AMS reads genuine Bambu tags only; an NTAG215 or other third-party tag will not read in a Bambu bay. For a
    Bambu tag, reseat the spool or run `AFC_BAMBU_SCAN LANE=<lane>`. An untagged spool still loads and prints; set the
    lane by hand or pick its spool in Spoolman.

!!! Question

    A second unit refuses to start drying.

??? Answer

    One AMS 2 Pro can dry on the BridgeBox's own supply. AFC won't start a second heater on the same supply, because
    it would reset every unit on the bus. Plug an adapter into one of the units. An AMS HT always needs its own mains
    cord to heat. See [Drying](drying.md#power-for-drying).

!!! Question

    An AMS HT refuses to dry with a lane loaded, or an AMS 2 Pro load fails while drying.

??? Answer

    Both are fixed by Bambu's updated AMS firmware. See [Updating the AMS units](firmware.md#updating-the-ams-units).

!!! Question

    I moved a BridgeBox to another printer and it keeps dropping.

??? Answer

    The old printer is still reaching for it at its old address. On the old printer, set the chain to
    `connection: usb` or remove its section, then run `AFC_BRIDGEBOX_RESET_WIFI` on the new printer, or hold the BOOT
    button for 10 seconds, and set it up again. See [Moving to another printer](connection.md#moving-to-another-printer-or-network).

!!! Question

    The print paused with a BridgeBox link error.

??? Answer

    AFC pauses a print after 5 seconds without hearing from the BridgeBox. Check its power and its USB cable or WiFi
    signal. Once it is back, the units come back online by themselves and you can resume.

!!! Question

    A firmware update failed.

??? Answer

    See [What can go wrong](firmware.md#what-can-go-wrong). Nothing is erased until the whole image has arrived and
    checked out, so a failed update leaves the BridgeBox running its old firmware.

## Getting help

If you're still stuck, collect `~/printer_data/logs/klippy.log` and `~/printer_data/logs/AFC.log` and ask on the
[AFC Discord](https://discord.gg/z2tgWEnfDT).
