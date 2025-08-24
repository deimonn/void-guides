# Bluetooth device keeps erroring

You're likely missing the corresponding packages if you never installed them. Install them with:

```Shell
sudo xbps-install bluez libspa-bluetooth
```

Then add yourself to the `bluetooth` group and enable the service with:

```Shell
sudo usermod -a -G bluetooth <USER>
sudo ln -s /etc/sv/bluetoothd /var/service
```

Replace `<USER>` with your user name.

Note that `libspa-bluetooth` is for PipeWire (which is the media server that I explained how to set up in the [Installation Guide](../1-installation/guide.md)). If you're using a different audio server, see the [Void Linux Handbook](https://docs.voidlinux.org/config/bluetooth.html).
