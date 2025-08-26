# Setting up japanese input

To get Microsoft IME-like japanese input in Linux, you can use `mozc`. Simply install `mozc` and `ibus-mozc`:

```Shell
sudo xbps-install mozc ibus-mozc
```

You may have to reboot in order for the input method to get detected by GNOME.

After rebooting, go to **Settings > Keyboard** and add a new input source; press the three dots and select **Other**. In the search box, look for an input method called **Japanese (Mozc)** or similar.

To default the japanese input to kana, use **Japanese (Mozc-あ)**.
