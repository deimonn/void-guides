# NetworkManager not running

If you're using GNOME, you might have seen an error trying to access network settings - and if you're using KDE, you might have noticed there's no icon indicating whether you have an internet connection. That's because both expect you to be running `NetworkManager`, whereas by default Void Linux uses `dhcpcd`.

If this doesn't bother you, `dhcpcd` seems to have less issues than `NetworkManager` and is perfectly capable of serving you internet.

However, if you'd still like for network settings to be modifiable via the user interface, you may elect to replace it. Simply do:

```Shell
sudo xbps-install NetworkManager
sudo rm /var/service/dhcpcd
sudo ln -s /etc/sv/NetworkManager /var/service
```

After doing this, opening network settings on GNOME should display appropiately and KDE should display the appropiate icon.
