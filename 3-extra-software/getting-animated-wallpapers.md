# Getting animated wallpapers

Coming from Windows and looking for a replacement to Wallpaper Engine? I can recommend a couple of options; some of which are tied to specific desktop environments.

- [Hidamari](https://github.com/jeffshee/hidamari) is desktop environment agnostic, pretty easy to install, supports setting video files, web videos/streams and web pages as your wallpaper, and is quite configurable.

  However, it leaks memory over time, has a tendency to hide the desktop when no other windows are open, and in general seems less maintained than the alternatives.

- [Hanabi](https://github.com/jeffshee/gnome-ext-hanabi) is GNOME-specific and requires a more involved installation, only supports setting video files as wallpaper, and may or may not work on your system without significant issues.

  However, it does provide better integration with the GNOME shell (e.g. the wallpaper is animated even in the overview). The activity bar bug only occurs sometimes and can easily be fixed by disabling and re-enabling Hanabi.

- [Wallpaper Engine for Kde](https://github.com/catsout/wallpaper-engine-kde-plugin) is KDE-specific. It makes use of Wallpaper Engine files outright, so if you already have Wallpaper Engine on Steam, you can use this.

  There's a couple caveats, however. Not all wallpapers will be rendered correctly; some will even **crash the desktop shell**. Configuration is also non-existent, with only the very basics being available. Still, the ability to use wallpapers from the steam workshop without having to convert them to video first is convenient - so if you're on KDE, give it a try.

## Using Hidamari

Hidamari is pretty easy to install as it is available from the [FlatHub](installing-apps-via-flatpak.md):

```
flatpak install flathub io.github.jeffshee.Hidamari
```

Simply open the application; setting up a wallpaper is pretty straightforward.

## Using Hanabi

The installation of Hanabi is more involved as it works as a GNOME extension rather than an application.

### Installing

1.  Install `git`, `meson` and optionally `clapper` ([severely](https://github.com/jeffshee/gnome-ext-hanabi?tab=readme-ov-file#optimization) improves the performance of Hanabi):

    ```Shell
    sudo xbps-install git meson clapper
    ```

2.  Clone the `gnome-ext-hanabi` repo to a sensible location (you'll be keeping it around):

    ```Shell
    git clone https://github.com/jeffshee/gnome-ext-hanabi
    ```

3.  Change directory to the repository root and execute the install script:

    ```Shell
    cd gnome-ext-hanabi && ./run.sh install
    ```

You will have to restart GNOME to see the extension and thus enable it.

### Uninstalling

If at any point you wish to remove Hanabi after having installed it, simply disable the extension - then enter the repository root and run:

```Shell
./run.sh uninstall
```

## Using Wallpaper Engine for Kde

### Installing

First off, you **will** need Steam and Wallpaper Engine to be able to download wallpapers at all; see [Installing Steam](installing-steam.md). To install Wallpaper Engine, you will have to turn on "Enable Steam Play for all other titles" within **Steam Settings > Compatibility**.

Secondly, while "Wallpaper Engine for Kde" can be installed [from the store](https://store.kde.org/p/2194089), it will lack the ability to render scene wallpapers, which means the vast majority of them. Thus, it will be more desirable to build it from source.

I maintain a template for this plugin in my [void-templates](https://github.com/deimonn/void-templates) repository which makes the process easier; simply follow the usage instructions, replacing `<package-name>` with `wallpaper-engine-kde-plugin`.

### Configuring

Once you've installed it, simply open **System Settings > Wallpaper** and set the "Wallpaper type" to "Wallpaper Engine for Kde".

You will have to select your library location:

- If you installed steam as per [Installing Steam](installing-steam.md), your library will be at `~/.local/share/Steam`
- If you installed steam via flatpak, your library will be at `~/.var/app/com.valvesoftware.Steam/.local/share/Steam`

If you did everything correctly, you should now be able to see and set wallpapers you've installed through the steam workshop.

### Fixing crashes

If you've managed to select a wallpaper that crashes your shell, you can restore it as follows:

1.  Use `ALT + SPACE` to open the runner, then type "konsole" and press enter.

2.  Open file `~/.config/plasma-org.kde.plasma.desktop-appletsrc` with a text editor, and look for a line starting with `WallpaperSource`; remove said line.

3.  Start the shell again using the following command:

    ```Shell
    kstart plasmashell
    ```

## Finding wallpapers

> If you're using **Wallpaper Engine for Kde** this section is not relevant to you.

By far, the easiest wallpapers to use are those in the form of video files. You can find these in many places in the internet; see [mylivewallpapers.com](https://mylivewallpapers.com/) for example.

If you have Wallpaper Engine on Steam, you can extract video and web wallpapers you've subscribed to from its steam workshop folder:

1.  Open Steam (assuming you've already installed it; see [Installing Steam](installing-steam.md)), then go to **Settings > Compatibility** and check **Enable Steam Play for all other titles**. You will have to restart Steam.

2.  Install Wallpaper Engine (don't bother running it; it won't).

3.  Download wallpapers as you usually would from the Steam Workshop. All items you subscribe to will go to one of the following locations:

    - `.local/share/Steam/steamapps/workshop/content/431960` if Steam was installed via package manager.

    - `~/.var/app/com.valvesoftware.Steam/.local/share/Steam/steamapps/workshop/content/431960` if Steam was installed via flatpak.

    Make sure the wallpaper you download is in video or web form.

4.  For Hidamari, copy the videos to your `~/Videos/Hidamari` folder. Hanabi can just select them directly.

    If your wallpaper is in web form, you will probably have to locally host the files. You can do this easily with Python by running the following command in the directory containing the files:

    ```Shell
    python -m 'http.server' 1337 --bind '127.0.0.1'
    ```

    Then setting the website URL in Hidamari to `http://localhost:1337` (you can replace `1337` with whatever port you want).
