# Further configuring GNOME

This page covers more possible configuration and recommendations for the GNOME desktop. These are all subjective; see what you wish or don't wish to do/install.

## Getting rid of GNOME packages

By default you will not be able to uninstall any of the apps installed by the `gnome` metapackage, like the default weather app, as they are listed as dependencies of it.

You can remove the metapackage, but this will leave all of the dependencies orphaned. That means that running `xbps-remove -o` (a command to clean up orphans) will uninstall the entire desktop!

The dependencies of the `gnome` metapackage must be switched to manual mode first, before being removed. Create file `remove-meta` anywhere with contents:

```Bash
#!/bin/bash

packages=$(xbps-query -x gnome | grep -o '^[-a-zA-Z_0-9]*')

for package in "$packages"; do
    xbps-pkgdb -m manual $package
done

xbps-remove -y gnome
```

Run it with `sudo bash remove-meta`, after which you can just delete it with `rm remove-meta`.

To verify it worked, run `xbps-query -O | grep gnome-tweaks`; the command should not output anything.

You can now start removing those pesky apps you'll never use! Keep in mind some of the apps have a different package name and display name; e.g. the Videos app's package is actually called `totem`; `xbps-query --search` is your friend here.

## Better terminal

The default console app in GNOME is quite lackluster and missing almost everything you'd want in a terminal application.

I recommend switching back to the old GNOME Terminal; install it with:

```Shell
sudo xbps-install gnome-terminal nautilus-gnome-terminal-extension
```

You can remove the lackluster console with:

```Shell
sudo xbps-remove -R gnome-console
```

**Note:** It can take a while to open the Preferences menu for the first time in the old GNOME terminal; if you experience this, don't worry, it only happens once per session.

## Better font manager

The default fonts app for GNOME is rather lackluster too; it requires you to install new fonts **one file at a time**. Since most fonts will be composed of a lot of files, this is beyond tedious.

You could install them via command line instead, but for convenience's sake I recommend you use [font-manager](https://github.com/FontManager/font-manager):

```Shell
sudo xbps-install fontmanager
```

Simply open Font Manager, click the "+" icon, and select all the fonts you wish to install; they will all be installed at once.

## Installing extensions

The desktop environment of GNOME can be further customized via extensions. To begin, install the `gnome-browser-connector` package:

```Shell
sudo xbps-install gnome-browser-connector
```

