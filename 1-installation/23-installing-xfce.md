# Setting up XFCE

## Getting the desktop

### Installing packages

Install the X server and XFCE metapackage:

```Shell
sudo xbps-install xorg xfce4
```

### Setting up a display manager

XFCE does not come with a display manager, so one must be installed explicitly; `lightdm` and `lxdm` are relatively light options that work pretty well.

Install and enable one of the two with:

```Shell
sudo xbps-install <DM>
sudo touch /etc/sv/<DM/down
sudo ln -s /etc/sv/<DM> /var/service
```

Replace `<DM>` with either `lightdm` or `lxdm`, at your discretion.

After a reboot, the desktop session can be started with:

```Shell
sudo sv start <DM>
```

## On the desktop

You should've loaded into XFCE at this point. If you did, congratulations! You can go ahead and make your display manager start automatically if so:

```Shell
sudo rm /var/service/<DM>/down
```

If you have more than one GPU and starting either display manager causes you to get stuck, see [Can't start X server with multiple GPUs](../4-notes-troubleshooting/cant-start-x-server-with-multiple-gpu.md).

The rest of configurations here are all optional and subjective; see what you wish or don't wish to do/install.

### Getting a trash bin

Unlike GNOME or KDE, the XFCE package does not install `gvfs` as a dependency, which is required to be able to use the trash bin. You can do this yourself with:

```Shell
sudo xbps-install gvfs
```

### Ricing it up

When you first come into contact with XFCE, perhaps through the Void live image, you may be a little horrified at how ancient it looks and give it a pass. However, you can achieve a lot more with just a little effort and creativity.

[Here's a peek](../screenshots/xfce.png) of my haphazard server install (if you're wondering, that is the Yaru theme).

#### Look at the settings

Note that **most configuration is already available via the settings**. I implore you to look through them; there's a lot available out-of-the-box. Stuff like top bar transparency and shell theming are built-in.

#### Installing plugins

The base `xfce4` package is actually quite minimal - there's a lot more you can add to the desktop by installing individual plugins. If you're lazy and/or want everything available immediately, you can install most of them at once with:

```Shell
sudo xbps-install xfce4-plugins
```

Below is my personal list of configured plugins which I've situated in my top bar. The name corresponds to the individual package name.

- `xfce4-whiskermenu-plugin`: Provides a nicer combined application menu and actions menu. Looks good and resembles more what you see in other desktop environments.

- `xfce4-fsguard-plugin`: Displays how much disk space you have left.

- `xfce4-diskperf-plugin`: Displays disk activity.

- `xfce4-netload-plugin`: Displays network activity.

- `xfce4-systemload-plugin`: Displays additional system load statuses.

- `xfce4-clipman-plugin`: Clipboard manager which also includes clipboard history. Once you're used to having clipboard history, you can't really live without it.

## Next steps

You'll probably want to set up audio next; [Installing an audio and media server](30-setting-up-audio.md) explains how to do that.
