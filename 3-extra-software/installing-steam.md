# Installing Steam

Installing Steam natively on Void Linux can be kind of a hassle, so this guide will attempt to summarize the necessary steps.

Of course, if you can't be bothered, you can always just install [Steam from the flathub](https://flathub.org/apps/com.valvesoftware.Steam) - it just won't run as smoothly.

## Enabling required repositories

You'll need to enable the 32-bit and propietary code repos before continuing. You can do so with the following two commands:

```Shell
sudo xbps-install void-repo-multilib void-repo-nonfree
sudo xbps-install -S
```

## Installing a 32-bit graphics driver

This is just like the [Installing the graphics driver](../1-installation/guide.md#installing-the-graphics-driver) section of the installation guide, but this time we're dealing with 32-bit package names.

### AMD GPU

Install the required packages with:

```Shell
sudo xbps-install mesa-dri-32bit vulkan-loader-32bit mesa-vulkan-radeon-32bit mesa-vaapi-32bit mesa-vdpau-32bit
```

### Intel GPU

Install the required packages with:

```Shell
sudo xbps-install mesa-dri-32bit vulkan-loader-32bit mesa-vulkan-intel-32bit
```

### Nvidia GPU

#### Propietary

- For models 400 and greater, run `sudo xbps-install void-repo-multilib-nonfree && sudo xbps-install -S`, then based on your model:

  - For 800 and newer, run `sudo xbps-install nvidia-libs-32bit`

  - For 600 or 700, run `sudo xbps-install nvidia470-libs-32bit`

  - For 400 or 500, run `sudo xbps-install nvidia390-libs-32bit`

- For models older than 400 you must use the nouveau drivers instead.

#### Nouveau

Simply run `sudo xbps-install mesa-dri-32bit`

## Installing other dependencies

- The following 32-bit libraries are ubiquotous (and likely already installed if using open source drivers):

  ```Shell
  sudo xbps-install libgcc-32bit libstdc++-32bit libdrm-32bit libglvnd-32bit
  ```

- If you need controller support, install `steam-udev-rules`:

  ```Shell
  sudo xbps-install steam-udev-rules
  ```

- For 32-bit games, you'll need 32-bit PipeWire and related packages or you won't hear any audio:

  ```Shell
  sudo xbps-install pipewire-32bit alsa-pipewire-32bit libspa-alsa-32bit libspa-audioconvert-32bit libspa-audiomixer-32bit libspa-control-32bit libspa-v4l2-32bit
  ```

## Installing Steam

You can finally install Steam with:

```Shell
sudo xbps-install steam
```

Open it and log in to see if it worked. You should also try some Linux-native games to see if there's any issues.

If you get any errors, feel free to open an issue and we can try troubleshooting it - and potentially help others who run into such issues too by writing about it in this guide.
