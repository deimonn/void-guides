# Windows dual-boot

## Installing or re-installing Windows

**If you plan on ever installing/re-installing Windows**, please beware that Windows will try to put its bootloader on the same partition as GRUB if it exists, which is not only undesirable but will create several issues later (e.g. trying to update Windows may fail, even if it managed to install).

You have to force Windows to ignore your disk with Linux on it. I found the simplest way to do this was to just unplug my Linux one during the installation process.

The issue with that, is that your bootloader entry for GRUB in your BIOS may be removed afterwards (depends on the BIOS). See the [Repairing](../2.%20More%20Configuration/GRUB%20customization%20&%20repair.md#repairing) section on the GRUB guide for help in how to restore it.

## Adding a GRUB entry for Windows

You can make GRUB automatically detect other operating systems besides Void Linux, such as Windows, via the OS prober.

Open the `/etc/default/grub` file for editing, and append the following line:

```
GRUB_DISABLE_OS_PROBER=false
```

Then run `sudo update-grub` to update the configuration.

## Switching Windows to UTC

Write the following to a file with any name ending in `.reg` (e.g. `utc.reg`):

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation]
"RealTimeIsUniversal"=dword:00000001
```

Double-click the file and allow it to make changes to the registry, then reboot. Windows will afterwards use UTC time and no longer conflict with other OSes.
