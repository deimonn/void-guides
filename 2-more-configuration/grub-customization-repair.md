# GRUB customization & repair

GRUB can be further configured and customized by editing the `/etc/default/grub` file. This page covers various customizations I like to do to mine.

It also covers how to fix the GRUB bootloader if its entry got removed from BIOS or its partition contents got messed up.

## Customization

***Always remember*** to run `sudo update-grub` after you're done customizing, so the configuration is updated.

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

## Repairing

If you find yourself unable to boot into GRUB (and thus Void Linux), it is most likely because the bootloader entry got removed (e.g. if you unplugged the disk to install Windows, BIOS may have automatically removed your entry). It could also be that the files in the EFI partition got damaged somehow.

Either way, it is quite the simple fix. First you will have to flash an USB with the Void Linux installation medium, as you presumably did to install Void originally; then boot into it.

Inside the installation medium, mount your actual system to `/mnt`, like:

```Shell
mount /dev/sdXA /mnt
mount /dev/sdXB /mnt/boot/efi
```

- Replace *X* with the letter of the disk where your Void Linux system is. You can find it by running `fdisk -l`.
- Replace *A* with the number of your root partition (the one of type "Linux root" or similar).
- Replace *B* with the number of your EFI partition (the one of type "EFI System").

Now chroot into it with:

```Shell
xchroot /mnt
```

If you think the partition may have had its contents modified, I recommend removing existing files from the partition with:

```Shell
rm -rf /boot/efi/**
```

Finally, to re-install GRUB, simply run:

```Shell
grub-install
```

You can now run `exit` then `reboot`. You should be able to boot into your Linux system once more.
