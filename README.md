# Beepy Software Repository

Hosting packages and documentation for the Beepy Raspbian fork

See the [quick start page](docs/quick-start.html) for getting started with your new Beepy device.

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

## Package Reference

* [beepy-kbd](docs/beepy-kbd.html): Keyboard driver and firmware interface
* [sharp-drm](docs/sharp-drm.html): Display driver and console font configuration
* [beepy-fw](docs/beepy-fw.html): Device firmware, updating and configuration
* [beepy-symbol-overlay](docs/beepy-symbol-overlay.html): Keymap reference utility
* [beepy-gomuks](docs/beepy-gomuks.html): Gomuks Beeper client customized for Beepy
* [beepy-tmux-menus](docs/beepy-tmux-menus.html): Tmux plugin for visually managing Tmux
* [beepy-poll](docs/beepy-poll.html): OS service to wake up and run polling scripts
