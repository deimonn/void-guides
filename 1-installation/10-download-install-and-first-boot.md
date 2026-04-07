# Download, installation & first boot

## Download

You can download Void Linux from the [official web site](https://voidlinux.org/). Some notes about the choice of image:

- I use the glibc version as it has better compatibility with most software.

- If you want to set up the desktop environment yourself, it is probably best to also choose a base image instead of an XFCE one.

Flashing the image to an USB is outside the scope of these guides. There are plenty of internet resources on the topic already. I personally just use [Ventoy](https://www.ventoy.net/en/index.html).

## Installation

You can choose to log in as either `anon` or `root` at your preference (really the only difference you'll feel is the need to use `sudo` everywhere as the former).

To begin the installation, just run `void-installer` as root. The installation procedure is guided and pretty straightforward, just a few notes:

- When choosing a source, ***make sure*** to select `local`, which will copy over most of what's available in the live image to the installation. These guides work with that.

- If you'd rather not dedicate an entire partition to swap space, just skip the swap partition during installation; you can set up a swap file later instead, see [Setting up a swap file](../2-extra-setup/setting-up-a-swap-file.md).

If you're lost regarding partitioning, check the [Partitioning Notes](https://docs.voidlinux.org/installation/live-images/partitions.html) in the official documentation.

If you're having trouble booting into Void Linux, see [Can't boot into Void Linux](../4-notes-troubleshooting/cant-boot-into-void-linux.md).

## First boot

### Updating the system

The first step to take is to update the whole system. You may need to invoke `xbps-install` multiple times for this, depending on how up-to-date your image was:

```Shell
sudo xbps-install -Su xbps
sudo xbps-install -Su
```

### Ensuring `date` is correct

By default, Void Linux gets installed with the `chronyd` service enabled, so your system clock should already be getting synced via internet connection.

You can check by running the `date` command. If it returns an incorrect result, its very likely your hardware clock is not on the correct time setting.

- Check whether it is correct with `sudo hwclock --localtime`; if its correct with this setting, then its most likely because you've booted Windows before.

  Windows likes `localtime`, whereas Linux prefers `utc`. You can force Windows to use `utc`; see [Windows dual-booting](../2-extra-setup/windows-dual-booting.md).

  To set the hardware clock back to utc from linux, run `sudo hwclock --systohc --utc`.

- If its not, then the issue may stem from somewhere else and you're going to have to troubleshoot it by looking at the internet.

## Next steps

If all is good, then you currently hold a nice and clean Void Linux installation with no added fluff to do whatever you want with. From here, you can take it wherever.

Since these guides mostly focus around desktop configurations, I'll assume you're looking to set up a desktop environment next; in that case, continue to [Preparing for the desktop](20-desktop-environment-setup.md).
