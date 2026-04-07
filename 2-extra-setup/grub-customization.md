# GRUB customization

## Customization

***Always remember*** to run `sudo update-grub` after you're done customizing, so the configuration is updated.

### Skipping the bootloader

If you only have one OS and would rather have GRUB instantly boot into it instead of giving you a 5 second countdown, go to `/etc/default/grub` and look for:

```
GRUB_TIMEOUT=5
```

Replace `5` with `0`.

### Letting GRUB remember your last choice

You can make GRUB remember the last system you booted into; it will be already selected the next time you see the GRUB menu, meaning after the countdown finishes, it will automatically boot into that.

In `/etc/default/grub`, look for:

```
GRUB_DEFAULT=0
```

And replace it with:

```
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
```

### Giving it a theme

Have you noticed those fancy bootloaders you get by installing other distros? You can achieve those through the use of GRUB themes.

Go on over to [this page](https://www.gnome-look.org/browse?cat=109&ord=rating) and see what picks your fancy; if the theme comes with installation instructions, you should follow those.

Otherwise you can do the following:

1.  Extract or move the theme files to `/boot/grub/themes/<NAME>`, replacing `<NAME>` as appropiate.

2.  In `/etc/default/grub`, look for:

    ```
    #GRUB_GFXMODE=1920x1080x32
    ```

    And uncomment it (remove the `#`). You may also wish to change it to fit your desired resolution if the default is not. The number at the end (`x32` above) is the bit-depth.

3.  To `/etc/default/grub`, append the following line:

    ```
    GRUB_THEME="/boot/grub/themes/<NAME>/theme.txt
    ```

    Replacing `<NAME>` with the name you gave to the folder. Do ensure that the `theme.txt` actually exists; otherwise you extracted the theme files incorrectly.

If I may make a recommendation, [Distro Grub Themes](https://www.gnome-look.org/p/1482847) is pretty good and even contains a Void Linux specific theme for GRUB.

### Other configuration

There's more that you can do to the GRUB configuration file than what's mentioned here. If you're savvy enough, consider looking at the [manual page](https://www.gnu.org/software/grub/manual/grub/html_node/Simple-configuration.html#Simple-configuration).
