---
title: Beepy Tmux Menus
layout: default
---

# Beepy Tmux Menus

## User Guide

This package contains an adapted version of [`tmux-menus`](https://github.com/ardangelo/beepy-tmux-menus/README.md), a Tmux plugin with popup menus to help visually manage Tmux. This fork is customized to work better on the small Beepy display.

```
┌───────────── Main menu ────────────┐
│ Application Menu -->           (a) │
│ Handling Pane -->              (p) │
│ Handling Window -->            (w) │
│ Layouts -->                    (l) │
│ Split view -->                 (v) │
│ Advanced Options -->           (d) │
├────────────────────────────────────┤
│ Navigate & select ses/win/pane (n) │
└────────────────────────────────────┘
```

### Navigation

The main menu provides a series of submenus.
* Up and down arrow keys will move the selection cursor
* For items with a key shortcut (displayed in parenthesis such as `(a)`), pressing the listed key will immediately open the submenu
* While inside a submenu, pressing Escape (mapped to the Back key on the Toolbelt) will navigate to the previous menu
* On the main menu, Escape will exit the menu plugin

### Configuring Tmux

On the default Beepy Raspbian distribution, pressing the End Call key is configured to send the Tmux prefix. A short hold (less than one second) of the End Call key will open `tmux-menus`. To manually configure `tmux-menus` to open on a short hold of the End Call key, add the following line to your `~/.tmux.conf` configuration file:

```
bind-key e run-shell "/usr/share/beepy-tmux-menus/items/main.sh"
```

The full default configuration for Beepy Tmux includes settings to switch to last window after pressing End Call (Tmux prefix) twice, as well as display a top status bar with battery status:

```
# beepy-kbd sends C-b prefix when pressing End Call
bind-key b send-keys C-b

# Double-press End Call to switch to last window
bind-key C-b last-window

# Double-press End Call to switch to last window
bind-key e run-shell "/usr/share/beepy-tmux-menus/items/main.sh"
set -g @menus_location_x 'R'
set -g @menus_location_y 'T'

# Status bar
set -g status-position top
set -g status-left ""
set -g status-right "_ [#(cat /sys/firmware/beepy/battery_percent)] %H:%M"
set -g status-interval 30
set -g window-status-separator '_'
```

### Application Menu

The first submenu of the main menu is "Application Menu". This is a menu that will change based on the active application of your Tmux pane, obtained internally by the command `tmux list-panes -F "#{pane_current_command}`. The application menu is sourced from the directory

```
/usr/share/beepy-tmux-menus/items/apps/<app>.sh
```

For example, if you are running the `nano` text editor, the default Tmux title for that window will be `nano`, and the application menu sourced from `/usr/share/beepy-tmux-menus/items/apps/nano.sh`.

```
┌─────────────── nano ───────────────┐
│ Back to Main menu  <==      (Left) │
| Help                           (h) |
| Write out                      (o) |
| Read file                      (r) |
| Where is                       (w) |
| Set mark                       (m) |
| Cut text                       (k) |
| Uncut text                     (u) |
| To spell                       (t) |
| Cursor position                (c) |
| Go to line                     (l) |
└────────────────────────────────────┘
```

This package only contains menus for `nano` and `neomutt` currently. Custom application menus can be added by adapting the existing application menus.

## Developer Guide

See [documentation for upstream tmux-menus](https://github.com/ardangelo/beepy-tmux-menus/README.md)