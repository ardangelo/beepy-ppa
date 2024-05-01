# Beepy Software Repository

Hosting packages and documentation for the Beepy Raspbian fork.

See the [quick start page](docs/quick-start.html) to set up your new Beepy device with the [Beepy Raspbian distribution](https://github.com/ardangelo/beepy-gen/releases/).

## Package Documentation

* [beepy-kbd](docs/beepy-kbd.html): Keyboard driver and firmware interface
* [sharp-drm](docs/sharp-drm.html): Display driver and console font configuration
* [beepy-fw](docs/beepy-fw.html): Device firmware, updating and configuration
* [beepy-symbol-overlay](docs/beepy-symbol-overlay.html): Keymap reference utility
* [beepy-gomuks](docs/beepy-gomuks.html): Gomuks Beeper client customized for Beepy
* [beepy-tmux-menus](docs/beepy-tmux-menus.html): Tmux plugin for visually managing Tmux
* [beepy-poll](docs/beepy-poll.html): OS service to wake up and run polling scripts

Documentation for these packages is also available in manpage format. Run `man package` e.g. `man beepy-kbd` for keyboard documentation.

## Installing on stock Raspbian

If you are using your own 32-bit `armhf` or 64-bit `arm64` Raspbian Bookworm image, you can also manually install packages from this repository.
This is not necessary when using the [Beepy Raspbian distribution](https://github.com/ardangelo/beepy-gen/releases/).

Ensure you have a swapfile of at least 512MB,

	( . /etc/dphys-swapfile && if (( CONF_SWAPSIZE < 512 )); then \
        sudo sed -i 's/^CONF_SWAPSIZE=.*/CONF_SWAPSIZE=512/' /etc/dphys-swapfile \
     && sudo reboot; fi )

Then, add the Beepy PPA key, update, and install packages,

    sudo curl -s --compressed -o /etc/apt/sources.list.d/beepy.list "https://ardangelo.github.io/beepy-ppa/beepy.list"
    curl -s --compressed "https://ardangelo.github.io/beepy-ppa/KEY.gpg" | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/beepy.gpg >/dev/null
    sudo apt update
    sudo apt-get -y install beepy-fw sharp-drm beepy-symbol-overlay beepy-kbd tmux beepy-tmux-menus beepy-gomuks beepy-poll
    sudo reboot

The keyboard driver package will run a preinstall check to ensure that the Beepy firmware is compatible with the driver. If the installed firmware is detected as incompatible, the installation will be canceled. The latest firmware release can be found here: <https://github.com/ardangelo/beepberry-rp2040/releases/latest/>.

