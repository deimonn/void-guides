# Setting up Sway

## Getting the desktop

### Installing packages

Install the `sway`, `foot` and `wmenu` packages:

```Shell
sudo xbps-install sway foot wmenu
```

You will also need at least one font or nothing will render. If you don't have any particular choice in mind, `noto-fonts-ttf` ought to do:

```Shell
sudo xbps-install noto-fonts-ttf
```

### Starting the shell

While Sway can be started through a display manager, all it takes to start the shell is just one command:

```Shell
XDG_CURRENT_DESKTOP=sway:wlroots exec dbus-run-session sway
```

To have it automatically start, you can just add it to your `.bash_profile` like this:

```Shell
# Start Sway on main TTY login.
if [[ $(tty) = /dev/tty1 ]]; then
    XDG_CURRENT_DESKTOP=sway:wlroots exec dbus-run-session sway
fi
```

You can even expand it to ask whether to start Sway first:

```Shell
# Ask to start Sway on main TTY login.
if [[ $(tty) = /dev/tty1 ]]; then
    read -p "Start Sway? [y]es/[N]o" -n 1 -r

    if [[ $REPLY =~ ^[Yy]$ ]]; then
        XDG_CURRENT_DESKTOP=sway:wlroots exec dbus-run-session sway
    fi
fi
```

This way, if Sway crashes or you exit, you'll be dropped back into a TTY that you can just log back in to and fix any issues.

