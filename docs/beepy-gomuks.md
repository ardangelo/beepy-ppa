---
title: Beepy Gomuks
layout: default
---

# Beepy Gomuks

Upstream Gomuks Readme: <https://github.com/tulir/gomuks>

- [Setting up Gomuks](#setting-up-gomuks)
  - [Logging into Gomuks](#logging-into-gomuks)
  - [Export data from computer](#export-data-from-computer)
  - [Import data to Gomuks](#import-data-to-gomuks)
- [Using Gomuks](#using-gomuks)
  - [Room list](#room-list)
  - [Message view](#message-view)

## Setting up Gomuks

See the original Gomuks Beeper instructions for more detail: <https://beeper.notion.site/Beepy-Beeper-Client-Setup-Tutorial-a2200b76f8764813bf7a70e9f69f46b3>

### Logging into Gomuks

If you have a large amount of Beeper chats, you can use Gomuks on a more powerful computer to [synchronize your account](#export-data-from-computer). It is a more involved process, however. If you don't have a large amount of chats, you can log into Gomuks directly without exporting.

```
     ╔═══════════Log in to Matrix═══════════╗
     ║                                      ║     
     ║ Email example@example.com            ║
     ║                                      ║
     ║       [Enter email to request code ] ║
     ║                                      ║
     ║ Code  123456                         ║
     ║                                      ║
     ║ [       Enter code to log in       ] ║
     ║                                      ║
     ║ [               Quit               ] ║
     ╚══════════════════════════════════════╝
```

Press `Alt + Enter` for `Tab`, or [Meta mode arrow keys](beepy-kbd.html#meta-mode) to move between fields. For keymaps including arrow keys and symbols, refer to [`beepy-kbd` keymap reference](beepy-kbd.html#basic-key-mappings).

* Enter your email address used to sign up for Beeper.
* The first button will change to `Request code`.
* Select the button, and you should recieve a 6-digit code in your email.
* Type the code into the `Code` text box.
* The second button will change to `Log in`.

After logging in, refer to [Using Gomuks](#using-gomuks).

### Export data from computer

If you have a large amount of Beeper chats, you can use Gomuks on a more powerful computer to synchronize your account. Then, you can transfer the synchronized state to your Beepy device. If you would like to sync all data on your Beepy device, refer to [Logging into Gomuks](#logging-into-gomuks).

* You will need an additional device running Linux, Mac OS, or Windows with the Beeper Desktop application installed and logged in.
* You will also need to have set up a password for your Beeper account. To do this, contact Beeper Help, or use this third-party script: <https://github.com/0xdevalias/poc-beeper-password-reset>.
* Download the Gomuks binary:
    - Linux [ARMv7](https://mau.dev/tulir/gomuks/-/jobs/35903/artifacts/download), [ARM64](https://mau.dev/tulir/gomuks/-/jobs/35912/artifacts/download), [x86-64](https://mau.dev/tulir/gomuks/-/jobs/35902/artifacts/download)
    - Mac OS [Universal](https://mau.dev/tulir/gomuks/-/jobs/35908/artifacts/download), [ARM64](https://mau.dev/tulir/gomuks/-/jobs/35907/artifacts/download), [x64-64](https://mau.dev/tulir/gomuks/-/jobs/35906/artifacts/download). You may also need to install `libolm` with `brew install libolm`.
    - Windows [x86-64](https://mau.dev/tulir/gomuks/-/jobs/35905/artifacts/download)
* Make the binary executable: `chmod +x gomuks`
* Export end-to-end encryption keys
    - Open Beeper Desktop, navigate to `Gear > Settings > Security & Privacy`, heading `Cryptography`, click `Export E2E room keys`
    - Enter a password to help secure the keys.
    - Save the file as `keys.txt` in the same folder as the `gomuks` binary.
* Run `./gomuks --log-in-for-transfer`
    - Sign in to your Beeper account, with the username `@username:beeper.com` and password created from Beeper Support or using the script.
    - Input encryption key password.

### Import data to Gomuks

Copy the exported folder to your Beepy device using `scp`.

    scp -r /path/to/transfer user@beepy_ip:/destination/transfer

Then, edit the file at `<transfer>/config/config.yaml` to update the following paths to the destination directory:

    data_dir: <transfer>/data/gomuks
    cache_dir: <transfer>/cache
    history_path: <transfer>/cache/history.db
    room_list_path: <transfer>/cache/rooms.gob.gz
    media_dir: <transfer>/cache/media
    state_dir: <transfer>/cache/state

For example, if you transfered the folder to the path `/destination/transfer/`, update the `data_dir` setting to

    data_dir: /destination/transfer/data/gomuks

Finally, edit your `~/.profile` or other configuration to set the environment variable `GOMUKS_ROOT` to the path of the transfer directory. If you are using the [`beepy-poll` Gomuks polling service](beepy-poll.html#50-gomuks-gomuks-polling-script), also update the polling script `GOMUKS_ROOT` setting at `~/.config/beepy-poll.d/50-gomuks`.

## Using Gomuks

### Room list

On the Beepy's small screen, the default view for Gomuks is a single-column view, opening to the room list.

```
GOMUKS                                       12:00
* My Name and Telegram bridge bot       2024-03-31
  Last Message
Note to self                            2024-03-31
  Note Contents
Beeper Help                             2024-03-27
  Welcome to Beeper!
Beeper Updates                          0001-01-01
  Beeper Android Beta v4.1.48
```

Exit Gomuks from the room list by pressing the `Back` key, mapped to `Escape`.

The selected room is highlighted, and prefixed with a diamond marker. Scroll the list using arrow keys ([Meta mode arrow keys](beepy-kbd.html#meta-mode) `Berry + E/S` or [touchpad arrow keys](beepy-kbd.html#touchpad-mode)), and select a room using `Enter`. For keymaps including arrow keys and symbols, refer to [`beepy-kbd` keymap reference](beepy-kbd.html#basic-key-mappings).

In the room list, press `k` to open a room search dialog. Start typing the name of the room to narrow down the list, then `Enter` to select the room, or `Escape` to exit the dialog without selecting a room.

Press `s` to open the Gomuks settings menu.

* `Notify on new messages` Configure new message behavior.
    - `Flash LED until key pressed` New messages will flash the notification LED.
    - `Notification LED disabled` New messages will not flash the notification LED.
* `Room list` Configure the room list view style.
	- `Group rooms by tag` Group rooms into a collapsible list based on room type.
	- `Sort by updated` The default view, sort all rooms in a single list by last message.

### Message view

The selected room view shows a list of all messages for that room, and a text entry box.

```
Chat with Other Person
                           Scroll up to load more 

                  other-persons-name · 17:00:00 <
                      Message from Other Person
Date changed to March 31, 2024                  
> My Name · 07:00:00
Message to Other Person
                                   
Send an encrypted message... 
```

Scroll the message view using arrow keys, or using Page Up and Page Down ([Meta mode](beepy-kbd.html#meta-mode), `Berry + O/P`). Type a message using the alpha keys, and press `Enter` to send.

Exit the message view and return to the room list by pressing the `Back` key, mapped to `Escape`.

On a larger screen, such as connected through SSH, Gomuks will display in a two-column view. The currently selected view, either chat list or message view, will be highlighted with a green header bar.
