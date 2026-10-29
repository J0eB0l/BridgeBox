# Spools, RFID and Spoolman

The AMS has its own RFID readers and reads Bambu tags itself. BridgeBox passes what it reads to AFC, so a tagged spool
fills in its lane with no extra hardware.

## Tagged spools

When a spool with a Bambu tag goes into a bay, the AMS reads the tag and AFC fills in the lane:

- material and color;
- print temperatures;
- spool weight;
- the tag's identity, used to find the same spool again later.

A spool without a tag loads and prints the same way. Set its material, color and weight on the lane yourself, or pick
its spool in Spoolman.

!!! note

    The AMS reads genuine Bambu tags. An NTAG215 or other third-party tag will not read in a Bambu bay.

## Spoolman

Add the Spoolman section to link tags to spools:

```ini
[AFC_BambuAMS_rfid]
```

It needs Spoolman set up in `moonraker.conf`. With it:

- **Each tag is linked to its spool.** With Spoolman 0.27 or newer, the tag is stored on the spool itself, so the
  spool is found again in whichever bay it goes into.
- **New spools from a tag.** For a tag Spoolman doesn't know yet, open the spool's **Add tag** dialog in Spoolman and
  put the spool in a bay: the scan fills it in. Or turn on `auto_spoolman_create` and AFC makes a new spool from the
  tag by itself:

    ```ini
    [AFC_BridgeBox chain1]
    auto_spoolman_create: True
    ```

- **Remaining weight.** What the AMS measures is written back to the spool. Set `sync_measured_to_spoolman: False` in
  the chain section to stop that.

## Measuring what is left

On the boxed units and the AMS HT, the AMS can turn a spool with a genuine Bambu tag and work out how much filament is
left from one revolution, using its encoder and the tag's data. The result shows up in Spoolman and in your printer's
web interface.

=== "On insert"

    `measure_on_insert` is on by default. Each tagged spool is measured when it goes in. The unit pulls filament for 8
    to 25 seconds to do it. Turn it off per model in a model section:

    ```ini
    [AFC_BridgeBox ht]
    measure_on_insert: False
    ```

=== "On demand"

    Measure a lane at any time, whatever `measure_on_insert` is set to:

    ```
    AFC_BAMBU_CAPSCAN LANE=lane2
    ```

!!! note

    Untagged spools can't be measured. They keep the weight Spoolman or you set, and AFC counts it down as it prints.
    The AMS Lite does not measure either; see [AMS Lite](ams-lite.md).

## Re-reading a tag

To read a tag again on a spool that is already seated:

```
AFC_BAMBU_SCAN LANE=lane2
```

[Next step: Drying](drying.md){ .md-button }