Alternatively, see the [Login managers](https://github.com/swaywm/sway/wiki/Useful-add-ons-for-sway#login-managers) section in the Sway wiki for a list of compatible display managers.

## On the desktop

You should've loaded into Sway at this point. If you did, congratulations!

There's still quite a bit to do however, as Sway is a pretty barebones system as-is; on the flip side, its also extremely modular and customizable.

If you're new to the Sway/i3 workflow, I recommend you watch this [i3wm Jump Start video](https://www.youtube.com/watch?v=j1I63wGcvU4) and this [i3wm Configuration video](https://www.youtube.com/watch?v=8-S0cWnLBKg). While they cover i3 rather than Sway, most of the configuration still applies (except for things Sway already handles by itself, such as monitors and wallpapers).

### Copying the default config

A good starting point for beginning to set up Sway is to copy the default configuration. Simply do:

```Shell
mkdir ~/.config/sway
cp /etc/sway/config ~/.config/sway/
```

Then open `~/.config/sway/config` to edit the configuration. The comments should be able to guide you.

You should also **definitely** check out the [Configuration](https://github.com/swaywm/sway/wiki#configuration) section of the wiki, which covers display monitors, input configuration, wallpapers and more. These will not be covered on this guide to avoid repetition.

### Setting up the XDG environment

The XDG desktop portal will let applications installed via [flatpak](../3-extra-software/installing-apps-via-flatpak.md), as well as some others, perform common desktop tasks such as sending notifications or screensharing.

There are multiple portal backend implementations, often designed for specific desktops; but two are considered generic and should work well with Sway:

```Shell
sudo xbps-install xdg-desktop-portal-wlr xdg-desktop-portal-gtk
```

Additionally, many applications expect XDG utilities to be available, for example to be able to open links:

```Shell
sudo xbps-install xdg-utils
```

Lastly, its a good idea to generate the usual directories found in one's home (such as Downloads) so programs that make use of them can open them without problems:

```Shell
sudo xbps-install xdg-user-dirs-gtk && xdg-user-dirs-update && xdg-user-dirs-gtk-update
```

### Setting up the D-Bus environment

For the XDG desktop portals to get started automatically, the D-Bus session must be updated with the appropiate environment variables. This is pretty simple to do from `~/.config/sway/config`, just append the following:

```Shell
exec dbus-update-activation-environment WAYLAND_DISPLAY DISPLAY XDG_CURRENT_DESKTOP SWAYSOCK I3SOCK XCURSOR_SIZE XCURSOR_THEME
```

### Getting a file manager

Sway by default does not come with a file manager. If you need one that is relatively flexible while remaining light, a popular choice is Thunar:

```Shell
sudo xbps-install Thunar thunar-archive-plugin
```

The archive plugin enables options like "Extract here..." if you also install `xarchiver`, `file-roller`, `ark` or `engrampa` at your preference.

### Getting a trash bin

To enable the trash bin in supported applications (such as Thunar), you need to install `gvfs`:

```Shell
sudo xbps-install gvfs
```

This also enables the mounting of disks and SSH filesystems via the file manager, but note that mounting also requires a PolKit agent to be present; see below.

### Setting up a PolKit agent

PolKit agents allow that little dialog asking you to input your password to appear when you're trying to do something that requires root, such as mounting a disk. Sway does not have an associated PolKit agent; you need to install one from another distribution. See:

```Shell
xbps-query -Rs polkit
```

For example,  you could install XFCE's, `xfce-polkit`. Then all you have to do is start it. To do it automatically on session start, add the following to your `~/.config/sway/config`:

```Shell
exec /usr/libexec/xfce-polkit
```

Replace `xfce-polkit` with the name of the PolKit Agent binary you installed.

### Changing default applications

You can set default applications to use to open files by editing `~/.config/mimeapps.list`. If modifying the text file is too much of a hassle, [selectdefaultapplication](https://github.com/sandsmark/selectdefaultapplication) exists, but it has to be built from source.

Alternatively, another desktop environment's settings app can be installed, such as XFCE's.

## Additional desktop stuff

There are tons of great add-ons for sway; definitely check out [Useful addons](https://github.com/swaywm/sway/wiki/Useful-add-ons-for-sway) on the Sway Wiki to see what you can get up to.

Here are some of the things I've set up.

### Terminal & launcher

In the first step, I made you install `foot` and `wmenu` as those are the terminal and launcher applications set on the default Sway config. If you replace them in your config, feel free to uninstall them afterwards.

Your choice of terminal and launcher are entirely up to preference, and shouldn't make too big of a difference or require much setup besides changing the variables in the config.

### XCompose

Keyboard configuration is covered in the Sway wiki, but if all you need is to set your layout to `us` and enable XCompose, here's the short version of it:

```
input type:keyboard {
    xkb_layout "us"
    xkb_options compose:ralt
}
```

This allows you to compose characters using right-alt. For other keys, see:

```Shell
cat /usr/share/X11/xkb/rules/base | grep "compose:"
```

### Status bar

By far one of the most popular choices for a status bar on Sway appears to be [Waybar](https://github.com/Alexays/Waybar). It is quite flexible and overall works pretty well, bar some minor issues. You can install it with:

```Shell
sudo xbps-install Waybar
```

And enable it by appending the following to `~/.config/sway/config`:

```Shell
bar swaybar_command waybar
```

You can copy the default configuration with:

```Shell
mkdir -p ~/.config/waybar
cp /etc/xdg/waybar/* ~/.config/waybar
```

See the repo for examples; the man pages are also pretty thorough in explaining the config file.

Note that to render icons, you'll need an icon font. `font-awesome6` looks great but has a very limited set; `nerd-fonts` has an extremely wide variety, but requires ~8GiB of disk space. Whichever you choose, both are available on the official Void repos.

### Idle behavior

Turning off monitors or locking the session after a certain period of inactivity is not functionality that comes with Sway. For that, you must install `swayidle`:

```Shell
sudo xbps-install swayidle
```

To turn off the monitors automatically after 30s of inactivity, you can use:

```Shell
exec swayidle timeout $timeout 'swaymsg "output * power off"' resume 'swaymsg "output * power on"'
```

See the `swayidle` man page.

### Notifications

For notifications to work, you will need a notifications daemon. There are a lot of choices and most aren't too complicated to set up (just an `exec` in your config).

If you don't need much, a lightweight, simplistic daemon is `mako`.

For a full on notification center, with the possibility of adding buttons to it and a high degree of customizability, try `SwayNotificationCenter`. The default configuration files can be copied with:

```Shell
mkdir -p ~/.config/swaync
cp /etc/xdg/swaync/config.json ~/.config/swaync
cp /etc/xdg/swaync/style.css ~/.config/swaync
```

### Screenshotting

There are many screenshot utilities available to Sway, but I've found `grimshot` to work best; simply install it with:

```Shell
sudo xbps-install grimshot
```

The command I like to use is `grimshot copy anything`, which lets you choose a screen, window, or region to copy to the clipboard as an image. I have it bound to the print screen key.

### Color picking

While its possible to do color picking just using `grim` and a long series of pipes, I prefer to just use `hyprpicker`:

```Shell
sudo xbps-install hyprpicker
```

Running `hyprpicker -a` lets you select a color on the screen more intuitively to then copy it to the clipboard as hexadecimal.

### Clipboard history

I set up clipboard history via [cliphist](https://github.com/sentriz/cliphist) as it seems to be the simplest implementation that remains featureful. Just install the package:

```Shell
sudo xbps-install cliphist
```

Then add the following to `~/.config/way/config`:

```Shell
exec wl-paste --watch cliphist store
```

Once the daemon is running, to copy an item from clipboard history just run the following command (you may want to assign it a keybind):

```Shell
cliphist list | <MENU> | cliphist decode | wl-copy
```

Replace `<MENU>` with a menu tool of your choice, such as `wofi`, `rofi` or some other application launcher with the capability.

See the [GitHub page](https://github.com/sentriz/cliphist?tab=readme-ov-file#usage) for instruction on delete and clear as well.

### Night light

A nice night light implementation is [wl-gammarelay-rs](https://github.com/MaxVerevkin/wl-gammarelay-rs), which runs as daemon rather than an application. You can install it with:

```Shell
sudo xbps-install wl-gammarelay-rs
```

Then just add an `exec` for it in your config.

See the [Status bar integration](https://github.com/MaxVerevkin/wl-gammarelay-rs?tab=readme-ov-file#status-bar-integration) instructions for how you could set it up with e.g. Waybar.

## Ricing Sway

This section is dedicated solely to aesthetic enhancements without any change in actual functionality.

### Sway with eye candy

For blur, anti-aliased rounded borders, shadows and other effects, install SwayFX:

```Shell
sudo xbps-install swayfx
```

SwayFX is a fork of Sway and works as a drop-in replacement; in fact, installing this package on Void removes the regular variant. To see the effects in action, restart the shell.

See the [README](https://github.com/WillPower3309/swayfx?tab=readme-ov-file#new-configuration-options) for information on how to enable the various effects.

### Installing themes for apps

Installing themes is relatively straightforward:

- Fonts go in `~/.fonts`
- Icon themes go in `~/.icons`
- GTK themes go in `~/.themes`
- Kvantum themes go in `~/.config/Kvantum`

Getting them to apply, however, is a lot more involved. I recommend you automate all of the following operations in your Sway config via `exec_always` if you don't want to lose your sanity just to change a theme.

#### GTK 2 applications

For GTK 2, simply create file `~/.gtkrc-2.0` and write the following to it:

```ini
gtk-icon-theme-name = "<ICON_THEME>"
gtk-theme-name = "<GTK_THEME>"
gtk-font-name = "<FONT_NAME> <FONT_SIZE>"
```

Replace `<ICON_THEME>`, `<GTK_THEME>`, `<FONT_NAME>` and `<FONT_SIZE>` as appropiate.

#### GTK 3 & GTK 4 applications

Applying themes for GTK 3 & 4 is not too disimilar, simply create `~/.config/gtk-3.0/settings.ini` and write to it:

```ini
[Settings]
gtk-icon-theme-name = "<ICON_THEME>"
gtk-theme-name = "<GTK_THEME>"
gtk-font-name = "<FONT_NAME> <FONT_SIZE>"
gtk-application-prefer-dark-theme = <TRUE_OR_FALSE>
```

Additionally, some applications might expect the following to be set via `gsettings`:

```Shell
gsettings set org.gnome.desktop.interface icon-theme <ICON_THEME>
gsettings set org.gnome.desktop.interface gtk-theme <GTK_THEME>
exec_always gsettings set org.gnome.desktop.interface color-scheme <PREFERENCE>
```

Where `<PREFERENCE>` should be `prefer-dark` or `prefer-light`.

#### GTK 4 applications using `libadwaita`

GTK 4 applications based on `libadwaita` are more annoying, as they refuse to obey the above given settings. The only way I've found to get them to acquiesce is by setting `GTK_THEME` prior to starting them:

```Shell
GTK_THEME=<GTK_THEME> <APP>
```

To get this to happen dynamically, I simply prefixed `GTK_THEME=<GTM_THEME>` to my launcher. Its not pretty, but alas.

#### QT applications

QT applications are by far easiest to theme via Kvantum:

```Shell
sudo xbps-install kvantum
```

You can set a theme via the provided GUI, or using command:

```Shell
kvantummanager --set <KVANTUM_THEME>
```

To get the theme to apply, `QT_STYLE_OVERRIDE` must be set. This is most easily done by writing the following to your `~/.bash_profile`, prior to starting Sway:

```Shell
export QT_STYLE_OVERRIDE=kvantum
```

## Next steps

You'll probably want to set up audio and screenshare next; [Installing an audio and media server](30-setting-up-audio.md) explains how to do that.
