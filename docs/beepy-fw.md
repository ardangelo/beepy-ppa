---
title: Beepy Firmware
layout: default
---

# Beepy Firmware

- [User Guide](#user-guide)
  - [Flashing firmware directly](#flashing-firmware-directly)
  - [Firmware update utility](#firmware-update-utility)
  - [Working with the keyboard driver](#working-with-the-keyboard-driver)
- [Developer Reference](#developer-reference)
  - [Building from source](#building-from-source)
  - [Key values](#key-values)
  - [Power draw readings](#power-draw-readings)
  - [Register reference](#register-reference)

## User Guide

In the Beepy device, separate from the Raspberry Pi running Linux, there is an RP2040 microcontroller chip. This chip controls basic hardware input and output functions, including keyboard and touchpad input. The firmware discussed here runs directly on the RP2040. It cooperates with the Beepy keyboard driver [beepy-kbd](beepy-kbd.html) to provide key input and other functionality to Linux running on the Raspberry Pi.

See keyboard driver reference [beepy-kbd](beepy-kbd.html) for more information on keymaps.

### Flashing firmware directly

If you're setting up a new Beepy device, it's recommended to flash the firmware directly from the latest firmware release.

1. [Download the latest firmware image](https://github.com/ardangelo/beepberry-rp2040/releases/latest/download/i2c_puppet.uf2)

2. Turn the power switch off. With the device facing up, slide the power switch in the bottom-left hand corner to the left.

![Diagram showing direction of power switch](assets/beepy-switch-off.png)

3. Connect the Beepy to your computer via USB-C.

4. Locate the "End Call" key. It is the rightmost key on the top row of four function keys. 

5. While holding the "End Call" key, slide the power switch back on to enter firmware flash mode. In firmware flash mode, the LED will light up, and the Beepy will present itself as a USB mass storage device on your computer.

7. Copy the firmware image onto the presented drive just like a normal file. When copying is complete, Beepy will automatically flash and reboot with the new firmware.

If you're setting up a new device, you can proceed with the rest of the steps in the [quick start guide](../quick-start.html).

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

This section is intended for developers to reference for working directly with the firmware over I2C, at a lower level than the keyboard driver. For changing firmware settings from Linux through control files, see the keyboard driver reference [beepy-kbd](beepy-kbd.html).

This firmware is based off of the `i2c_puppet` firmware for keyboard interaction and communication. It adds several Beepy-specific features and improvements, including sticky modifier keys, real-time-clock support, self-update capability, deep sleep mode, and touchpad tuning.

Original readme for `i2c_puppet` including wiring and USB reference:

https://github.com/solderparty/i2c_puppet/blob/main/README.md

### Building from source

The code depends on the Raspberry Pi Pico SDK, which is added as a submodule. You can either perform a recursive submodule init, or rather follow these steps in the root of the repository:

    cd 3rdparty/pico-sdk
    git submodule update --init
    cd 3rdparty/pico-flashloader
    git submodule update --init
    cd 3rdparty/pico-extras
    git submodule update --init

Run `cmake` to build the firmware:

    mkdir build
    cd build
    cmake -DPICO_BOARD=beepy ..
    make

In the `build` directory, you will find the files `i2c_puppet.uf2` and `app/firmware.hex`. The primary firmware file is `i2c_puppet.uf2`, that can be [flashed directly over USB](#flashing-firmware-directly). `app/firmware.hex` is an Intel HEX encoded firmware file that can be applied on-device after converting line format to Unix and prepending a firmware header:

    cp app/firmware.hex beepy.hex
    dos2unix beepy.hex
    sed -i '1s;^;+Beepy dev build\n;' beepy.hex
    cat beepy.hex | sudo tee /sys/firmware/beepy/update_fw

See keyboard driver reference [beepy-kbd](beepy-kbd.html) for more information on using `/sys/firmware/beepy/update_fw`.

### Key values

Firmware has been updated to use BB10-style sticky modifier keys. It has a corresponding kernel module that has been updated to read modifier fields over I2C.

Holding a modifier key (shift, physical alt, Symbol) while typing an alpha keys will apply the modifier to all alpha keys until the modifier is released.

One press and release of the modifier will enter sticky mode, applying the modifier to
the next alpha key only. If the same modifier key is pressed and released again in sticky mode, it will be canceled.

Call is mapped to Control. The Berry button is mapped to `KEY_PROPS`. Clicking the touchpad button is mapped to `KEY_COMPOSE`. Back is mapped to Escape. End Call is not sent as a key, but holding it will still trigger the power-off routine. Symbol is mapped to AltGr (Right Alt).

Physical alt does not send an actual Alt key, but remaps the output scancodes to the range 135 to 161 in QWERTY order. This should be combined with a keymap for proper symbol output. This allows symbols to be customized without rebuilding the firmware, as well as proper use of the actual Alt key.

See keyboard driver reference [beepy-kbd](beepy-kbd.html) for more information on keymaps.

### Power draw readings

Approximate power draw readings, obtained using a USB-C power meter, in amps.

```
.000 Power switch off

.50 Pi booting
.25 Pi idle, wireless connected, keyboard backlight full
.20 Pi idle, wireless connected, keyboard backlight off
.09 Pi shut down from command line (Pi pin still powered)
.025 Pi pin unpowered, no deep sleep
.005 Pi pin unpowered, deep sleep

.027 Backlight brightness off
.030 Backlight brightness dim
.053 Backlight brightness half
.079 Backlight brightness full

.030 Touchpad disabled
.031 Touchpad enabled
```

### Register reference

The device uses I2C slave interface to communicate, the address can be configured in `app/config/conf_app.h`, the default is `0x1F`.

You can read the values of all the registers, the number of returned bytes depends on the register. It's also possible to write to the registers, to do that, apply the write mask `0x80` to the register ID (for example, the backlight register `0x05` becomes `0x85`).

Some registers are read-only or write-only. For read-only registers, writes are discarded. For write-only registers, arbitrary byte is returned.

#### `0x01` `REG_ID_VER`

Read-only, 1 byte.

The first nibble contains the major version and the second nibble contains the minor version of the firmware.

#### `0x02` `REG_ID_CFG`

Read-write, 1 byte.

Bitmap of various settings that can be changed to customize the behavior of the firmware.

See `REG_CF2` for additional settings.

* Bit `7` `CFG_USE_MODS`: deprecated, unused
* Bit `6` `CFG_REPORT_MODS`: deprecated, unused
* Bit `5` `CFG_PANIC_INT`: unused
* Bit `4` `CFG_KEY_INT`: Interrupt when a key is pressed
* Bit `3` `CFG_NUMLOCK_INT`: Interrupt when numlock is pressed
* Bit `2` `CFG_CAPSLOCK_INT`: Interrupt when capslock is pressed
* Bit `1` `CFG_OVERFLOW_INT`: Innterrupt when event queue overflows
* Bit `0` `CFG_OVERFLOW_ON`: Overwrite oldest event when overflow occurs

Defaut value: `CFG_OVERFLOW_INT | CFG_KEY_INT`

#### `0x03` `REG_ID_INT`

Read-write, 1 byte.

On interrupt, this contains the cause.

* Bit `7` Unused
* Bit `6` `INT_TOUCH` Generated by trackpad motion
* Bit `5` `INT_GPIO` Generated by input GPIO changing level
* Bit `4` `INT_PANIC` Unused
* Bit `3` `INT_KEY` Generated by a key press
* Bit `2` `INT_NUMLOCK` Generated by Num Lock
* Bit `1` `INT_CAPSLOCK` Generated by Caps Lock
* Bit `0` `INT_OVERFLOW` Generated by FIFO overflow

After reading this register, write `0x00` to reset it.

For `INT_GPIO`, check [`REG_GIN`](#0x10-reg_id_gin) to see which GPIO triggered the interrupt. The GPIO interrupt must first be enabled in [`REG_GIC`](#0x0f-reg_id_gic).

#### `0x04` `REG_ID_KEY`

Read-only, 1 byte.

Contains FIFO status and modifier key status.

* Bit `7` Unused
* Bit `6` `KEY_NUMLOCK` Num Lock enabled?
* Bit `5` `KEY_CAPSLOCK` Caps Lock enabled?
* Bits `0-4` `KEY_COUNT` Unread FIFO event count

#### `0x05` `REG_ID_BKL`

Read-write, 1 byte.

Set keyboard backlight brightness from `0x00` for off to `0xff` for full brightness. Full brightness draws approximately 25% more power on an idle Raspberry Pi (see [Power draw readings](#power-draw-readings)), so a lower setting is recommended.

#### `0x06` `REG_ID_DEB`

Unimplemented, 1 byte.

Keyboard debounce setting.

#### `0x07` `REG_ID_FRQ`

Unimplemented, 1 byte.

Keyboard poll frequency.

#### `0x08` `REG_ID_RST`

Read-write, 1 byte.

Access will cause RP2040 to reset.

#### `0x09` `REG_ID_FIF`

Read-only, 2 bytes.

Return topmost event in FIFO. First byte contains key scancode. Second byte contains key state:

* `0` `KEY_STATE_IDLE`
* `1` `KEY_STATE_PRESSED`
* `2` `KEY_STATE_HOLD`
* `3` `KEY_STATE_RELEASED`
* `4` `KEY_STATE_LONG_HOLD`

#### `0x0A` `REG_ID_BK2`

Unimplemented, 1 byte.

Secondary backlight control.

#### `0x0B` `REG_ID_DIR`

Read-write, 1 byte.

Cnotrols direction of the GPIO expander pins, each bit corresponding to one pin.

The assignment of pin to MCU depends on the board, see `boards/beepy.h` for the assignments.

Bit set to `1` configures pin as input, bit set to `0` configures pin as output.

Default value: `0xFF` (all pins configured as input)

#### `0x0C` `REG_ID_PUE`

Read-write, 1 byte.

If a GPIO pin is configured as an input using [`REG_DIR`](#0x0b-reg_id_dir), an optional pull-up/pull-down can be enabled. If pin is configured as output, its bit in this register has no effect.

The assignment of pin to MCU depends on the board, see `boards/beepy.h` for the assignments.

Bit set to `1` enables input pull for that pin, bit set to `0` disables input pull.

The direction of the pull is set in [`REG_PUD`](#0x0d-reg_id_pud).

Default value: 0 (all pulls disabled)

#### `0x0D` `REG_ID_PUD`

Read-write, 1 byte.

If a GPIO pin is configured as an input using [`REG_DIR`](#0x0b-reg_id_dir), an optional pull-up/pull-down can be enabled. If pin is configured as output, its bit in this register has no effect.

The assignment of pin to MCU depends on the board, see `boards/beepy.h` for the assignments.

Bit set to `1` sets input pull to pull-up for that pin, bit set to `0` sets input pull to pull-down.

Default value: `0xFF` (all pulls set to pull-up, if enabled in `REG_PUE` and set to input in `REG_DIR`)

#### `0x0E` `REG_ID_GIO`

Read-write, 1 byte.

Contains the values of the GPIO Expander pins, each bit corresponding to one pin.

The assignment of pin to MCU depends on the board, see `boards/beepy.h` for the assignments.

If a pin is configured as an output in [`REG_DIR`](#0x0b-reg_id_dir), writing to this register will change the value of that pin.

Reading from this register returns the value for both input and output pins.

#### `0x0F` `REG_ID_GIC`

Read-write, 1 byte.

If a GPIO pin is configured as an input using [`REG_DIR`](#0x0b-reg_id_dir), an optional interrupt on value change can be enabled. If pin is configured as output, its bit in this register has no effect.

The assignment of pin to MCU depends on the board, see `boards/beepy.h` for the assignments.

Bit set to `1` triggers interrupt on value change, bit set to `0` disables interrupt.

On interrupt, GPIO that triggered the interrupt can be determined by reading [`REG_GIN`](#0x10-reg_id_gin). Additionally, the `INT_GPIO` bit will be set in [`REG_INT`](#0x03-reg_id_int).

Default value: `0x00`

#### `0x10` `REG_ID_GIN`

Read-only, 1 byte.

On interrupt, this register contains which GPIO pin caused the interrupt, each bit corresponding to one pin.

The assignment of pin to MCU depends on the board, see `boards/beepy.h` for the assignments.

After reading, reset it to `0x00`.

#### `0x11` `REG_ID_HLD`

Read-write, 1 byte.

Sets time threshold for "press and hold" key state, in units of 10ms.

When a key is held for longer than this time, a [key hold event](#0x09-reg_id_fif) is generated and enqueued into the FIFO event queue.

Default value: 30 (300ms)

#### `0x12` `REG_ID_ADR`

Read-write, 1 byte.

Device's primary I2C bus address. On write, applies address change immediately. The next communication must be performed on the new address. Not saved after a reset.

Default value: `0x1F`

#### `0x13` `REG_ID_IND`

Read-write, 1 byte.

Sets time for which the INT/IRQ pin is held LOW on interrupt, in units of 1ms.

Default value: 1 (1ms)

#### `0x14` `REG_ID_CF2`

Read-write, 1 byte.

Bitmap of various settings that can be changed to customize the behavior of the firmware.

See [`REG_CFG`](#0x02-reg_id_cfg) for additional settings.

* `7` Unused
* `6` Unused
* `5` Unused
* `4` Unused
* `3` `CF2_AUTO_OFF` When [driver state unloaded](#0x2d-reg_id_driver_state) set to unloaded, wait for `REG_ID_SHUTDOWN_GRACE` seconds, then enter deep sleep
* `2` `CF2_USB_MOUSE_ON` Send trackpad events over USB
* `1` `CF2_USB_KEYB_ON` Send keyboard events over USB
* `0` `CF2_TOUCH_INT` Generate interrupt for trackpad event

Default value: `CF2_TOUCH_INT | CF2_AUTO_OFF`

#### `0x15` `REG_ID_TOX`

Read-only, 1 byte.

Trackpad X-axis position delta since the last time this register was read. Signed, in range[-128, 127]. Resets to 0 on read.

Recommended to read value when touch event received, or overflow may occur.

#### `0x16` `REG_ID_TOY`

Read-only, 1 byte.

Trackpad Y-axis position delta since the last time this register was read. Signed, in range[-128, 127]. Resets to 0 on read.

Recommended to read value when touch event received, or overflow may occur.

#### `0x17` `REG_ID_ADC`

Read-only, 2 bytes.

Raw battery level from ADC. 16-bit result:

    (read(REG_ID_ADC)[1] << 8) | read(REG_ID_ADC)[0]

#### `0x20` `REG_ID_LED`

Read-write, 1 byte.

Write the LED color registers before this register.

* `0x00` LED off
* `0x01` LED on
* `0x02` LED flashes
* `0x03` LED flashes until key is pressed

Mode 3, flash until key pressed, will overlay on top of an existing LED setting. For example, the following write sequence will set the LED to be solid red, but with a blue flash. Then, when a key is pressed, it will return to a solid red.

    Set LED to solid red
    REG_ID_LED_R <- 0xff
    REG_ID_LED_G <- 0x00
    REG_ID_LED_B <- 0x00
    REG_ID_LED   <- 0x01

    Set LED to flash blue until key pressed
    REG_ID_LED_R <- 0x00
    REG_ID_LED_G <- 0x00
    REG_ID_LED_B <- 0xff
    REG_ID_LED   <- 0x03

#### `0x21` `REG_ID_LED_R`

Read-write, 1 byte.

Set LED red values unsigned in range [0, 255].

Color settings are applied after [`REG_LED`](#0x20-reg_id_led) is written.

#### `0x22` `REG_ID_LED_G`

Read-write, 1 byte.

Set LED green values unsigned in range [0, 255].

Color settings are applied after [`REG_LED`](#0x20-reg_id_led) is written.

#### `0x23` `REG_ID_LED_B`

Read-write, 1 byte.

Set LED blue values unsigned in range [0, 255].

Color settings are applied after [`REG_LED`](#0x20-reg_id_led) is written.

#### `0x24` `REG_ID_REWAKE_MINS`

Read-write, 1 byte.

Write to shut down the Pi, then power-on in that many minutes. Useful for polling services in conjunction with [`REG_ID_STARTUP_REASON`](#0x2e-reg_id_startup_reason), such as with the [beepy-poll](beepy-poll.html) service.

#### `0x25` `REG_ID_SHUTDOWN_GRACE`

Read-write, 1 byte.

Due to the Beepy hardware design, there is no way to reliably determine the power state of the Pi. To avoid powering off the Pi while it is still running, this register is set to the number of seconds to wait between a shut down signal and Pi power off. This helps ensure that the Pi has time to process the power-off command and to shut down cleanly.

Used for [shutdown followed by rewake](#0x24-reg_id_rewake_mins) and [entering deep sleep mode](#0x2d-reg_id_driver_state).

Default: 30 (30s)

#### `0x26` `REG_ID_RTC_SEC`

Read-write, 1 byte.

Read to get the seconds value from the real-time clock. Write is committed after a write to [`REG_ID_RTC_COMMIT`](#0x2c-reg_id_rtc_commit).

#### `0x27` `REG_ID_RTC_MIN`

Read-write, 1 byte.

Read to get the minutes value from the real-time clock. Write is committed after a write to [`REG_ID_RTC_COMMIT`](#0x2c-reg_id_rtc_commit).

#### `0x28` `REG_ID_RTC_HOUR`

Read-write, 1 byte.

Read to get the hour value from the real-time clock. Write is committed after a write to [`REG_ID_RTC_COMMIT`](#0x2c-reg_id_rtc_commit).

#### `0x29` `REG_ID_RTC_MDAY`

Read-write, 1 byte.

Read to get the day-of-month value from the real-time clock. Write is committed after a write to [`REG_ID_RTC_COMMIT`](#0x2c-reg_id_rtc_commit).

#### `0x2A` `REG_ID_RTC_MON`

Read-write, 1 byte.

Read to get the month value from the real-time clock. Write is committed after a write to [`REG_ID_RTC_COMMIT`](#0x2c-reg_id_rtc_commit).

#### `0x2B` `REG_ID_RTC_YEAR`

Read-write, 1 byte.

Year value is expressed in years since 1900.

Read to get the year value from the real-time clock. Write is committed after a write to [`REG_ID_RTC_COMMIT`](#0x2c-reg_id_rtc_commit).

#### `0x2C` `REG_ID_RTC_COMMIT`

Write-only, 1 byte.

Write `1` to commit the values written to RTC registers to the real-time clock. Due to the Beepy hardware design, RTC settings are lost on power off of the RP2040 via power switch, or when entering deep sleep.

The keyboard driver [beepy-kbd](beepy-kbd.html) will update the RP2040 RTC with network time settings when available.

#### `0x2D` `REG_ID_DRIVER_STATE`

Read-write, 1 byte.

Write `1` to indicate that the keyboard driver is loaded and that shutdown commands will be read and processed. The keyboard driver implementation will set this value to `1` when the driver is loaded and `0` when unloaded.

In most cases, the driver is loaded on boot and unloaded during shutdown. For substantial power savings, the default-enabled [`CF2_AUTO_OFF`](#0x14-reg_id_cf2) setting will trigger when `0` is written to this register. After a 30 second wait to allow for the driver to potentially be reloaded, the RP2040 will send a shutdown signal to the Pi, wait for [`REG_ID_SHUTDOWN_GRACE`](#0x25-reg_id_shutdown_grace) seconds, then power off the Pi and enter deep sleep.

Default: `0`

#### `0x2E` `REG_ID_STARTUP_REASON`

Read-only, 1 byte.

Contains the reason why the Pi was booted. Useful for polling services in conjunction with [`REG_ID_REWAKE_MINS`](#0x24-reg_id_rewake_mins).

* `0` RP2040 initialized and booted Pi
* `1` Power button held to turn Pi back on
* `2` Rewake triggered from [`REG_ID_REWAKE_MINS`](#0x24-reg_id_rewake_mins)
* `3` During rewake polling, `0` was written to [`REG_ID_REWAKE_MINS`](#0x24-reg_id_rewake_mins). This allows the [beepy-poll](beepy-poll.html) service to cancel the poll and proceeded with a full boot

#### `0x30` `REG_ID_UPDATE_DATA`

Read-write, 1 byte.

RP2040 firmware is loaded in two stages. The first stage is a modified version of [pico-flashloader](https://github.com/rhulme/pico-flashloader). It allows updates to be flashed to the second stage firmware while booted. The second stage is the actual Beepy firmware.

Reading `REG_ID_UPDATE_DATA` will return an update status code

- `0` `UPDATE_OFF` Update not in progress
- `1` `UPDATE_RECV` In the process of receiving an update
- `2` `UPDATE_FAILED` General update failure
- `3` `UPDATE_FAILED_LINE_OVERFLOW` Firmware line overflowed buffer
- `4` `UPDATE_FAILED_FLASH_EMPTY` Firmware flash request was empty
- `5` `UPDATE_FAILED_FLASH_OVERFLOW` Firmware overflows allowed update region
- `6` `UPDATE_FAILED_BAD_LINE` Failed to parse line in Intel HEX format
- `7` `UPDATE_FAILED_BAD_CHECKSUM` Failed checksum

Firmware updates are flashed by writing byte-by-byte to `REG_UPDATE_DATA`:

- Header line beginning with `+` e.g. `+Beepy`
- Followed by the contents of an image in Intel HEX format

By default, `REG_UPDATE_DATA` will be set to `UPDATE_OFF`.
After writing, `REG_UPDATE_DATA` will be set to `UPDATE_RECV` if more data is expected.

If the update completes successfully:

- `REG_UPDATE_DATA` will be set to `UPDATE_OFF`
- Shutdown signal will be sent to the Pi
- Delay to allow the Pi to cleanly shut down before poweroff (configurable with [`REG_SHUTDOWN_GRACE`](#0x25-reg_id_shutdown_grace))
- Firmware is flashed and the system is reset

Please wait until the system reboots on its own before removing power.

If the update failed, `REG_UPDATE_DATA` will contain an error code and the firmware will not be modified.

The header line `+...` will reset the update process, so an interrupted or failed update can be retried by restarting the firmware write.

#### `0x40` `REG_ID_TOUCHPAD_REG`

Read-write, 1 byte.

To send or recieve data from the touchpad firmware, write the desired touchpad register number. Touchpad registers can be found in the [ADBS A320 datasheet](https://www.mouser.com/datasheet/2/38/V02-1859EN+DS+ADBS-A320+16Nov2011-20613.pdf). Then, read or write [`REG_ID_TOUCHPAD_VAL`](#0x41-reg_id_touchpad_val).

#### `0x41` `REG_ID_TOUCHPAD_VAL`

Read-write, 1 byte.

To send or recieve data from the touchpad firmware, write the desired touchpad register number to [`REG_ID_TOUCHPAD_REG`](#0x40-reg_id_touchpad_reg). Then, read or write this register.

#### `0x42` `REG_ID_TOUCHPAD_MIN_SQUAL`

Read-write, 1 byte.

Reject touchpad input if surface quality as reported by touchpad sensor is lower than this threshold.

Default: `16`

#### `0x43` `REG_ID_TOUCHPAD_LED`

Read-write, 1 byte.

Touchpad LED power setting. "High" is recommended for reliable input.

* `0x0` power medium
* `0x3` power high
* `0x5` power low

Default: `0x3` power high
