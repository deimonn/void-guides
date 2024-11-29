# Setting up japanese input

To get Microsoft IME-like japanese input in Linux, you can use `mozc`. Simply install `mozc` and `ibus-mozc`:

```Shell
sudo xbps-install mozc ibus-mozc
```

You may have to reboot in order for the input method to get detected by GNOME.

After rebooting, go to **Settings > Keyboard** and add a new input source; press the three dots and select **Other**. In the search box, look for **Japanese (Mozc)**.

## Making hiragana mode be the default

By default, `mozc` will be on alphanumeric mode. With a japanese keyboard you can change into hiragana mode via the eisu key, but in the case you don't have one, you can make the japanese input default to it.

Simply open file `~/.config/mozc/ibus_config.textproto`, then on the line that says:

```
active_on_launch: False
```

Change it to:

```
active_on_launch: True
```

Finally, run the following to restart `ibus`:

```Shell
ibus write-cache; ibus restart
```
