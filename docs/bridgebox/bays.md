# Bays, Lanes and Hot-Swap

An AMS can be plugged in or pulled out while Klipper runs. Its lanes, its `T#` commands and its place in the web
interface and the dryer panel appear and go away live, with no restart.

## Why bays

Klipper builds its lanes when it reads the config, and there is no way to add a lane to a running Klipper. So at
startup the BridgeBox chain builds a **pool of bays** ahead of time:

- **AMS bays**, four lanes each, up to `pool_ams` (at most 4);
- **AMS HT bays**, one lane each, up to `pool_ht`.

An empty bay is hidden: its lanes have no `T#` and AFC's PREP, the web interface and the dryer panel skip it. When a
unit comes online it **claims** a bay of its kind, and the bay's lanes and `T#` come to life. When it is unplugged and
`auto_drop` is on, it **releases** the bay again.

## The first claim

1. A new unit comes online and claims a free bay of its kind, lowest lanes first. Its lanes appear at once.
2. Once it has stayed online for 15 seconds (`enroll_grace`), the bay is **saved** for it. The console says:

    ```
    AFC_BridgeBox chain1: saved <UID> on Bambu_AMS_1 (lane0-lane3); it comes back there after a restart.
    ```

3. From then on the bay is reserved for that unit. After a restart, or when it is plugged back in, it returns to the
   same lanes and `T#`, with its spools and settings.

A **New AMS** popup in Mainsail or Fluidd tells you which bay the unit took and offers the other free bays of the same
kind, in case you want it elsewhere. Dismiss keeps it where it is.

## Lane layout

Lanes are laid out in fixed bands, so a unit's lanes never move when other units come and go:

- AMS bay *N* takes four lanes starting at `lane_base + 4 × (N - 1)`.
- The AMS HT bays follow the AMS band, one lane each.

With `lane_base: 0` and the default pools:

| Bay | Lanes |
|-----|-------|
| `Bambu_AMS_1` | lane0 - lane3 |
| `Bambu_AMS_2` | lane4 - lane7 |
| `Bambu_AMS_3` | lane8 - lane11 |
| `Bambu_AMS_4` | lane12 - lane15 |
| `Bambu_AMS_HT_1` | lane16 |
| ... up to `Bambu_AMS_HT_8` | lane23 |

The HT band starts in the same place whether one AMS is plugged in or four.

### `lane_base`

With the default `lane_base: 0`, the chain picks the base at the first start: one past the highest lane any other unit
uses, or 0 when the BridgeBox is the only unit. Beside a BoxTurtle on lane0 - lane3 that gives 4, so `Bambu_AMS_1` is
lane4 - lane7. The base is then saved, so lanes added to other units later do not move the Bambu ones.

!!! warning

    Set `lane_base` before the first start if you want the Bambu lanes somewhere specific. Changing it later moves
    every Bambu lane, and those lanes lose their saved details.

## `T#` commands

`T#` does not follow lane numbers. A claimed lane takes the next unused `T#`, the same way AFC numbers any lane. On its
own, the first AMS gets `T0` - `T3`; beside a unit already on `T0` - `T3`, it gets `T4` - `T7`.

- A unit with a saved bay keeps its `T#` while it is unplugged and across restarts.
- `SET_MAP` and AFC's other mapping commands change a Bambu lane's `T#` like any other lane, and the change is saved.
- When a unit is retired with `AFC_BRIDGEBOX_FORGET`, the next start packs the other Bambu `T#` down into the freed
  ones. Lane numbers and names stay.

## Unplugging a unit

=== "auto_drop: True"

    Once a unit has been offline for `release_grace` (10 seconds), its lanes and `T#` drop live and an **AMS removed**
    popup appears. Its bay stays reserved, so plugging it back in brings it back on the same lanes.

    A unit is never dropped during a print, or while one of its lanes is loaded to the toolhead.

=== "auto_drop: False"

    The unit's lanes stay until the next restart. Plug it back in and it carries on where it was.

## Managing bays

| Command | What it does |
|---------|--------------|
| `AFC_BRIDGEBOX_BAYS` | Opens the bay manager: every bay, who is on it and an Unassign button for each. |
| `AFC_BRIDGEBOX_ASSIGN UID=<uid> NAME=<bay>` | Moves a unit to another bay and saves it there. Without `NAME` it opens a bay picker. |
| `AFC_BRIDGEBOX_UNASSIGN UID=<uid>` | Frees a unit's bay but keeps what the chain learned about the unit. |
| `AFC_BRIDGEBOX_FORGET UID=<uid>` | Retires a unit for good and frees its bay. Without arguments it opens a picker. |
| `AFC_BRIDGEBOX_REPLACE UID=<new> OLD=<old>` | Puts a new unit on an offline unit's bay. |

Use a unit's UID or its bay name. `AFC_BAMBU_UIDS` lists the UIDs on the bus. On a printer with more than one
BridgeBox, add `CHAIN=<name>` for any chain but the first, such as `AFC_BRIDGEBOX_BAYS CHAIN=chain2` (see
[More than one BridgeBox](commands.md)).

!!! note "A lane loaded to the toolhead"

    FORGET, UNASSIGN, ASSIGN and REPLACE refuse a bay with a lane loaded to the toolhead. Unload the lane first. If
    the filament is already out, `FORCE=1` clears AFC's record of it.

### No free bay

When every bay of a unit's kind is taken, the new unit waits. If one of the bays belongs to a unit that is offline, a
**No free bay** popup offers to replace it. Replacing forgets the old unit and moves the new one onto its bay, lanes
and `T#`.

## Turning hot-swap off

`pool_ams: 0` together with `pool_ht: 0` turns live claiming off. The chain then only records the units it sees, and
their lanes come up only through `AFC_BRIDGEBOX_ASSIGN`, again after every restart. Most setups should leave the pool
on.

[Next step: Spools, RFID and Spoolman](spools.md){ .md-button }
