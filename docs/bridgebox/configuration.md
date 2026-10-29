# Configuration

Everything a BridgeBox needs is in one section, `[AFC_BridgeBox <chain>]`, in `AFC/AFC_BridgeBox.cfg`. At startup it
builds an `[AFC_BambuAMS]` unit, its `[AFC_lane]` and `[AFC_hub]` sections, a temperature and humidity sensor for each
recorded unit, and one `[AFC_buffer]` for the chain.

!!! warning

    Write none of the `[AFC_BambuAMS]`, `[AFC_lane]`, `[AFC_hub]` or `[AFC_buffer]` sections for Bambu units yourself,
    and keep your other units off the Bambu lane numbers and bay names. A clash can stop Klipper from starting.

## A basic configuration

```ini
[AFC_BridgeBox chain1]
serial_port: auto
extruder: extruder
pool_ams: 4
pool_ht: 8
auto_drop: True
```

That is enough for most setups. The rest of this page covers the options you may want to change.

## Extruder

```ini
extruder: extruder
```

The `[AFC_extruder]` this chain feeds. One chain feeds one extruder. Each AMS feeds a load until that extruder's
`pin_tool_start` sensor sees filament, so the extruder needs a toolhead sensor.

??? tip "No toolhead sensor"

    With `buffer_type: bambu`, the chain's buffer also watches the AMS odometer, and can stand in for a toolhead
    sensor. Set `pin_tool_start: buffer` in the `[AFC_extruder]` the chain feeds:

    ```ini
    [AFC_BridgeBox chain1]
    buffer_type: bambu

    [AFC_extruder extruder]
    pin_tool_start: buffer
    ```

## Chain options

| Option | Default | Description |
|--------|---------|-------------|
| `serial_port` | required | `auto`, a USB port, or `tcp://<address>:8888`. See [USB and WiFi](connection.md). |
| `connection` | `usb` | `wifi` moves a USB BridgeBox to WiFi once it has joined a network, falling back to USB while the cable is in. |
| `wifi_ssid`, `wifi_password` | empty | With `connection: wifi`, the network Klipper gives the BridgeBox over USB, instead of setting it up from the hotspot. |
| `extruder` | `extruder` | The `[AFC_extruder]` this chain feeds. |
| `pool_ams` | `4` | Four-lane AMS bays to build, at most 4. Set it to the most AMS you will run: raising it later moves every HT lane up. |
| `pool_ht` | `8` | One-lane AMS HT bays to build. Keep it at 8 or below. |
| `ams_names` | `Bambu_AMS_1` ... `Bambu_AMS_4` | Names of the AMS bays, lowest lanes first. Only four are used. |
| `ht_names` | `Bambu_AMS_HT_1` ... `Bambu_AMS_HT_8` | Names of the AMS HT bays, lowest lanes first. |
| `unit_prefix` | `Bambu_AMS` | Prefix of the default bay names. On a second chain it is `Bambu_AMS_<chain>`. |
| `lane_base` | `0` | First Bambu lane number. `0` picks one past every other unit's lanes at the first start and saves it. |
| `roster` | empty | A `model:uid` list of known units. Leave it unset and the chain records each unit it sees. |
| `auto_drop` | `False` | Drop an unplugged unit's lanes and `T#` live once it has been offline for `release_grace`. Never during a print or while one of its lanes is loaded to the toolhead. The template sets it to `True`. |
| `release_grace` | `10` | Seconds offline before `auto_drop` drops a unit (minimum 2). |
| `enroll_grace` | `15` | Seconds a unit must stay online before its bay is saved for it. |
| `buffer_type` | `FPS_PSF` | `FPS_PSF` follows the AMS buffer's tension. `bambu` also watches the AMS odometer so the buffer can stand in for a toolhead sensor. |
| `dry_max_temp` | `0` | Drying ceiling in °C for every heated unit. `0` uses each model's own (AMS 2 Pro 65, AMS HT 85). A higher value never raises it. |
| `state_file` | this file | Where the chain keeps what it learns. |

??? note "Settle these before the first start"

    `pool_ams`, the bay names and `lane_base` decide which lanes each unit gets. Changing them later can move lanes,
    and moved lanes lose their saved details. See [Bays, Lanes and Hot-Swap](bays.md).

## Unit options

Any `[AFC_BambuAMS]` option set in the chain section applies to every unit the chain builds. A few you may want:

```ini
[AFC_BridgeBox chain1]
# Create a Spoolman spool for a tag Spoolman does not know yet.
auto_spoolman_create: True
# On a jam mid-print, unload, reload the same spool and resume.
auto_error_recovery: True
auto_error_recovery_limit: 1
```

| Option | Default | Description |
|--------|---------|-------------|
| `auto_spoolman_create` | `False` | Create a Spoolman spool from a Bambu tag Spoolman does not know yet. Needs `[AFC_BambuAMS_rfid]`. |
| `auto_error_recovery` | `True` | On a jam mid-print, unload, reload the same spool and resume the print. `False` leaves the print paused. |
| `auto_error_recovery_limit` | `1` | How many resumes in a row it may make by itself before it leaves the print paused. `0` recovers the filament but never resumes. |
| `measure_on_insert` | `True` | Measure how much filament a tagged spool holds when it is inserted. See [Spools, RFID and Spoolman](spools.md). |
| `sync_measured_to_spoolman` | `True` | Write the remaining weight the AMS measures back to Spoolman. |

!!! warning

    Keep `unit_uid`, `ams_index` and the `mc_*` options out of the chain section. `unit_uid` stops Klipper, and the
    others address a single unit, not a whole chain.

## Model and bay sections

A section named after a model applies to every unit of that model. A section named after a bay applies to that bay
alone. These win over the chain section.

```ini
# Every AMS HT
[AFC_BridgeBox ht]
measure_on_insert: False

# Every AMS 2 Pro
[AFC_BridgeBox ams2]
measure_on_insert: False

# Every AMS 1
[AFC_BridgeBox ams1]
measure_on_insert: False

# One bay
[AFC_BridgeBox Bambu_AMS_2]
measure_on_insert: False
```

!!! danger

    Never give a model or bay section a `serial_port`. A section with one is a second chain.

## Spoolman

```ini
[AFC_BambuAMS_rfid]
```

The AMS reads Bambu tags itself, so the units load, scan and name their filament without this section. Add it for the
Spoolman side: matching each spool by its tag and writing the remaining weight back to it. It needs Spoolman set up in
`moonraker.conf`. See [Spools, RFID and Spoolman](spools.md).

## A second BridgeBox

Add another section below the first, with its own `serial_port` and `extruder`:

```ini
[AFC_BridgeBox chain2]
serial_port: auto
extruder: extruder1
```

With `serial_port: auto`, it takes the BridgeBox no other chain uses. Its units are named `Bambu_AMS_chain2_1` and
so on by themselves. Leave `ams_names` and `ht_names` out of it, or give it names of its own. The installer's
additional unit menu writes this section for you.

## Saved state

The chain saves what it learns (the units it has seen, their names, lanes, `T#` and measured path lengths) at the end
of the file that holds its section, as lines starting with `#~#`. Klipper reads them as comments.

!!! warning

    Leave the `#~#` lines alone and keep your own lines above them. Everything below the managed-state marker line is
    rewritten by the chain.

[Next step: Bays, Lanes and Hot-Swap](bays.md){ .md-button }
