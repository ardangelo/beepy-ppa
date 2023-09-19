# Beepy package repository

{:toc}

## Updating firmware

* [Download the latest firmware image](https://github.com/ardangelo/beepberry-rp2040/releases/latest/download/i2c_puppet.uf2)
	* *Update Aug 27 2023*: there is a known issue with SQFMI's published firmware image v2.1 causing stuck keys. Please ensure that you have installed the release from the link above.
* Slide the power switch off (left if facing up)
* Connect the Beepy to your computer via USB-C
* While holding the "End Call" key (top right on the keypad), slide the power switch on
* The Beepy will present itself as a USB mass storage device. Copy the firmware image into the drive and it will reboot with the new firmware

## Adding to APT and installing drivers

SSH into the Pi and install driver packages. The LED will remain green until drivers are installed and the system has rebooted.

	curl -s --compressed "https://ardangelo.github.io/beepy-ppa/KEY.gpg" | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/beepy.gpg >/dev/null
	sudo curl -s --compressed -o /etc/apt/sources.list.d/beepy.list "https://ardangelo.github.io/beepy-ppa/beepy.list"
	sudo apt update
	sudo apt-get -y install beepy-kbd sharp-drm
	sudo reboot

The keyboard and firmware driver package will run a preinstall check to ensure that the Beepy firmware is compatible with the driver.

If the installed firmware is detected as incompatible, the installation will be canceled.

A link to a compatible firmware release will be output as part of the error message.

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

## Package listing

### `beepy-kbd`

#### Basic key mappings

- Call is mapped to Control
- "Berry" key is mapped to Tmux prefix (customize the prefix in the keymap file)
- Touchpad click enters Meta mode (see the section on Meta mode). Double click enters touchpad scroll mode
- Back is mapped to Escape
- Holding "End Call" safely shuts down the Pi
- Physical Alt is mapped to symbols printed on the keycap
- Symbol is mapped to AltGr (Right Alt), mapped to more symbols via the keymap file
- Physical Alt + Enter is mapped to Tab

#### Alt and Sym modifiers

The alternate symbols printed directly on the Beepy keys are triggered by pressing the physical Alt key, then the key on which the symbol is printed. For additional symbols not printed directly on the keys, the Sym key is used.

#### Symbol key map

<img src="https://github.com/sqfmi/beepy-docs/blob/gh-pages/img/symbol-keys.png?raw=true" width="100%"/>

#### Sticky modifier keys

The keyboard driver supports sticky modifier keys. Holding a modifier key (Shift, Alt, Sym) while typing an alpha keys will apply the modifier to all alpha keys until the modifier is released.

One press and release of the modifier will enter sticky mode, applying the modifier to the next alpha key only. If the same modifier key is pressed and released again in sticky mode, it will be canceled.

Visual mode indicators are drawn in the top right corner of the display, with indicators for Shift, Physical Alt, Control, Alt, Symbol, and Meta mode.

#### Meta mode

Meta mode is a modal layer that assists in rapidly moving the cursor and scrolling with single keypresses. To enter Meta mode, click the touchpad button once. Then, the following keymap is applied, staying in Meta mode until dismissed:

- E: up, S: down, W: left, D: right
    - Why not WASD? This way, you can place your thumb in the middle of all four of these keys, and more fluidly move the cursor without mistyping
- R: Home, F: End, O: PageUp, P: PageDown
- Q: Alt+Left (back one word), A: Alt+Right (forward one word)
- T: Tab (dismisses Meta mode)
- X: Apply Control to next key (dismisses Meta mode)
- C: Apply Alt to next key (dismisses Meta mode)
- 0: Toggle display black/white inversion
- N: Decrease keyboard brightness
- M: Increase keyboard brightness
- $: Toggle keyboard backlight
- Touchpad click (while in Meta mode): Enable touchpad scroll mode (up and down arrrow keys)
   - Other Meta mode keys will continue to work as normal
   - Exiting meta mode will also exit touchpad scroll mode
   - Subsequent clicks of the touchpad will type Enter.
- Esc: ("Back" button): exit meta mode

Typing any other key while in Meta mode will exit Meta mode and send the key as if it was typed normally.

#### `sysfs` Interface

The following sysfs entries are available under `/sys/firmware/beepy`:

- `led`: 0 to disable LED, 1 to enable. Write-only
- `led_red`, `led_green`, `led_blue`: set LED color intensity from 0 to 255. Write- only
- `keyboard_backlight`: set keyboard brightness from 0 to 255. Write-only
- `battery_raw`: raw numerical battery level as reported by firmware. Read-only
- `battery_volts`: battery voltage estimation. Read-only
- `battery_percent`: battery percentage estimation. Read-only

#### Module parameters

Write to `/sys/module/beepy_kbd/parameters/<param>` to set, or unload and reload the module with `beepy-kbd param=val`.

- `touchpad`: one of `meta` or `keys`
    - `meta`: default, will use the touchpad button to enable or disable Meta mode
    - `keys`: touchpad always on, swiping sends arrow keys, clicking sends Enter

#### Custom Keymap

The Alt and Sym keymaps and the Tmux prefix sent by the "Berry" key can be edited in the file `/usr/share/kbd/keymaps/beepy-kbd.map`. To reapply the keymap without rebooting, run `loadkeys /usr/share/kbd/keymaps/beepy-kbd.map`.

### `sharp-drm`

Linux DRM Driver for 2.7" Sharp Memory LCD

#### Module parameters

Write to `/sys/module/sharp_drm/parameters/<param>` to set, or unload and reload the module with `sharp-drm param=val`.

- `mono_cutoff`: Consider all pixels with one of R, G, B below this threshold to be black, otherwise white (default 32)
- `mono_invert`: 0 for white-on-black, 1 for black-on-white
- `indicators`: 0 to disable mode indicators (default enabled)
