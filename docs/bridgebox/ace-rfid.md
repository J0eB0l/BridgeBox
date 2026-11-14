# ACE RFID

The ACE Pro and ACE Pro 2 are documented in the AFC docs:
[ACE commands](https://www.afcproject.dev/klipper/internal/ace/) and the
[`[AFC_ACE]` section](https://www.afcproject.dev/configuration/AFC_UnitType_1.cfg/#afc_ace-unit_name-section).
This page covers only what BridgeBox adds to them: reading spool tags.

## ACE Pro

The ACE Pro reads Anycubic tags itself. When a tagged spool goes into a slot, AFC asks the ACE for the tag and fills in
the lane's material, color and print temperatures. Anything already set on the lane is kept. There is nothing to
configure.

To read a slot's tag again and replace the lane's values with it:

```
AFC_ACE_RFID_RESCAN UNIT=Ace_1 LANE=lane1
```

Nothing moves on an ACE Pro, so this works on the lane in the toolhead too.

## ACE Pro 2

The ACE Pro 2 has a tag reader for each pair of slots, but its stock firmware does not hand the reads to the host.
With the AFCACE2 firmware from AFC's `firmwares/ACE2` folder, AFC drives the readers itself. That lets it read many more tag types than the ACE does:

- MIFARE Classic: Bambu (needs the Bambu key), Snapmaker, Creality CFS (needs the Creality keys), BTT and QIDI;
- NTAG: Anycubic, Elegoo, TigerTag, OpenSpool, OpenTag3D and SpoolEase.

Add the section next to your `[AFC_ACE2]` unit:

```ini
[AFC_ACE2_rfid]
lane_slot_map: lane1:0, lane2:1, lane3:2, lane4:3
#auto_spoolman_create: False
#bambu_master_key:
#creality_key:
#creality_encryption_key:
```

When a spool goes in, AFC stages it to the hub and reads the tag as the spool turns past the reader. If the read
misses, it backs the filament off in small steps and tries again. It then fills in the lane and links the tag to the
spool in Spoolman.

| Option | Default | What it does |
|---|---|---|
| `lane_slot_map` | | Which ACE slot (0 to 3) each lane is on. The unit's own slot map wins when it has one. |
| `auto_spoolman_create` | `False` | Create a Spoolman spool for a tag that matches none. |
| `bambu_master_key` | | Key for reading Bambu tags. Without it, Bambu tags are skipped. |
| `creality_key`, `creality_encryption_key` | | Both are needed to read Creality CFS tags. |
| `read_on_insert` | `True` | Read the tag when a spool goes in. |
| `disable_rfid` | `False` | Stage spools without reading tags. `ACE_RFID_READ` still works. |
| `auto_tag_adjust` | `True` | Move an idle sibling spool off the shared reader so it does not answer for this one. When off, the console asks you to turn it by hand. |
| `scanner_lanes` | | Lanes used only as a tag scanner, for `ACE_RFID_SCAN`. |

!!! note

    Keys are never shipped with AFC. Leave them unset unless you have your own.

### Commands

| Command | What it does |
|---|---|
| `AFC_ACE_RFID_RESCAN UNIT=<unit> LANE=<lane>` | Winds the lane back, feeds it past the reader, reads the tag and feeds the lane back. The lane must be staged and not in the toolhead, and no print running. |
| `ACE_RFID_READ LANE=<lane>` or `SLOT=<n>` | Reads the tag in place and reports it. |
| `ACE_RFID_SCAN [LANE=<lane>] [SECONDS=<n>]` | Watches a scanner lane for a tag you hold to it and stages that spool as the next one loaded. |
| `AFC_ACE_RFID_DUMP UNIT=<unit> SLOT=<n>` | Prints everything the ACE reports for a slot's tag. |
