BridgeBox connects stock Bambu Lab AMS units to a Klipper printer running the AFC-Klipper-Add-On. It takes the place
of the Bambu printer on the AMS bus, so the AMS keeps its own board, firmware and cables, and AFC runs it like any
other unit. This documentation will guide you through wiring, installing, configuring and using a BridgeBox with
the AFC-Klipper-Add-On.

This documentation is intended for users who are already familiar with Klipper and the AFC-Klipper-Add-On. If you
are new to AFC, we recommend starting with the [AFC Project documentation](https://www.afcproject.dev/) and getting
comfortable with the basics before adding a BridgeBox.

!!! note

    Throughout this documentation, when we refer to Klipper, it also applies to forks such as Kalico, unless
    otherwise specified.

## Organization

The documentation is organized into several sections, each covering a specific part of running a BridgeBox.

- [**Overview**](bridgebox/index.md): What BridgeBox is, which AMS models it supports and how it fits into AFC.
- [**Hardware and Wiring**](bridgebox/hardware.md): Where BridgeBox plugs in, how it is powered and what the AMS
  units need.
- [**Installation**](bridgebox/installation.md): Adding a BridgeBox to AFC with the installer.
- [**USB and WiFi**](bridgebox/connection.md): How Klipper reaches the BridgeBox, and how to move it to WiFi.
- [**Configuration**](bridgebox/configuration.md): The `[AFC_BridgeBox]` section and its options.
- [**Bays, Lanes and Hot-Swap**](bridgebox/bays.md): How units claim their lanes and `T#`, and how to plug units in
  and out while Klipper runs.
- [**Spools, RFID and Spoolman**](bridgebox/spools.md): Bambu tags, spool measuring and Spoolman linking.
- [**Drying**](bridgebox/drying.md): Running the AMS 2 Pro and AMS HT dryers from Klipper.
- [**AMS Lite**](bridgebox/ams-lite.md): How the AMS Lite differs from the boxed units.
- [**Firmware Updates**](bridgebox/firmware.md): Updating the BridgeBox and the AMS units themselves.
- [**Touchscreen Display**](bridgebox/display.md): The optional BridgeBox display.
- [**Klipper Commands**](bridgebox/commands.md): Every BridgeBox and Bambu AMS command.
- [**Troubleshooting**](bridgebox/troubleshooting.md): Common issues and how to resolve them.

Throughout the documentation, you'll find code snippets, configuration examples and troubleshooting tips to help
you get the most out of your BridgeBox.

You may come across items in the documentation in the following tabbed format:

-----

=== "First Tab"
    This is the first tab of a tabbed section. It contains information about the first option.

=== "Second Tab"
    This is the second tab of a tabbed section. It contains information about the second option.

-----

Annotations are also used throughout the documentation to provide tips, warnings or additional information. They
can be expanded or collapsed by clicking on the title:

??? tip
    This is a tip annotation.

??? warning
    This is a warning annotation.

!!! note "Developed with the AFC team"

    BridgeBox was developed in conjunction with the AFC team to bring Bambu AMS support to Klipper printers. The
    AFC-Klipper-Add-On is open source; the BridgeBox transport component is proprietary and licensed exclusively to
    the AFC project.
