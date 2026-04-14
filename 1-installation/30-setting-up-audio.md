# Installing an audio and media server

Audio (and also screenshare on Wayland) are possible on Void via PipeWire. There are other audio servers available, but PipeWire kills multiple birds with one stone and otherwise works just fine.

## Setup

1.  Begin by installing `pipewire` and `alsa-pipewire`:

    ```Shell
    sudo xbps-install pipewire alsa-pipewire
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

6.  You can finally start PipeWire with:

    ```Shell
    pipewire
    ```

Running `wpctl status` should tell you whether PipeWire was set up correctly or not.

If it is, you may want to go check your sound settings. You can select your output and input devices there. Make sure to test them!

## Autostart

You'll likely want PipeWire to start automatically with the session instead of having to start it manually.

If you're on Sway you can do this with an `exec`; on other environments, see [Running user scripts after login](../2-extra-setup/running-user-scripts-after-login.md).

## More audio

For further, more advanced configuration of the audio setup (such as microphone or output post-processing), see [Advanced audio configuration](../2-extra-setup/advanced-audio-configuration.md).

## Next steps

At this point, your desktop should be more or less complete. There are only a couple of post-install things left to do; see [Final notes](99-final-notes.md).