Afterwards navigate to the [extensions site](https://extensions.gnome.org/). There will be a banner asking you to install an add-on for your choice of browser.

After doing so, simply go to any desired extension's page and press on the switch to turn the extension from "off" to "on", which will prompt you to install it.

I recommend you at least restore tray icons with [this extension](https://extensions.gnome.org/extension/615/appindicator-support/) or one that serves the same purpose, as those are pretty significant.

### My selection

These are all the extensions I personally use.

- **Auto Move Windows** by fmuellner, available natively on Void. Allows one to, for example, move autostarted programs to other workspaces on login.

- **Removable Drive Menu** by fmuellner, available natively on Void. Useful to quickly eject any mounted drives, as well as realize at a glance if you've forgotten any mounted in the first place.

- [App Hider](https://extensions.gnome.org/extension/5895/app-hider/) by lynith; allows you to hide apps in the Overview. Specially useful for those pesky apps you can't uninstall but will never use.

- [AppIndicator and KStatusNotifierItem support](https://extensions.gnome.org/extension/615/appindicator-support/) by 3v1n0; mentioned ealier. Restores tray icons in GNOME, which I find pretty significant. There are other extensions that can do this but I found this one to work the best.

- [Caffeine](https://extensions.gnome.org/extension/517/caffeine/) by eon is very useful to keep the screen from going off indefinitely without having to change the system settings.

- [Clipboard History](https://extensions.gnome.org/extension/4839/clipboard-history/) by SUPERCILEX; once you're used to having a clipboard history, you can't live without this.

- [Custom Command List](https://extensions.gnome.org/extension/7024/custom-command-list/) by storageb; let's you define commands that will be readily accessible on a list in the top bar. Potential is limitless; I mostly use it to spawn applications with specific arguments.

- [ddterm](https://extensions.gnome.org/extension/3780/ddterm/) by amezin; a dropdown terminal just like KDE's Yakuake. Once you're used to having a terminal drop down on-demand, you can't live without this either.

- [OpenWeather Refined](https://extensions.gnome.org/extension/6655/openweather/) by tealpenguin; a neat shell extension for displaying weather information for any location on earth. I prefer this over GNOME's default weather app.

- [Resource Monitor](https://extensions.gnome.org/extension/1634/resource-monitor/) by 0ry0n; lets you stay always on status regarding resource usage in your system.

- [Steal my focus window](https://extensions.gnome.org/extension/6385/steal-my-focus-window/) by detro; tired of that little "*window* is ready" notification that often fails to take you to said window when you click it? So was I. This extension removes the notification and just allows windows to steal your focus without prompt.

- [Transparent Top Bar](https://extensions.gnome.org/extension/3960/transparent-top-bar-adjustable-transparency/) by Gonzague; if you use programs with transparency effects, adding this on top will give you bonus style points.

## Extending the GNOME theme to Qt apps

The default theme of GNOME is `adwaita` and it extends to most applications, except Qt ones. This means that not only do Qt apps look noticeably different, GNOME's dark mode does not affect them.

If you wish to achieve a more uniform look, you can override the Qt style.

Begin by installing the `adwaita-qt` package:

```Shell
sudo xbps-install adwaita-qt
```

Afterwards, append the following to your `~/.bash_profile` file:

```Bash
export QT_STYLE_OVERRIDE=adwaita-dark # or just adwaita for light mode
```

You will have to reboot for the changes to take effect.

***Note*** that some applications that do not obey GNOME's dark mode are not Qt apps, but legacy GNOME ones. For these, simply open the Tweaks app, go to Appearance, and set Legacy Applications to `Adwaita-dark` or similar.

## Creating custom color profiles

You can control color tint and temperature per-monitor via ICC profiles; you can see these as applied per monitor in **Settings > Color**. Unfortunately there's no built-in way in GNOME to edit these profiles, and calibration makes use of equipment most of us aren't willing to afford.

Thankfully, [there's already a Python script](https://github.com/zb3/gnome-gamma-tool) that can easily create and manipulate these profiles for us. Simply clone the repository and run the `gnome-gamma-tool.py` script contained within.

The following example increases the color temperature and gamma of the *second* display:

```Shell
./gnome-gamma-tool.py -d1 -t13000 -g1.2
```

## Advanced configuration via dconf

The `dconf-editor` is a graphical editor that will let you edit more advanced application settings not exposed by the UI. For example, you can force the "Files" app to always display the full path to the current folder instead of the prettified version, as well as change the default visible columns.

Simply install the `dconf-editor` package:

```Shell
sudo xbps-install dconf-editor
```

Below is some of the stuff that I personally always change.

### Changing the file explorer's default visible columns

Inside `dconf-editor`, navigate to `/org/gnome/nautilus/list-view`; the field you're looking to edit is `default-visible-columns`.

You can look at the value of `default-column-order` to tell which columns are available for display.

### Increasing the timeout limit

By default, the "application is not responding" dialog appears rather prematurely on GNOME due to its ridiculously low default timeout setting (5 seconds).

You can change this by navigating to `/org/gnome/mutter` and changing the `check-alive-timeout` setting. I've set it to `30000` (30 seconds).

### Changing file chooser behavior

File chooser settings can be found at `/org/gtk/settings/file-chooser`; for some reason these do not mirror your Nautilus settings.

Thus you may be interested in manually editing:
- `show-hidden` to be able to see hidden files.
- `sort-directories-first` if you set the option with the same name in Nautilus.
