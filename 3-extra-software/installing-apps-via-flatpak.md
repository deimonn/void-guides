# Installing apps via Flatpak

A very neat way to install everyday, run-of-the-mill applications on linux these days is via flatpak. Flatpak allows you to install ***a lot*** of applications that will work right out of the box with GNOME, many of which *won't* be available via `xbps-install`.

Additionally, the installed applications will be sandboxed and only minimally touch your home directory, which helps a lot in keeping the installation clean. Resetting the entire configuration for one is as easy as `flatpak run <application> --reset`.

### Disclaimer

Not all flatpak applications are the same. Some run extremely well ([Bitwig Studio](https://flathub.org/apps/com.bitwig.BitwigStudio) is a shining example; you can't even tell the difference with its native counterpart). Others are either buggy or so severely handicapped its a hassle to use them (e.g. [Visual Studio Code](https://flathub.org/apps/com.visualstudio.code)).

You can always check out the flatpak version of a program first before deciding if its good enough; and cleanup is easy if you uninstall it after.

## Setup

To setup flatpak, simply run the following two commands:

```Shell
sudo xbps-install flatpak
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

Note that you may have to restart the session for flatpak to start working correctly.

The [flathub](https://flathub.org/) is where you'll find applications to install. For example, to install [Vesktop](https://flathub.org/apps/dev.vencord.Vesktop) (a much better maintained alternative to Discord on linux):

```Shell
flatpak install flathub dev.vencord.Vesktop
```

Just look at the [Popular apps](https://flathub.org/apps/collection/popular/1) page of flathub and you'll see many familiar faces available to install on Linux.
