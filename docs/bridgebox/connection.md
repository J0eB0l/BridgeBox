# USB and WiFi

Klipper talks to the BridgeBox over USB or WiFi. Either way, Klipper makes the link key and stores it on the
BridgeBox by itself, so there is never a key to type.

## USB

```ini
[AFC_BridgeBox chain1]
serial_port: auto
```

With `serial_port: auto`, AFC finds the BridgeBox on USB and saves which board it took, so the chain keeps that
board across restarts. A second chain on `auto` takes a BridgeBox no other chain uses. If no BridgeBox is plugged in
at startup, the chain keeps looking once Klipper is up and picks up a board plugged in later.

??? tip "Naming the port yourself"

    You can name the port instead of using `auto`. Find it with:

    ```bash
    ls /dev/serial/by-id/
    ```

    and set it as the `serial_port`:

    ```ini
    serial_port: /dev/serial/by-id/usb-Espressif_..._if00
    ```

### Replacing a board

The chain saves the board it found. To move the chain to a replacement BridgeBox, run `AFC_BRIDGEBOX_FORGET_BOARD`,
then `RESTART`. The chain then takes the next free BridgeBox it finds on USB.

## WiFi

=== "USB, then WiFi"

    This is the easiest way onto WiFi, and the BridgeBox falls back to USB whenever WiFi is down while the cable is in.

    1. Start with the BridgeBox on USB and add `connection: wifi` to its section:

        ```ini
        [AFC_BridgeBox chain1]
        serial_port: auto
        connection: wifi
        ```

    2. Run `FIRMWARE_RESTART`.
    3. On a phone, join the BridgeBox's hotspot, named `bb-bridge-XXXX` after the board. The setup page opens by
       itself; if it doesn't, browse to `http://10.42.42.1`.
    4. Choose your network and enter its password. The page shows **Connected** with the address the BridgeBox got.

    Klipper sees the BridgeBox join, stores the link key on it and moves the link to WiFi by itself. The cable can
    stay in as a fallback or come out.

=== "WiFi only"

    Use this when the BridgeBox will never be on a USB cable.

    1. Power the BridgeBox. On a phone, join its `bb-bridge-XXXX` hotspot; the setup page opens by itself, or browse
       to `http://10.42.42.1`.
    2. Choose your network and enter its password. Note the address the page shows.
    3. Put that address in the section in place of `serial_port: auto`:

        ```ini
        [AFC_BridgeBox chain1]
        serial_port: tcp://192.168.1.50:8888
        ```

    4. Restart Klipper. It stores the link key on the BridgeBox at the first connect.

    !!! tip

        Give the BridgeBox a fixed DHCP lease in your router so its address doesn't change.

=== "Network in the config"

    If the hotspot page doesn't work for you, put the network in the config and Klipper gives it to the BridgeBox
    over USB once per start:

    ```ini
    [AFC_BridgeBox chain1]
    serial_port: auto
    connection: wifi
    wifi_ssid: YOUR_NETWORK
    wifi_password: YOUR_PASSWORD
    ```

    !!! warning

        The password is stored in plain text, so keep this file private. Double quotes and backslashes can't be used,
        and the password must be 8 to 63 characters, or empty for an open network.

## Moving to another printer or network

A BridgeBox remembers its network and the link key of the printer that set it up. Before moving it:

1. On the old printer, run `AFC_BRIDGEBOX_RESET_WIFI`. It wipes the BridgeBox's network and key and brings its
   hotspot back. You can also hold the BOOT button for 10 seconds, which does the same without Klipper.
2. Stop the old printer's chain from reaching for it: set `connection: usb` or remove the section. A chain left on
   `connection: wifi` keeps reconnecting to the board's old address and takes it back from the new printer.
3. Set it up on the new printer as above.

`AFC_BRIDGEBOX_RESET_WIFI` is refused during a print and while the BridgeBox is not connected.

## If the link drops

If Klipper loses the BridgeBox during a print, AFC pauses the print after 5 seconds of silence. A load in progress
rides out 12 seconds before it fails. When the link comes back, the units come back online by themselves.

[Next step: Configuration](configuration.md){ .md-button }
