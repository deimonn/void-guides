# Installing and configuring KDE

## Getting the desktop

### Installing packages

Install the X server and KDE metapackages:

```Shell
sudo xbps-install xorg kde5 kde5-baseapps
```

### Enabling the display manager

Enable the `sddm` service without automatic start:

```Shell
sudo touch /etc/sv/sddm/down
sudo ln -s /etc/sv/sddm /var/service
```

Then reboot. After logging back in, you can test that `sddm` works correctly with:

```Shell
sudo sv start sddm
```

## On the desktop

You should've loaded into KDE at this point. If you did, congratulations! You can go ahead and make `sddm` start automatically if so:

```Shell
sudo rm /var/service/sddm/down
```

KDE works relatively well out of the box and is rather customizable through settings alone. There are plenty of internet resources on ricing it up if you're interested.

## Next steps

You'll probably want to set up audio and screenshare next; [Installing an audio and media server](30-setting-up-audio.md) explains how to do that.
