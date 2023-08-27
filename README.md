# Beepy package repository

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

## `beepy-kbd` firmware check

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
