# Installation

BridgeBox is installed with the AFC-Klipper-Add-On installer, the same way as any other AFC unit. The installer
writes a single `[AFC_BridgeBox]` section, and that section builds every unit, lane, hub and sensor for the AMS units
on the bus.

!!! note

    All examples in this section assume that you are running the commands on the SBC that runs Klipper and the
    AFC-Klipper-Add-On.

!!! warning "Testing branch"

    Until BridgeBox is part of an AFC release, it lives on the `BridgeBox` branch of
    [J0eB0l/AFC-Klipper-Add-On](https://github.com/J0eB0l/AFC-Klipper-Add-On/tree/BridgeBox). Use that branch for the
    steps below.

## Prerequisites

- Everything on the AFC [installation prerequisites](https://www.afcproject.dev/installation/getting-started.html)
  list: Klipper, Moonraker, Mainsail or Fluidd, and Python 3.8 or newer.
- A BridgeBox wired to your AMS units and powered (see [Hardware and Wiring](hardware.md)).
- An `[AFC_extruder]` section for the extruder the AMS will feed, with a toolhead sensor.

## Step 1: Get the AFC-Klipper-Add-On

=== "Command Line"
    ```bash
    cd ~
    git clone -b BridgeBox https://github.com/J0eB0l/AFC-Klipper-Add-On.git
    cd AFC-Klipper-Add-On
    ```
=== "Explanation"

    This clones the AFC-Klipper-Add-On with BridgeBox support into your home directory and changes into it. If you
    already have AFC installed from another copy, see [Updating an existing install](#updating-an-existing-install).

## Step 2: Run the installer

From within the `AFC-Klipper-Add-On` directory, start the installer on the BridgeBox branch:

```bash
./install-afc.sh -b BridgeBox
```

!!! warning "Keep `-b BridgeBox`"

    Without it, the installer switches the copy to `main`, which has no BridgeBox support. Use it every time you run
    the installer or `./update-afc.sh`.

=== "First unit"

    From the main menu choose **Install**, then pick **BridgeBox** as the installation type with `T`.

=== "Additional unit"

    If AFC is already installed with another unit, such as a BoxTurtle, choose the additional unit menu with `A` and
    pick **BridgeBox**. The installer asks for a chain name; it offers the first free one (`chain1`, `chain2` and so
    on).

## Step 3: BridgeBox options

The installer shows the BridgeBox options below. Each one can be changed later in `AFC/AFC_BridgeBox.cfg`.

#### `P. Units on this chain`

=== "Description"
    How many AMS or AMS 2 Pro units (0 to 4) and how many AMS HT units (0 to 8) this BridgeBox will run. Count the most
    units you will ever run on it: raising the AMS count later moves every HT lane up. The defaults, 4 and 8, are the
    most a Bambu bus takes.
=== "Configuration"
    Sets `pool_ams` and `pool_ht`. See [Bays, Lanes and Hot-Swap](bays.md).

#### `W. Connection`

=== "Description"
    How Klipper reaches the BridgeBox:

    1. **USB.**
    2. **USB, then WiFi:** once the BridgeBox joins your WiFi from its `bb-bridge` hotspot, Klipper moves over by
       itself.
    3. **WiFi only:** join the `bb-bridge` hotspot from a phone, choose your network and enter the address the page
       shows.
=== "Configuration"
    Sets `connection` and `serial_port`. See [USB and WiFi](connection.md).

#### `S. USB serial port`

=== "Description"
    `A` for **auto**, which lets AFC find the BridgeBox on USB by itself. You can also pick one of the listed ports by
    number or type a `/dev/serial/by-id/` path. Auto is recommended.
=== "Configuration"
    Sets `serial_port`. A WiFi only BridgeBox has no USB serial port.

#### `X. Extruder`

=== "Description"
    The `[AFC_extruder]` this BridgeBox feeds, for example `extruder` or `extruder1`. One BridgeBox feeds one extruder.
=== "Configuration"
    Sets `extruder`. The installer reminds you to check that your AFC config has that section.

!!! note "Buffer"

    The installer does not ask about a buffer for a BridgeBox. Every AMS has its own buffer, and the BridgeBox chain
    reads it for you.

## Step 4: Restart and check

Restart Klipper, then run:

```
AFC_BAMBU_UIDS
```

This lists every unit on the bus and the BridgeBox firmware version. Each unit claims a bay once AFC's PREP has run,
and its lanes appear in Mainsail or Fluidd. `AFC_BRIDGEBOX_BAYS` shows which bay each unit took.

!!! tip

    The first load on each unit measures its own filament path. There is no bowden length to set for a BridgeBox.

## Updating an existing install

If you already run AFC from the official repository and want to add BridgeBox, switch that copy to the BridgeBox
branch and run the installer again:

```bash
cd ~/AFC-Klipper-Add-On
git remote add bridgebox https://github.com/J0eB0l/AFC-Klipper-Add-On.git
git fetch bridgebox BridgeBox
git checkout -b BridgeBox bridgebox/BridgeBox
./install-afc.sh -b BridgeBox
```

Your existing units and their config files are kept. Use the additional unit menu (`A`) to add the BridgeBox.

## Manual installation

If you would rather not use the installer, copy `templates/AFC_BridgeBox.cfg` from the AFC-Klipper-Add-On folder to
`AFC/AFC_BridgeBox.cfg` in your Klipper config folder, where AFC's `[include AFC/*.cfg]` loads it. Set `extruder`,
then restart. The template is commented throughout; [Configuration](configuration.md) covers every option.

!!! warning

    Do not write `[AFC_BambuAMS]`, `[AFC_lane]`, `[AFC_hub]` or `[AFC_buffer]` sections for Bambu units yourself. The
    `[AFC_BridgeBox]` section builds them, and a hand-written section on the same lane numbers or names can stop
    Klipper from starting.

[Next step: USB and WiFi](connection.md){ .md-button }
