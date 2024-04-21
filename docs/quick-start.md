---
title: Beepy Quick Start
layout: default
---

# Beepy package repository

{:toc}

## Updating firmware from 2.X

* [Download the latest firmware image](https://github.com/ardangelo/beepberry-rp2040/releases/latest/download/i2c_puppet.uf2)
* Slide the power switch off (left if facing up)
* Connect the Beepy to your computer via USB-C
* While holding the "End Call" key (top right on the keypad), slide the power switch on
* The Beepy will present itself as a USB mass storage device. Copy the firmware image into the drive and it will reboot with the new firmware

## Updating firmware from 3.0 on-device

Starting with firmware version 3.0 and `beepy-kbd` 2.4, the package `beepy-fw` can be used to update firmware directly from Beepy.
If you have a firmware version older than 3.0 installed, please update manually by flashing the `uf2` image over USB.

`beepy-fw` provides a script `/sbin/update-beepy-fw` and a copy of the firmware.
After installing `beepy-fw`, you can install a newer firmware by running the interactive script

	/sbin/update-beepy-fw

## Adding to APT and installing drivers

SSH into the Pi and install driver packages. The LED will remain green until drivers are installed and the system has rebooted.

	curl -s --compressed "https://ardangelo.github.io/beepy-ppa/KEY.gpg" | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/beepy.gpg >/dev/null
	sudo curl -s --compressed -o /etc/apt/sources.list.d/beepy.list "https://ardangelo.github.io/beepy-ppa/beepy.list"
	sudo apt update
	sudo apt-get -y install beepy-fw sharp-drm beepy-symbol-overlay beepy-kbd tmux beepy-tmux-menus beepy-gomuks beepy-poll
	sudo reboot

The keyboard driver package will run a preinstall check to ensure that the Beepy firmware is compatible with the driver.

If the installed firmware is detected as incompatible, the installation will be canceled.

A link to a compatible firmware release will be output as part of the error message.

## Package listing

### `beepy-fw`

`beepy-fw` provides a script `update-beepy-fw` and a copy of the firmware.
After installing `beepy-fw`, you can install a newer firmware by running as root

	/sbin/update-beepy-fw

Newer firmware versions will be listed along with a prompt to select the version to install.

	Installed firmware: 3.0
	Newer firmware in /usr/lib/beepy-firmware:
	[ 0] 3.1: beepy_3.1.hex
	Enter number of newer firmware to install: _

If the update completes successfully, the system will be rebooted.
There is a 30 second delay (configurable at `/sys/module/beepy_kbd/parameters/shutdown_grace` to allow the operating system to cleanly shut down before the Pi is powered off.
The firmware is flashed right before the Pi boots back up, so please wait until the system reboots on its own before removing power.

If the update fails or is otherwise interrupted, the firmware will not be installed. You can retry the firmware installation by re-running the utility.

If your firmware is up to date, no newer firmware will be listed.

	Installed firmware: 3.1
	Newer firmware in /usr/lib/beepy-firmware:
		(None found)

All available firmware images can be listed with

	/sbin/update-beepy-fw --list

	Beepy firmware updater
	Installed firmware: 3.0
	Installed firmware in /usr/lib/beepy-firmware:
		 3.0: beepy_3.0.hex

	Older firmware in /usr/lib/beepy-firmware:
		 2.9: beepy_2.9.hex

A custom second stage firmware image can be flashed with

	/sbin/update-beepy-fw path_to_custom_firmware.hex

Second stage firmware files start with a header line followed by the firmware image in Intel HEX format.

	+My custom Beepy firmware
	:020000041000EA
	...

### `beepy-kbd`

#### Alt and Sym modifiers

The alternate symbols printed directly on the Beepy keys are triggered by pressing the physical Alt key, then the key on which the symbol is printed.

For additional symbols not printed directly on the keys, the Sym key is used. Sym sends AltGr (Right Alt), which is mapped to more symbols via the keymap file at `/usr/share/kbd/keymaps/beepy-kbd.map`.

#### Sticky modifier keys

The keyboard driver supports sticky modifier keys. Holding a modifier key (Shift, Alt, Sym) while typing an alpha keys will apply the modifier to all alpha keys until the modifier is released.

One press and release of the modifier will enter sticky mode, applying the modifier to the next alpha key only. If the same modifier key is pressed and released again in sticky mode, it will be canceled.

Visual mode indicators are drawn in the top right corner of the display, with indicators for Shift, Physical Alt, Control, Alt, Symbol, and Meta mode.

#### Other key mappings

- Physical Alt + Enter is mapped to Tab
- Holding Shift temporarily enables the touch sensor until Shift is released

- Single click
	- Call: Control
	- Berry: Enter Meta mode (see the section on Meta Mode)
	- Touchpad: Enable optical touch sensor, sending arrow keys. Subsequent clicks send Enter
	- Back: Escape
	- End Call: Tmux prefix (customize the prefix in the keymap file)

- Short hold (1 second)
	- Call: Lock Control
	- Berry: Display Meta mode overlay (requires `beepy-symbol-overlay` package)
	- End Call: Open Tmux menu (requires `beepy-tmux-menus` package and Tmux configuration)
	- Symbol: Display Symbol overlay (requires `beepy-symbol-overlay` package)

- Long hold (5 seconds)
	- End call: send shutdown signal to Pi

#### Meta mode

Meta mode is a modal layer that assists in rapidly moving the cursor and scrolling with single keypresses. To enter Meta mode, click the touchpad button once. Then, the following keymap is applied, staying in Meta mode until dismissed:

- E: up, S: down, W: left, D: right
    - Why not WASD? This way, you can place your thumb in the middle of all four of these keys, and more fluidly move the cursor without mistyping
- R: Home, F: End, O: PageUp, P: PageDown
- Q: Alt+Left (back one word), A: Alt+Right (forward one word)
- T: Tab (dismisses meta mode)
- X: Apply Control to next key (dismisses meta mode)
- C: Apply Alt to next key (dismisses meta mode)
- 0: Toggle Sharp display inversion
- N: Decrease keyboard brightness
- M: Increase keyboard brightness
- $: Toggle keyboard backlight
- Esc: (Back button): exit meta mode

Typing any other key while in Meta mode will exit Meta mode and send the key as if it was typed normally.

#### `sysfs` Interface

The following sysfs entries are available under `/sys/firmware/beepy`:

- `led`: 0 to disable LED, 1 to enable, 2 to flash, 3 to flash until key pressed. Write-only
- `led_red`, `led_green`, `led_blue`: set LED color intensity from 0 to 255. Write to `led` to apply color settings. Write- only
- `keyboard_backlight`: set keyboard brightness from 0 to 255. Write-only
- `battery_raw`: raw numerical battery level as reported by firmware. Read-only
- `battery_volts`: battery voltage estimation. Read-only
- `battery_percent`: battery percentage estimation. Read-only

#### Module parameters

Write to `/sys/module/beepy_kbd/parameters/<param>` to set, or unload and reload the module with `beepy-kbd param=val`.

- `touch_act`: one of `click` or `always`
  - `click`: default, will disable touchpad until the touchpad button is clicked
  - `always`: touchpad always on, swiping sends touch input, clicking sends Enter
- `touch_as`: one of `keys` or `mouse`
  - `keys`: default, send arrow keys with the touchpad
  - `mouse`: send mouse input (useful for X11)
- `touch_shift`: default on. Send touch input while the Shift key is held
- `sharp_path`: Sharp DRM device to send overlay commands. Default: `/dev/dri/card0`

#### Custom Keymap

The Alt and Sym keymaps and the Tmux prefix sent by the "Berry" key can be edited in the file `/usr/share/kbd/keymaps/beepy-kbd.map`. To reapply the keymap without rebooting, run `loadkeys /usr/share/kbd/keymaps/beepy-kbd.map`.

### `sharp-drm`

Linux DRM Driver for 2.7" Sharp Memory LCD

#### Module parameters

Write to `/sys/module/sharp_drm/parameters/<param>` to set, or unload and reload the module with `sharp-drm param=val`.

- `mono_cutoff`: Consider all pixels with one of R, G, B below this threshold to be black, otherwise white (default 32)
- `mono_invert`: 0 for white-on-black, 1 for black-on-white
- `indicators`: 0 to disable mode indicators (default enabled)

### `beepy-tmux-menus`

Fork of `https://github.com/jaclu/tmux-menus` customized for Beepy. Can be trigged with a 1 second hold of the End Call key by adding the following line to `~/.tmux.conf`:

```
bind-key e run-shell "/usr/share/beepy-tmux-menus/items/main.sh"
```

#### Example Beepy Tmux configuration

```
# beepy-kbd sends C-b prefix when pressing End Call
set-option -g prefix C-b
bind-key b send-keys C-b

# Double-press End Call to switch to last window
bind-key C-b last-window

# Short hold End Call for 1 second to open Tmux menu
bind-key e run-shell "/usr/share/beepy-tmux-menus/items/main.sh"
set -g @menus_location_x 'R'
set -g @menus_location_y 'T'

# Default status bar showing battery percentage and clock
set -g status-position top
set -g status-left ""
set -g status-right "█[#(sudo cat /sys/firmware/beepy/battery_percent)] %H:%M"
set -g status-interval 30
set -g window-status-separator '█'
```

## `beepy-poll`

Automatic boot polling service. Using the firmware's automatic rewake feature, automatically wake up and run polling scripts on a 15 minute timer.

Install the default polling scripts for your user with `beepy-poll install`. Currently there is a single script installed to `~/.config/beepy-poll.d/50-gomuks` to automatically update Gomuks and print a summary of new messages. You may need to edit this file to set `GOMUKS_ROOT` to the location of the transfered Gomuks directory.

To start polling, run `beepy-poll`.

```
beepy:~ $ beepy-poll 
   Shutting down and polling in 15 min.
```

Upon waking, it will wait up to 10 seconds for network access. If no network is found, it will shut down and poll again after the next interval.

```
Poll as USER. Press any key to interrupt...
Sun Mar 31 03:18:56 PM PDT 2024
Waiting for network... (1/5)
Running poll script ~/.config/beepy-poll.d/50-gomuks...
Gomuks: 3 new messages
* Example room A: 2
* Example room B: 1

Polling done, repoll in 15 min.
```

If new messages are received, the Gomuks poller script will flash the LED until a key is pressed, even after shutting down.

You can press a key to interrupt the polling sequence while it is running and continue to a normal login.

## Cleaning old source builds of drivers

If you have not installed previous versions of the drivers from source, disregard this section.

The `bbqX0kbd` driver has been renamed to `beepy-kbd`, and `sharp` to `sharp-drm`.

Driver packages will detect if one of these old modules is installed and cancel installation of the package.

Remove the following files:

* `/lib/modules/<uname>/extra/bbqX0kbd.ko*`
* `/lib/modules/<uname>/extra/sharp.ko*`
* `/boot/overlays/i2c-bbqX0kbd.dtbo`
* `/boot/overlays/sharp.dtbo`

Rebuild the module list:

* `depmod -a`

Remove the following lines from `/boot/config.txt`:

* `dtoverlay=bbqX0kbd,irq_pin=4`
* `dtoverlay=sharp`

Remove the following lines from `/etc/modules`:

* `bbqX0kbd`
* `sharp`