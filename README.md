<p align="center"><img alt="BridgeBox. Use your Bambu AMS with Klipper. The closed BridgeBox case beside a board from the first batch." src="assets/readme/hero-dark.webp" width="100%"></p>

# BridgeBox

BridgeBox connects a stock Bambu AMS to a Klipper printer through AFC. No need to open the AMS, replace its board, or change its firmware.

Running on hardware: AMS 1, AMS 2 Pro, AMS HT. In testing: AMS Lite.

The full page: **[j0eb0l.github.io/BridgeBox](https://j0eb0l.github.io/BridgeBox/)**

## Why BridgeBox?

Bambu makes useful AMS hardware, but normally you need a Bambu printer to use it. You can't just plug an AMS into a Klipper machine and expect it to work.

BridgeBox gives Klipper users another option. Whether you're running a Voron, a custom build, or another AFC-compatible printer, you can use an AMS without swapping its electronics.

## How it works

BridgeBox plugs into the four-pin connector on the filament buffer, where a Bambu printer would normally connect. It communicates with the AMS over its existing connection while AFC handles everything from Klipper.

**Nothing on the AMS needs to change.** You don't have to replace a board, flash custom AMS firmware, or open the case. Want to use the AMS with a Bambu printer again? Disconnect BridgeBox and plug the AMS back into it.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/readme/wiring-dark.webp">
  <img alt="Three wiring paths compared. With a Bambu printer, the printer drives the RS-485 bus to the filament buffer and on to the AMS. With BridgeBox, a Klipper host connects over USB or WiFi to BridgeBox, which drives the same RS-485 bus to the same unmodified buffer and AMS. An AMS Lite has no buffer, so BridgeBox is wired straight to it, in the place of the A1&#x27;s cable." src="assets/readme/wiring-light.webp" width="100%">
</picture>

<sub>The only thing that changes is who leads the conversation. The buffer, the six-pin cable and the AMS itself are untouched, running the firmware they shipped with. An AMS Lite has no buffer, so BridgeBox plugs straight into it.</sub>

## What BridgeBox supports

### Automatic filament changes

Multi-color and multi-material prints work the way they do on a Bambu machine. Load, unload, the buffer holding tension — all of it driven from Klipper.

### RFID spool detection

Put a tagged spool in a bay and the material, color and temperatures arrive by themselves. No typing in what you just loaded.

With Spoolman 0.27 or newer, each tag is linked to its spool, so the spool is found again in whichever bay it goes into. For a tag Spoolman doesn't know yet, open its Add tag dialog and put the spool in a bay: the scan fills it in. Or turn on `auto_spoolman_create` and AFC makes a new spool from the tag.

### Remaining filament tracking

The AMS turns the spool and measures it, by default as soon as a tagged spool goes into a bay. It is a real figure, not a running total that drifts, and it lands in Spoolman and on the spool cards in your web interface.

It works it out from the reel itself. The AMS turns the spool and counts filament off the encoder until the RFID tag comes back round past the reader — one revolution tells it how much is still wound on, and that becomes the percentage.

So measuring needs a genuine Bambu tag. An untagged or third-party spool loads and prints exactly the same; it just doesn't get measured, and its lane keeps whatever weight you or Spoolman set. The same goes for every spool on an AMS Lite, which has no way to measure one.

### Filament drying

On the units with heaters, start and stop a drying cycle from the printer's interface and watch the chamber temperature and humidity while it runs.

Turn on rotation and the AMS also turns the spools as it dries — a bay at a time, every few minutes — so a coil dries through rather than on one side. It will only turn a bay whose filament is out of the feeder, so unload those and leave the spools free to spin.

### Error detection and recovery

A jam or a failed feed is caught, named in plain words, and retried. If it still can't recover it stops the print rather than carrying on with no filament.

Knock the cable out mid-print and it pauses within seconds — before the extruder can grind away at an AMS nobody is driving. A load already running rides out a brief blip and finishes.

When a spool runs out mid-print, the empty bay is caught within two seconds. The print pauses, or carries on from a runout lane on another extruder, and the AMS keeps pushing the leftover through so it can be pulled out.

### Multiple AMS units

Chain up to four AMS 1 or AMS 2 Pro alongside eight HTs, mixed however you like — twenty-four bays on one bridge. That ceiling is Bambu's own, not ours: it is the room their bus leaves for addresses. The units claim theirs at power-up, exactly as they would behind a Bambu printer.

Need more? Add a second BridgeBox. Each one runs its own chain, AFC names its units so they never clash, and its tool numbers carry on from the first.

## AMS Lite support

*New, and in testing.*

The AMS Lite is the open four-spool changer Bambu sells for the A1 and A1 mini. It has no enclosure, no buffer and no heater, so BridgeBox runs it a little differently from the other units.

**It has its BridgeBox to itself.** BridgeBox goes on the AMS Lite's cable in the A1's place. Nothing else can share that bus, and AFC refuses a setup that tries, so boxed units and HTs go on a second BridgeBox on the same printer.

**A toolhead sensor is optional.** A boxed AMS feels the filament arrive at its buffer. An AMS Lite has nothing like that, so it feeds until the filament reaches the toolhead, then lets go and the extruder pulls, the same way an A1 runs it. Its four tubes meet at a splitter at the toolhead. A filament switch after the splitter, before the extruder gears, ends every load for all four trays, and on a Snapmaker U1 the U1's own sensor counts. Without one, the AMS Lite's own odometer ends the load: when the tip reaches the extruder gears it stops moving, and the feed stops there.

**It learns its own path.** On the first load it measures the run from the AMS Lite to the sensor, or to the gears, with its odometer and keeps that figure. After that, a load that feeds 250 mm past it without arriving is stopped, instead of pushing filament somewhere it shouldn't go. A feed that stops well short of it isn't taken as arrived. BridgeBox waits for the AMS Lite's own stall report and only then fails the load as a jam, so a brief hesitation never stops a print.

**The odometer keeps watch.** The AMS Lite counts the filament leaving each tray in 1.5 mm steps, even with its motor off. Once a load ends, the extruder takes over, and the count has to move with it. That proves the gears have hold of the filament; if the count stands still, the load fails instead of printing into the air. While printing, an extruder that keeps pushing while the count stands still means a clog, and AFC pauses the print. The AMS Lite reports a stuck or tangled spool itself, and BridgeBox stops the load and names the tray.

**Tags work as on the other units.** When a spool goes in, BridgeBox asks the AMS Lite to read its tag, the way an A1 does. A Bambu tag brings the material, color, temperatures and spool weight, along with the tag's own UID and the reel's UUID, so Spoolman links the spool and finds it again in any tray. An untagged spool loads and prints the same; you set the lane.

**It doesn't measure what's left.** A boxed AMS can turn a tagged spool to measure it. The AMS Lite can't, and it doesn't on an A1 either, so its lanes keep the weight Spoolman or you set.

**Unloads stop short.** Its spring-loaded holders take back only a little filament, so it never reels the tube home. With a sensor, it pulls back until the sensor clears, then 30 mm more (set that to clear your splitter). Without one, it pulls back a set distance, counted on the odometer. Either way it parks there, out of the other trays' way and ready for its next load. To take a spool out, eject the lane: the AMS Lite lets go and you pull the filament out by hand.

**Where it stands.** On a test printer it comes online, reads its tags, UIDs included, and shows a lane as loaded only while its tray holds a spool. Running without a sensor and the odometer checks for the path, jams, grip and clogs are built. Loads and unloads through AFC are being tested now, and AFC's installer doesn't set one up yet.

| | |
|---|---|
| **Trays** | 4, on spring-loaded holders |
| **Bus** | Required: alone on its BridgeBox |
| **Toolhead sensor** | Optional: one after the splitter serves all four trays (on a U1, its own sensor). Without one, the odometer finds the gears |
| **Hub** | None needed; a switch at the splitter can act as one |
| **Buffer** | None needed; one can be added for ramming |
| **Tags** | Read when a spool goes in: material, color, temperatures and weight, with the tag UID and reel UUID that Spoolman links by |
| **Untagged spools** | Load and print the same; you set the lane |
| **Drying, humidity** | None: no heater, no humidity sensor |
| **Remaining filament** | Not measured: the AMS Lite can't, even on an A1. The lane keeps the weight Spoolman or you set |
| **Load** | About 30 mm/s until the sensor trips, or the odometer stops at the gears |
| **Unload** | Back past the splitter: sensor clear, then 30 mm (settable), or a set distance by odometer |
| **Path length** | Learned on the first load |
| **Overfeed stop** | 250 mm past the learned path |
| **Jam** | A feed that stops short waits for the AMS Lite's own stall report, then fails |
| **Grip check** | The odometer has to move once the extruder pulls |
| **Clog** | Extruder pushing, odometer still: AFC pauses the print |
| **Stuck spool** | Reported by the AMS Lite itself; the load stops and names the tray |

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/readme/ams-lite-dark.webp">
  <img alt="An AMS Lite&#x27;s four trays, not to scale. Four tubes run from the AMS Lite to a splitter at the toolhead, then one path through an optional toolhead sensor to the extruder gears. Tray 1 is loaded and stops at the sensor, or at the gears where its odometer stalls when there is no sensor; the other three are parked back in their own tubes, clear of the splitter. The path length from the AMS Lite is learned on the first load. An eject lets the tray go so the filament can be pulled out by hand." src="assets/readme/ams-lite-light.webp" width="100%">
</picture>

<sub>Not to scale. All four trays share the splitter and the toolhead sensor, if there is one. A load ends at the sensor, or without one where the odometer stalls at the extruder gears, and an unload pulls back just far enough to clear the splitter instead of reeling the tube home.</sub>

## What you'll need

### A Klipper printer running AFC

BridgeBox is a unit inside the Automated Filament Control (AFC) Klipper Add-On. If you already run AFC with a BoxTurtle or similar, an AMS becomes another thing it can drive.

### A 24 V 2–4 A supply

The AMS was powered by the printer; now it needs its own — the one thing not in the box. Any 24 V adapter from 2 to 4 A does it: the draw stays under 1 A even with an AMS 2 Pro drying on it, and a 2 A fuse on the board holds the ceiling whatever you plug in. The barrel matches an AMS 2 Pro's own adapter, so a spare one goes straight in. Buying rather than raiding a drawer? One AMS 2 Pro can dry off this supply; any other unit drying at the same time needs an adapter of its own.

### One dryer can run off the bridge

One AMS 2 Pro can dry on the bridge's own supply, over the same wire as everything else. A second one heating off that wire would pull the supply down and reset every unit on it, so AFC won't start it while the first is drying on bridge power. Plug an adapter into either one, even mid-cycle, and the block lifts. An AMS HT always needs its own mains cord to heat.

### USB or WiFi, with nothing to type

Plug BridgeBox in over USB and AFC finds it by itself, with no serial port to look up. For WiFi, join the hotspot BridgeBox puts up, pick your network on the page it shows, and Klipper gives it a private key, so there is no key to type. Start it on USB and it moves over to WiFi by itself once it has joined. To move it to another printer, AFC can clear its WiFi, or you can hold its button for ten seconds.

### One config section, then swap units freely

This is a maker's product, not an appliance, so there is Klipper config to set up. Only once, though: AFC's installer writes one BridgeBox section, and that section builds every unit, bay and sensor for you. After that the hardware is yours to rearrange. Plug an AMS in or pull one out while everything is running and it claims its own bays there and then — spare bays are held open for exactly that, so the ones you already had never shift and your Spoolman spools and tool numbers stay put.

### An AMS Lite needs a BridgeBox of its own

Nothing else can share its bus, so boxed units and HTs go on a second BridgeBox. A toolhead sensor is optional: a filament switch after the splitter its four tubes share ends each load (on a Snapmaker U1, the U1's own sensor), and without one the AMS Lite's odometer finds the extruder gears. The [AMS Lite section](#the-a1s-ams-lite-on-a-bridgebox-of-its-own) has the rest.

## Technical specifications

| | |
|---|---|
| **Works with** | Bambu Lab AMS 1 · AMS 2 Pro · AMS HT, all stock and unmodified. AMS Lite in testing |
| **How many** | 4 boxed units + 8 HT per BridgeBox, 24 bays (Bambu's limit, not ours), or one AMS Lite on its own. A printer can run more than one BridgeBox |
| **Connects at** | The filament buffer's 4-pin socket, the printer's own plug. On an AMS Lite, its cable in the A1's place |
| **Bus** | RS-485, half-duplex, 1,228,800 baud |
| **Link to the printer** | USB or WiFi, plain text. Found by itself on USB; WiFi set up from its own hotspot, with no key to type. The host never touches the bus |
| **Power** | 24 V, 2–4 A adapter (not included). 2 A fuse on the board. Same barrel as an AMS 2 Pro's. One AMS 2 Pro may dry on it; other dryers need their own supply |
| **Addressing** | Boxed units claim 0x00–0x03, HT units 0x80–0x87, at power-up. An AMS Lite has the bus to itself |
| **If the link drops** | Print pauses after 5 s of silence; a load in progress rides out 12 s |
| **Enclosure** | A printed part — the STLs come with it, so a replacement costs an evening |
| **Updates** | Over USB, or over the air on WiFi. Ships flashed, so an update never needs a button held |
| **Reversible** | Nothing on the AMS is modified. Unplug it and it goes back on a Bambu printer |

## Optional touchscreen display

A 5-inch touchscreen that shows what every bay is holding, what's loaded, and how a drying cycle is going — on the bench, without opening a laptop.

**This one is firmware, not hardware.** It is an image you flash onto a BTT Panda Touch or K-Touch — the same board under two names, sold as a Bambu accessory — which turns it into a display for your printer instead. If one is already sitting in a drawer since you stopped using it, that is the whole cost of this.

It follows every unit on the bridge, all twelve if you have them, one on the screen at a time. The Unit button at the top opens a list of them with each bay's color and material, so you can see the whole chain at a glance and tap the one you want.

**Not only the AMS.** Tick *All AFC units* in that list and every other unit AFC runs joins it: Box Turtle, Night Owl, Quattro Box, EMU, HTLF, ViViD, Claymore, ACE, ACE 2, OpenAMS, a toolchanger's tools. Each gets the same card, with its lanes, spools and weights, the box temperature and humidity when it reports them, and its buffer when it has one. A lane with an AFC RFID reader, on an ACE 2, an OpenAMS, a ViViD or a Box Turtle, gets Scan Tag too, which reads the tag where the spool sits and fills in the lane. A unit with more than four lanes pages through them four at a time.

Every bay tile shows the filament, how full the spool is, its print temperatures and Spoolman spool. Its T# / Runout button sets the tool number and the infinite-runout lane from any lane on the printer, and Scan Tag reads the spool's tag. It dims itself when it has been left alone, and a tap brings it straight back.

**Heater control on the units that have one.** An AMS 2 Pro, an AMS HT, an Anycubic ACE and an ACE 2 each get a Dryer Menu on their card: pick a temperature and a time, start or stop a cycle, and the card shows it drying and at what temperature. It sends the same commands as AFC's own dryer page and offers only what each unit accepts: up to 65 °C on an AMS 2 Pro, 85 °C on an HT, the ACE's configured ceiling, and spool rotation where the AMS supports it. Units without a heater show their temperature and humidity only.

It reaches the printer over your WiFi and never touches the filament bus. Unplug it in the middle of a print and the print doesn't notice — nothing on the shelf should be able to spoil a job.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/readme/panel-dark.webp">
  <img alt="Three screens from the BridgeBox touchscreen. First, the home screen: a status bar with the BridgeBox name, the bridge firmware version, Bridge Online, a Unit button reading Bambu AMS 1, a settings cog and Log Out, and below it one AMS 2 Pro card filling the screen, with its Dryer Menu, a Following status, a buffer gauge, humidity and chamber readouts, and four bay tiles showing tool number, filament, weight with a fill meter, temperatures, Spoolman spool, a T# / Runout button and a Scan Tag button. Second, the Unit list with the All AFC units box ticked: two Bambu units, then an ACE 2, an OpenAMS and a Box Turtle, each row with a chip per lane in its spool color. Third, the ACE 2 card: its own Dryer Menu, its buffer at 10 percent Advancing, humidity and box temperature, two empty lanes and two loaded ones, each loaded lane with a T# / Runout and a Scan Tag button." src="assets/readme/panel-light.webp" width="520">
</picture>

<sub>The panel's own screens, rendered from its firmware. One unit fills the screen at a time; the Unit button opens a list of every unit on the bridge, and with All AFC units ticked, every other AFC unit on the printer too, like the ACE 2 at the bottom, with its own Dryer Menu and Scan Tag.</sub>

---

**BridgeBox** — use your Bambu AMS with Klipper. Available soon.
