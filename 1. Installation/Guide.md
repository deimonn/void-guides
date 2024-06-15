# Void Linux plus GNOME Installation Guide

A guide to installing Void Linux and setting up the GNOME desktop environment, along with a plethora of other smaller configurations.

I originally wrote these as "install notes" for myself, but I decided to rewrite it in a more agnostic manner and make it public in case its useful to anyone.

Do note however, that this guide assumes you already have some experience with Linux. It is most definitely not newbie friendly.

## Running the installer

Simply run `void-installer` to begin the installation process. See the [Void Linux Installation Guide](https://docs.voidlinux.org/installation/live-images/guide.html) for more details.

Some installation notes:

- I use the glibc version as it has better compatibility with most software.

- When choosing a source, ***make sure*** to select `local`, which will copy over most of what's available in the live image to the installation. This guide works with that.

- If you prefer to use swap files rather than a dedicated partition, skip the swap partition during installation; this guide will cover how to set up a swap file later.

The remaining options in the installer are up to you. Finish the installation before proceeding.

## First boot

Update the system. You may possibly need to invoke xbps multiple times for this:

```Shell
sudo xbps-install -Su
sudo xbps-install -u xbps
sudo xbps-install -Su
```

### Installing a text editor

Before we get on with the rest of the guide, I recommend you get a text editor; `nano` for is pretty good for editing text files on the command line:

```Shell
sudo xbps-install nano
```

You can enable syntax highlighting for the same by putting the following in `~/.nanorc`:

```
include /usr/share/nano/*.nanorc
include /usr/share/nano/extra/*.nanorc
```

And set it as the default editor by appending to `~/.bash_profile`:

```Bash
export EDITOR=nano
```

You're free to use your own choice of editor, though.

### Setting up a swap file

If you skipped setting up a swap partition like I did, you might still want to setup a swap file (having some form of swap is always good).

1.  To begin, allocate the desired amount of swap space as a file (replace `<SIZE>` on the following command):

    ```Shell
    sudo fallocate -l <SIZE> /swapfile
    ```

    I use `8GiB`.

2.  Update its permissions so only the root user can write and read to it:

    ```Shell
    sudo chmod 600 /swapfile
    ```

3.  Format and enable the swap file as you would a partition:

    ```Shell
    sudo mkswap /swapfile
    sudo swapon /swapfile
    ```

    You'll likely want to update your `/etc/fstab` too so the swap comes on automatically when the system starts:

    ```
    /swapfile swap swap defaults 0 0
    ```

You can verify your swap is active by running `sudo swapon --show`.

### Syncing the time with NTP

The NTP daemon will keep your system clock synced via internet connection. Install the package with:

```Shell
sudo xbps-install ntp
```

Enable the service with:

```Shell
sudo ln -s /etc/sv/isc-ntpd /var/service
```

If running `date` returns an incorrect result after this, its very likely your hardware clock is not on the correct time setting.

- Check whether it is correct with `sudo hwclock --localtime`; if its correct with this setting, then its most likely because you've booted Windows before.

  Windows likes `localtime`, whereas Linux prefers `utc`. You can force Windows to use `utc`; see [Windows dual-booting](../2.%20More%20Configuration/Windows%20dual-booting.md).

  To set the hardware clock back to utc from linux, run `sudo hwclock --systohc --utc`.

- If its not, then the issue may stem from somewhere else and you're going to have to troubleshoot it by looking at the internet.

## Installing the graphics driver

The first thing to do before setting up a desktop is install the appropiate graphics drivers. The following steps will differ depending on the GPU you're using.

### AMD GPU

Install the required packages with:

```Shell
sudo xbps-install mesa-dri vulkan-loader mesa-vulkan-radeon mesa-vaapi mesa-vdpau
```

### Intel GPU

Install the required packages with:

```Shell
sudo xbps-install mesa-dri vulkan-loader mesa-vulkan-intel intel-video-accel
```

If your Intel is generation Coffee Lake or older, then you must set `LIBVA_DRIVER_NAME` to `i965`. Create file `/etc/profile/intelgpu.sh` with contents:

```Shell
export LIBVA_DRIVER_NAME=i965
```

If your Intel is generation Broadwell, and you're experiencing issues, check [this link](https://docs.voidlinux.org/config/graphical-session/graphics-drivers/intel.html#troubleshooting).

### Nvidia GPU

Nvidia users have two choices of drivers; the open source nouveau drivers and the official propietary ones.

- If your card model is 900 or newer, try the propietary drivers.
- If your card is old and no longer supported by Nvidia, try the open source drivers.
- If you're somewhere in the middle, try either.
- And if you experience issues, you can always switch to the other by installing the other's package *and then* uninstalling the current ones.

#### Propietary

The propietary drivers have separate packages based on model:

- For laptops with both an Intel and Nvidia GPU (dual graphics), read [this](https://docs.voidlinux.org/config/graphical-session/graphics-drivers/optimus.html).

- For models 400 and greater, run `sudo xbps-install void-repo-nonfree && sudo xbps-install -S`, then based on your model:

  - For 800 and newer, run `sudo xbps-install nvidia`

  - For 600 or 700, run `sudo xbps-install nvidia470`

  - For 400 or 500, run `sudo xbps-install nvidia390`

- For models older than 400 you must use the nouveau drivers instead.

#### Nouveau

Simply run `sudo xbps-install mesa-dri`

## Installing the desktop

Install `dbus`, `elogind` and `gnome` with:

```Shell
sudo xbps-install dbus elogind gnome
```

This might take a while.

Afterwards, you will have to disable the `acpid` service, as it conflicts with `elogind`:

```Shell
sudo rm /var/service/acpid
```

Finally, enable the `dbus` and `gdm` services and immediately reboot:

```Shell
sudo ln -s /etc/sv/dbus /var/service
sudo ln -s /etc/sv/gdm /var/service
sudo reboot
```

If you take too long to reboot, `gdm` may get started in a broken state and you'll have to forcefully reset the computer.

## Configuring the desktop

If you've done everything right, at this point you should've greeted with the GNOME desktop. Congratulations!

But we're not done yet. There's more to a desktop environment than just the shell, so we have some more setup to go through.

### Installing fonts & the browser

The noto fonts cover the entirety of unicode, which means the majority of apps will render text correctly if you install them:

```Shell
sudo xbps-install noto-fonts-cjk noto-fonts-emoji noto-fonts-ttf noto-fonts-ttf-extra
```

If you haven't done so already, you may want to install `firefox` or `chromium` now and open this page on the browser. If you had, restart the browser to apply the fonts. I'd also recommend setting noto as the system-wide "Document Text" font via the "Tweaks" app.

With a proper browser installed, the rest of this guide should be easier to follow.

### Creating the user directories

By default, the typical directories found in one's home (Documents, Downloads, etc) do not exist. Because some applications *expect* them to exist, I recommend you run the following command:

```Shell
sudo xbps-install xdg-user-dirs-gtk && xdg-user-dirs-update && xdg-user-dirs-gtk-update
```

### Running user scripts after login

The need to run programs as soon as the desktop session starts arises quite often in Void Linux.

You *could* do this by setting up user services, but I prefer to use a startup script due both to its simplicity and because it allows easy access to standard output & error streams.

1.  Create file `~/.config/autostart/loginrc.desktop` with the following contents:

    ```
    [Desktop Entry]
    Type=Application
    Name=loginrc
    Comment=Run commands on login
    Exec=/home/<USER>/.loginrc
    ```

    Replace `<USER>` with your username as appropiate.

2.  Create file `~/.loginrc` with the following contents:

    ```Shell
    #!/bin/bash
    sleep 5
    zenity --info --text="HEY!"
    ```

3.  Make it executable with:

    ```Shell
    chmod +x ~/.loginrc
    ```

If you did everything correctly, on restarting your session a dialog saying "HEY!" should appear after a while.

Various parts of this guide will refer back to this section whenever a user service needs running on startup, so it is worth setting up beforehand.

### Installing an audio and media server

The PipeWire media server has come a long way since its early days; I definitely recommend you set it up as your system's audio server. It'll also allow screensharing the whole desktop.

1.  Begin by installing `pipewire`, `alsa-pipewire` and `rtkit`:

    ```Shell
    sudo xbps-install pipewire alsa-pipewire rtkit
    ```

2.  Create the `/etc/pipewire/pipewire.conf.d` directory:

    ```Shell
    sudo mkdir -p /etc/pipewire/pipewire.conf.d
    ```

3.  Enable the `wireplumber` session manager:

    ```Shell
    sudo ln -s /usr/share/examples/wireplumber/10-wireplumber.conf /etc/pipewire/pipewire.conf.d/
    ```

4.  Enable the PulseAudio interface such that PipeWire can act as its replacement:

    ```Shell
    sudo ln -s /usr/share/examples/pipewire/20-pipewire-pulse.conf /etc/pipewire/pipewire.conf.d/
    ```

5.  Enable the ALSA interface as well:

    ```Shell
    sudo mkdir -p /etc/alsa/conf.d
    sudo ln -s /usr/share/alsa/alsa.conf.d/50-pipewire.conf /etc/alsa/conf.d
    sudo ln -s /usr/share/alsa/alsa.conf.d/99-pipewire-default.conf /etc/alsa/conf.d
    ```

5.  Assuming you followed [Running user scripts after login](#running-user-scripts-after-login), append the following to your `~/.loginrc` script:

    ```Shell
    # Start PipeWire.
    (pipewire &>~/pipewire.log) &
    ```

A session restart will be required. Afterwards, running `wpctl status` should tell you whether PipeWire has been set up correctly or not.

If it is, you may go to **Settings > Sound**. You can select your output and input devices there. Make sure to test them!

For further, more advanced configuration of the audio setup see [Advanced audio configuration](../2.%20More%20Configuration/Advanced%20audio%20configuration.md).

### Switching to network manager

By default Void Linux uses `dhcpcd` to manage your connection, which works but isn't what GNOME expects. In order to allow for network settings to be modified via the user interface, you will have to enable `NetworkManager` instead.

```Shell
sudo rm /var/service/dhcpcd
sudo ln -s /etc/sv/NetworkManager /var/service
```

After doing this, going to **Settings > Network** should display appropiately.

### More GNOME configuration

There's a lot more you can do to improve the GNOME experience! I've only included thus far things that I'd consider basic or essential.

Definitely check out [Further configuring GNOME](../2.%20More%20Configuration/Further%20configuring%20GNOME.md) after going through this page.

### Reboot

This is a good point to restart your session, specially if you did the **Creating the user directories** or **Installing an audio and media server** steps, as those require the GNOME shell to restart to fully integrate.

You can do this by logging out and back in.

## Configuring the system

### Running superuser scripts periodically

This section requires the ability to periodically run scripts as root for us. The simplest way to achieve this is with the `snooze` package:

```Shell
sudo xbps-install snooze
```

Enable its services with:

```Shell
sudo ln -s /etc/sv/snooze-hourly /var/service
sudo ln -s /etc/sv/snooze-daily /var/service
sudo ln -s /etc/sv/snooze-weekly /var/service
sudo ln -s /etc/sv/snooze-monthly /var/service
```

The scripts go in `/etc/cron.<PERIOD>/`; e.g. `/etc/cron.weekly/` to run a script once a week.

Alternatively you can install any conforming cron implementation.

### Keeping the system up-to-date

It is worth setting up some scripts to check and notify about updates, so your system doesn't become quickly outdated.

My setup is pretty simple; one script is set to run once a day and syncs the void repositories. Another one runs when you open a terminal, and it prints a message to notify you when updates are available.

Create the file `/etc/cron.daily/xbps-sync` with contents:

```Shell
#!/bin/sh
xbps-install -S
```

Then make it executable with `sudo chmod +x /etc/cron.daily/xbps-sync`. This will sync the repository daily.

Then, on your `~/.bashrc` file, you can add the following:

```Bash
# Check for updates.
xbps_check_updates=$(xbps-install -nu)
if [[ -n $xbps_check_updates ]]; then
    echo "There are updates available! Run sudo xbps-install -Su to update the system."
    echo "The following packages can be updated:"
    echo "$xbps_check_updates"
    echo ""
    echo "It is recommended to update the system before installing any new packages to avoid breakage."
    echo ""
fi
```

### Automating SSD trimming

You can easily automate SSD [trimming](https://en.wikipedia.org/wiki/Trim_(computing)), which will definitely help in boosting the performance of your disk. To begin, check if your SSD supports trimming in the first place:

```Shell
lsblk --discard
```

If your SSD's `DISC-GRAN` and `DISC-MAX` columns are non-zero, then the trim command is supported.

To set up periodic trimming of your disk, create file `/etc/cron.weekly/fstrim` with contents:

```Shell
#!/bin/sh
fstrim /
```

Then make it executable with `sudo chmod +x /etc/cron.weekly/fstrim`.

Replace `/` with the mountpoint of your SSD partition; leave as is if your root is in an SSD itself.

You can add more `fstrim` commands if you have more than one partition in an SSD, but remember to make sure they're always mounted to guarantee the command doesn't fail.

## Final notes

### Managing runit services

Void Linux uses `runit` to run services. To enable a service, you create a symlink to it in `/var/service`, like so:

```Shell
sudo ln -s <SERVICE_DIR> /var/service
```

Enabling a service automatically starts running it, so you don't need to "start" it.

To disable a service, simply remove the symlink. This will stop the service too:

```Shell
sudo rm /var/service/<SERVICE_NAME>
```

You can manage an enabled service's status via `sv`:

```Shell
sudo sv up <SERVICE_NAME>
sudo sv down <SERVICE_NAME>
sudo sv status <SERVICE_NAME>
sudo sv restart <SERVICE_NAME>
```

### Managing packages with xbps

The package manager in Void Linux is quite powerful. Here is a cheatsheet of commands that may be useful in keeping the installation clean:

- `xbps-query -m`: Gives you a list of every package that you installed explicitly or is part of the base system. Useful to backtrack on what you've done.

- `xbps-remove -R <pkg>`: Removes a package as well all dependencies it pulled that would be orphaned otherwise.

- `xbps-remove -o`: Removes all orphan packages from the system. Orphans are packages that got installed as a dependency of another but are no longer needed, thus they can be removed.

- `xbps-remove -O`: Removes *obsolete* packages from the cache. This gets rid of older versions of packages that are normally only kept for downgrading purposes.

- `xbps-pkgdb -m manual <pkg>`: Switches a package installed as a dependency of another package from auto mode to manual mode. This means that, if the parent package is removed, this package won't be considered an orphan (and thus won't get accidentally removed by `xbps-remove -o` or `xbps-remove -R`).

- If you install the `xtools` package, `xlocate` can be used to find which package provides a particular file in the void repositories:

  ```
  xlocate -S
  xlocate xlocate
  ```

### More guides

You should definitely rummage through the [Void Linux Handbook](https://docs.voidlinux.org/); it is more thorough than these guides and includes information about things I didn't bother to cover.

Beyond that, definitely feel free to look at all of the other guides in the repository if you feel like they're relevant to you.
