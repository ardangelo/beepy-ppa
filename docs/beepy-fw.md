---
title: Beepy Firmware
layout: default
---

# Beepy Firmware

## User Guide

In the Beepy device, separate from the Raspberry Pi running Linux, there is an RP2040 microcontroller chip. This chip controls basic hardware input and output functions, including keyboard and touchpad input. The firmware discussed here runs directly on the RP2040. It cooperates with the Beepy keyboard driver [beepy-kbd](docs/beepy-kbd.html) to provide key input and other functionality to Linux running on the Raspberry Pi.

### Flashing firmware directly

If you're setting up a new Beepy device, it's recommended to flash the firmware directly from the latest firmware release.

1. [Download the latest firmware image](https://github.com/ardangelo/beepberry-rp2040/releases/latest/download/i2c_puppet.uf2)

2. Turn the power switch off. With the device facing up, slide the power switch in the bottom-left hand corner to the left.

![Diagram showing direction of power switch](assets/beepy-switch-off.png)

3. Connect the Beepy to your computer via USB-C.

4. Locate the "End Call" key. It is the rightmost key on the top row of four function keys. 

5. While holding the "End Call" key, slide the power switch back on to enter firmware flash mode. In firmware flash mode, the LED will light up, and the Beepy will present itself as a USB mass storage device on your computer.

7. Copy the firmware image onto the presented drive just like a normal file. When copying is complete, Beepy will automatically flash and reboot with the new firmware.

If you're setting up a new device, you can proceed with the rest of the steps in the [quick start guide](quick-start.html).

### Firmware update utility

Once you have an up-to-date firmware image installed, and your Linux system configured, you can update firmware directly on the device using firmware distribution packages. When a new firmware image is released, there will be an update available for this package, `beepy-fw`. Running `sudo apt-get upgrade` will upgrade this firmware package. Due to the way that package updates work, firmware updates must be applied using an updater utility.

The package comes with two primary files, a copy of the new firmware, and the firmware updater utility `update-beepy-fw`. Copies of new firmware are installed into the directory `/usr/lib/beepy-firmware`. After updating the firmware package, apply the firmware update by running `sudo beepy-firmware`.

For example, if you have firmware 3.0 installed, and update the `beepy-fw` package to version 3.1, running the firmware updater utility will look like this:

    $ sudo update-beepy-fw

    Beepy firmware updater
    Installed firmware: 3.0
    Newer firmware in /usr/lib/beepy-firmware:
    [ 0] 3.1: beepy_3.1.hex
    Enter number of newer firmware to install: 

In this case, you would want to type `0`, then press `Enter` to install version 3.1:

    Enter number of newer firmware to install: 0
    Installing /usr/lib/beepy-firmware/beepy_3.1.hex...
    Update applied. Please wait until system powers back on in 30 seconds

After a successful firmware install, the system will shut down and apply the firmware update. Please wait until the update is installed and for the system to reboot automatically. There will be a delay of 30 seconds between the shutdown of the Raspberry Pi and reboot to apply the firmware.

If the firmware cannot be written, the update will fail and new firmware will not be applied. But if you do get into a state with a broken firmware, you can always follow the steps listed above in the section [Flashing firmware directly](#flashing-firmware-directly) to reset or update the firmware.

If your firmware version is up to date, running `beepy-firmware` will report that there are no newer versions available to install:

    Beepy firmware updater
    Installed firmware: 3.1
    Newer firmware in /usr/lib/beepy-firmware:
        (None found)

### Working with the keyboard driver

When new functionality is added to the Beepy keyboard driver, there may be a corresponding update to the firmware as well. However, the keyboard driver will automatically check the installed firmware version. If there is a hard incompatibility, the installation of the keyboard driver will fail:

    error: driver requires firmware version 3.1
        (firmware has version 3.0)
    update the beepy-fw package and run
        /sbin/update-beepy-fw
    or manually install the firmware release from
        https://github.com/ardangelo/beepberry-rp2040/releases/latest

In this case, update the `beepy-fw` driver and run `sudo update-beepy-fw` (see [Firmware update utility](#firmware-update-utility)), or manually flash the updated firmware release (see [Flashing firmware directly](#flashing-firmware-directly)).

## Developer Reference

Original readme:

https://github.com/solderparty/i2c_puppet/blob/main/README.md

### Building from source

### Register reference
