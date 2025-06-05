# Wrongly styled title bar on GNOME

If you're running GNOME on dark mode and have found that some *very specific* applications render the title bar as if GNOME was in light mode, it is likely that your GTK configuration got messed up.

You can reset the configuration by simply removing the `~/.config/gtk-4.0/settings.ini` and `~/.config/gtk-3.0/settings.ini` files (do **not** remove the folders, just the files).
