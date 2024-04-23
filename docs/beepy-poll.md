---
title: Beepy Polling Service
layout: default
---

# Beepy Poll

## User Guide

The `beepy-poll` package installs a boot-time polling utility `beepy-poll`, as well as an idler service `beepy-idle`.

### `beepy-poll`

Running `beepy-poll` with no arguments will shut down the Pi, and schedule the Pi to boot back up in 15 minutes.

If woken back up on this timer, the service will wait for network access, then run all scripts in the directory `~/.config/beepy-poll.d/`. When all the scripts have completed, it will shut down again and wait for another 15 minutes.

    Poll as USER. Press any key to interrupt...
    Mon Jan 01 12:00:00 CDT 2024
    Waiting for network...
    Waiting for NetworkManager... (1/5)
    Waiting for network connection... (1/5)
    Running ~/.config/beepy-poll.d/50-gomuks
    Gomuks: 3 new messages
    * Example room A: 2
    * Example room B: 1

If no network connection is found, then the poller will not run the scripts, and schedule another rewake in 15 minutes.

    Waiting for network...
    Network not found, repoll in 15 min.

Pressing any key during the poll will cancel the poll and continue with a full boot process.

### `50-gomuks` Gomuks polling script

The `beepy-poll` package provides a script `50-gomuks` that will run the [Gomuks messenger](beepy-gomuks.html) in headless poll mode using `gomuks --headless`. Any new messages will cause the notification LED to flash until a key is pressed.

It is installed to `/etc/skel/.config/beepy-poll.d/50-gomuks`. The default Beepy Raspbian install has already run this step for the first user, and you can find your user's copy at `~/.config/beepy-poll.d/50-gomuks`.

### `beepy-poll install`

This will copy all default poller scripts from `/etc/skel/.config/beepy-poll.d/` to `~/.config/beepy-poll.d/`. Then, it will run each script with the argument `install`.

The default Beepy Raspbian install has already run this step for the first user, and you can find the default Gomuks poller script at `~/.config/beepy-poll.d/50-gomuks`.

### Adding new scripts

The `beepy-poll` package provides the Gomuks poller script ``/etc/skel/.config/beepy-poll.d/50-gomuks` On the default Beepy Raspbian configuration, it is also installed to the default user account at `~/.config/beepy-poll.d/50-gomuks`.

You can reference it to create other polling scripts. It uses `screen` to create a virtual sized TTY, as Gomuks requires starting its text-based graphical interface. For a simple terminal polling script, you do not need to use `screen`, and can send output to standard out.

    #!/bin/bash

    if [ "$1" == "install" ]; then
        SCRIPTPATH="$( cd -- "$(dirname "$0")" >/dev/null 2>&1 ; dirs +0 )"
        echo "Polling script ran from"
        echo "    $SCRIPTPATH/$(basename $0)"
        exit 0
    fi

    /run/your/command/here

### `beepy-idle`

On first setup of Beepy Raspbian, you will be presented with an option to either enable or disable the `beepy-idle` service. If enabled, this service will automatically shut down the Pi and deep sleep the RP2040 controller to preserve battery life.

The Pi will not shut down unless it has been a configurable amount of minutes since the last keypress, or if Tmux is running an active foreground process.

The timeout and allowed processes can be configured by editing the file at `/etc/beepy-idle.conf`. By default, if Tmux is running any process other than `bash` or `gomuks`, idle will be inhibited.

    # Automatic idle shutdown time in minutes
    # since last keypress. Change to 0 to disable
    IDLE_TIME=5

    # Allow automatic shutdown if the following processes
    # are running in tmux, otherwise deny
    ALLOW_PROCS="bash gomuks"

To disable `beepy-idle`, stop the idle service and idle timer:

    sudo systemctl disable beepy-idle.service
    sudo systemctl disable beepy-idle.timer

To enable `beepy-idle`, enable and start the idle timer:

    sudo systemctl enable beepy-idle.timer
    sudo systemctl start beepy-idle.timer

## Developer Reference

The [`beepy-kbd`](beepy-kbd.html) keyboard driver provides several sysfs entries. The [`rewake_timer`](beepy-kbd.html#sysfs-interface) entry, when written, shuts down the Pi, and boots it back up in that many minutes. After booting back up using `rewake_timer`, the [`startup_reason`](beepy-kbd.html#sysfs-interface) sysfs entry will contain the text `rewake`.

The `beepy-poll` utility will set and check these sysfs entries to determine if the system was woken up as part of a `rewake` timer. If so, it will run the poller scripts and then schedule another rewake.
