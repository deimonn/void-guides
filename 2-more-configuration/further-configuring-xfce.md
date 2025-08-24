# Further configuring XFCE

XFCE markets itself as a lightweight desktop environment, however lightweight does not mean devoid of aesthetic - it sports a very high level of customizability even compared to behemoths like GNOME.

When you first come into contact with XFCE, perhaps through the Void live image, you may be a little horrified at how it looks and give it a pass. However, you can achieve a lot more with just a little effort and creativity. [Here's a peek](../screenshots/xfce.png) of my haphazard server install (if you're wondering, that is the Yaru theme).

## Look at the settings

Note that **most configuration is already available via the settings**. I implore you to look through them; there's a lot available out-of-the-box. Stuff like top bar transparency and shell theming are built-in.

## Getting a trash bin

Unlike for the other environments, the `xfce` package does not install `gvfs` as a dependency, which is required to be able to use the trash bin. You can do this yourself with:

```Shell
sudo xbps-install gvfs
```

## Installing plugins

The base `xfce4` package is actually quite minimal - there's a lot more you can add to the desktop by installing individual plugins. If you're lazy and/or want everything available immediately, you can install most of them at once with:

```Shell
sudo xbps-install xfce4-plugins
```

### Recommendations

This is my personal list of configured plugins which I've situated in my top bar. The name corresponds to the individual package name.

- `xfce4-whiskermenu-plugin`: Provides a nicer combined application menu and actions menu. Looks good and resembles more what you see in other desktop environments.

- `xfce4-fsguard-plugin`: Displays how much disk space you have left.

- `xfce4-diskperf-plugin`: Displays disk activity.

- `xfce4-netload-plugin`: Displays network activity.

- `xfce4-systemload-plugin`: Displays additional system load statuses.

- `xfce4-clipman-plugin`: Clipboard manager which also includes clipboard history. Once you're used to having clipboard history, you can't really live without it.
