# Drying

The AMS 2 Pro and the AMS HT have built-in heaters. With a BridgeBox, you start and stop drying from Klipper and watch
the temperature and humidity from your printer's web interface.

| Unit | Heater | Ceiling |
|------|--------|---------|
| AMS 1 | No | - |
| AMS 2 Pro | Yes | 65 °C |
| AMS HT | Yes | 85 °C |
| AMS Lite | No | - |

Units without a heater still report their temperature and humidity where the hardware has the sensors.

## The dryer panel

`templates/AFC_Dryer.cfg` adds a web panel for every filament dryer on the printer. It finds every heated unit by
itself, AMS 2 Pro, AMS HT and ACE alike, each capped at its own ceiling.

1. Copy `templates/AFC_Dryer.cfg` into your `AFC` config folder and restart.
2. In Mainsail or Fluidd, add it as a webcam: **Settings → Webcams → Add**, service `iframe`, URL
   `http://<printer address>:8093/`.

The same address works on its own in any browser or on a phone. `AFC_DRYER_STATUS` prints where the panel is served
and which units it found.

## From the console

=== "Start"

    ```
    AFC_BAMBU_HEATER_START UNIT=Bambu_AMS_HT_1 TEMP=65 TIME=480
    ```

    `TEMP` is in °C and is capped at the unit's ceiling. `TIME` is in minutes. Add `ROTATE=1` to turn the spools while
    drying, on units that support it.

=== "Stop"

    ```
    AFC_BAMBU_HEATER_STOP UNIT=Bambu_AMS_HT_1
    ```

!!! tip "Rotating spools"

    A spool can only turn freely while drying if its filament is unloaded from the feeder.

## Lowering the ceiling

`dry_max_temp` in the chain section caps every heated unit at the lower of that value and the model's own ceiling. Set
it in a model or bay section to cap only those units:

```ini
[AFC_BridgeBox chain1]
dry_max_temp: 60
```

A higher value never raises a unit above its model's ceiling.

## Power for drying

!!! warning

    One AMS 2 Pro can dry on the BridgeBox's own supply. A second unit heating off the same wire would pull the supply
    down and reset every unit on the bus, so AFC refuses to start it while the first one is drying on bridge power, and
    says why. Plug an adapter into either unit, even mid-cycle, and the block lifts. An AMS HT always needs its own
    mains cord to heat.

## AMS firmware and drying

Some AMS firmware versions limit what can happen while drying:

- An AMS 2 Pro only loads a tray while its dryer is running once it has Bambu's updated AMS firmware. Before that, a
  load made while drying stalls and fails.
- An AMS HT on older firmware refuses to start drying with a lane loaded, which AFC reports as a refused dry.

Both are fixed by updating the AMS firmware; see [Firmware Updates](firmware.md#updating-the-ams-units).

[Next step: AMS Lite](ams-lite.md){ .md-button }
