# BridgeBox Commands

These commands are added by the `[AFC_BridgeBox]` chain and the Bambu AMS units it builds. They work alongside AFC's own
commands: loading, unloading, `T#`, `SET_MAP` and the rest work on Bambu lanes like on any other lane.

Most commands take `UNIT=<unit>`, the bay name of a unit such as `Bambu_AMS_1`, or `LANE=<lane>` for one lane. The
`AFC_BRIDGEBOX_*` commands take `CHAIN=<chain>` on a printer with more than one BridgeBox; the first chain also answers
without it. Each command prints its usage when run without arguments.

-----

## Status

### AFC_BAMBU_UIDS

Lists the units on the bus, read straight off the wire, and the BridgeBox firmware version.

```
AFC_BAMBU_UIDS UNIT=Bambu_AMS_1
```

-----

## Bays

See [Bays, Lanes and Hot-Swap](bays.md) for how bays work.

### AFC_BRIDGEBOX_BAYS

Opens the bay manager: every bay, its kind and the unit on it, with an Unassign button for each occupied bay.

```
AFC_BRIDGEBOX_BAYS
```

### AFC_BRIDGEBOX_ASSIGN

Moves a unit to a bay and saves it there. If the unit is online it moves live; an offline unit claims the bay when it
next comes online. Without `NAME`, it opens a bay picker.

```
AFC_BRIDGEBOX_ASSIGN UID=<unit uid> NAME=Bambu_AMS_2
```

### AFC_BRIDGEBOX_UNASSIGN

Frees a unit's bay but keeps its roster entry, its learned values and its saved lane details. A unit that is still
plugged in claims the same bay back on the next check; unplug it first to free the bay for another unit.

```
AFC_BRIDGEBOX_UNASSIGN UID=<unit uid>
```

### AFC_BRIDGEBOX_FORGET

Retires a unit: removes it from the roster, frees its bay, name and lanes, and erases what the chain learned about it.
Without arguments, it opens a picker listing every recorded unit.

```
AFC_BRIDGEBOX_FORGET NAME=Bambu_AMS_HT_2
```

### AFC_BRIDGEBOX_REPLACE

Puts a unit that found no free bay onto the bay of a unit that is offline, forgetting the old one. Without `OLD`, it
opens a popup to pick the bay.

```
AFC_BRIDGEBOX_REPLACE UID=<new uid> OLD=Bambu_AMS_3
```

!!! note

    FORGET, UNASSIGN, ASSIGN and REPLACE refuse a bay with a lane loaded to the toolhead. Unload it first, or add
    `FORCE=1` if the filament is already out.

-----

## Connection

### AFC_BRIDGEBOX_RESET_WIFI

Wipes the BridgeBox's WiFi network and link key and restarts it with its setup hotspot, for a move to another printer or
network. Refused during a print and while the BridgeBox is not connected.

```
AFC_BRIDGEBOX_RESET_WIFI
```

### AFC_BRIDGEBOX_FORGET_BOARD

Forgets the board a chain on `serial_port: auto` saved, and its WiFi address. After a `RESTART`, the chain takes the
next free BridgeBox it finds on USB.

```
AFC_BRIDGEBOX_FORGET_BOARD
```

-----

## Spools and tags

### AFC_BAMBU_SCAN

Reads a lane's tag again, on a spool that is already seated.

```
AFC_BAMBU_SCAN LANE=lane2
```

### AFC_BAMBU_CAPSCAN

Reads a lane's tag and measures how much filament the spool holds. Works whatever `measure_on_insert` is set to.

```
AFC_BAMBU_CAPSCAN LANE=lane2
```

-----

## Drying

See [Drying](drying.md).

### AFC_BAMBU_HEATER_START

Starts drying on an AMS 2 Pro or AMS HT. `TEMP` is in °C and is capped at the unit's ceiling, `TIME` is in minutes and
`ROTATE=1` turns the spools while drying.

```
AFC_BAMBU_HEATER_START UNIT=Bambu_AMS_HT_1 TEMP=65 TIME=480 ROTATE=0
```

### AFC_BAMBU_HEATER_STOP

Stops drying.

```
AFC_BAMBU_HEATER_STOP UNIT=Bambu_AMS_HT_1
```

-----

## Recovery

### AFC_BAMBU_RECOVER

Recovers a stuck or failed load: stops the motion and reels the lane's filament back to its bay.

```
AFC_BAMBU_RECOVER LANE=lane2
```

### AFC_BAMBU_CLEARFAULT

Clears a unit's latched fault, the way a Bambu printer does, and checks that it took. Refuses while the lane is still
jammed.

```
AFC_BAMBU_CLEARFAULT UNIT=Bambu_AMS_1
```

### AFC_BAMBU_FAULT_RELOAD

After a fault pauses a print, reloads the lane and resumes. Outside a print it works like a Bambu printer's Retry: it
loads the lane whose load last failed again. It does nothing when there is nothing to recover.

```
AFC_BAMBU_FAULT_RELOAD
```

### AFC_BAMBU_RELINK

Clears a unit's timeout or error state by relinking it.

```
AFC_BAMBU_RELINK UNIT=Bambu_AMS_1
```

-----

## Firmware

See [Firmware Updates](firmware.md).

### AFC_BAMBU_FLASH

Sends a firmware image to the BridgeBox, which writes its own flash. `APPLY=0` transfers and checks it without
writing; `ABORT=1` clears a stuck transfer. `UNIT` picks the BridgeBox through any unit on it.

```
AFC_BAMBU_FLASH UNIT=Bambu_AMS_1 FILE=signed_s3.bin APPLY=0
```

-----

## Advanced

These are for diagnosing a problem or tuning a unit by hand. You should not need them in normal use.

| Command | Description |
|---------|-------------|
| `AFC_BAMBU_FOLLOWER LANE=<lane> ENABLE=<0 or 1>` | Engage or stop the feeder following the extruder for a lane. |
| `AFC_BAMBU_FEED LANE=<lane> MM=<mm>` | Feed a set length from a lane toward the toolhead. |
| `AFC_BAMBU_PRIME LANE=<lane>` | Preload one bay with the feeder only. |
| `AFC_BAMBU_BITE UNIT=<unit> MM=<mm>` | Set how far the extruder grips past the toolhead sensor, until the next restart. |
| `AFC_BAMBU_ARRIVAL UNIT=<unit> ASSIST=<value>` | Set how long a load waits for the filament to arrive, until the next restart. |
| `AFC_BAMBU_REID LANE=<lane>` | Ask one bay to identify its spool again. |
| `AFC_BAMBU_BUFFER_PROBE UNIT=<unit>` | Print the AMS buffer's raw reading. |
| `AFC_BAMBU_SAVEIDS UNIT=<unit>` | Commit the configured unit UIDs to the BridgeBox. |
| `AFC_BAMBU_SNIFF UNIT=<unit> ON=<0 or 1>` | Put the BridgeBox into listen-only mode on the bus. |
| `AFC_BAMBU_AUTOSCAN UNIT=<unit> ON=<0 or 1>` | Turn the scan AFC sends when a spool goes in on or off. With it off, the AMS still reads the tag in its own sequence. |
