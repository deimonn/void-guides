# Preparing for the desktop

## Installing the graphics driver

The first thing to do before setting up a desktop is to install the appropiate graphics drivers. The following steps will differ depending on the GPU you're using.

### AMD GPU

Install the required packages with:

```Shell
sudo xbps-install mesa-dri vulkan-loader mesa-vulkan-radeon mesa-vaapi
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

## Setting up session and seat management

Install `dbus` and `elogind`:

```Shell
sudo xbps-install dbus elogind
```

Then enable the `dbus` service with:

```Shell
sudo ln -s /etc/sv/dbus /var/service
```

You will also have to disable the `acpid` service that's enabled by default, as it conflicts with `elogind`:

```Shell
sudo rm /var/service/acpid
```

## Installing the desktop

It is time to download and install the desktop environment. The packages and instructions to follow vary by quite a bit, so this guide branches out:

- For GNOME, see [Installing and configuring GNOME](21-installing-gnome.md).
- For KDE, see [Installing and configuring KDE](22-installing-kde.md).
- For XFCE, see [Installing and configuring XFCE](23-installing-xfce.md).
- For Sway, see [Installing and configuring Sway](24-installing-sway.md).

The environments above are those I've personally tested, not the extent of what is supported by Void Linux.
