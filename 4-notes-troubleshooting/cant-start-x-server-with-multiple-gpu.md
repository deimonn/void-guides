# Can't start the X server with multiple GPUs

If you have two or more GPUs and the X server has trouble starting, you may need to specify which should be used by X for rendering. You can do this by writing the following to a file, e.g. `~/xorg.conf`:

```
Section "Device"
    Identifier "Device0"
    Driver "<driver>"
    BusID "PCI:<device>"
EndSection
```

- Replace `<driver>` with `amdgpu`, `nouveau` or `nvidia` based on the driver you installed.

- Replace `<device>` with the corresponding identifier for the PCI device. You can list all devices with `lspci`, but note that `lspci` displays the numbers in hexadecimal and the last number separated with a dot, like `00:1f.5`; the numbers must be translated to decimal and the dot replaced with a colon, like `PCI:0:31:5`.

Then, you must tell the display manager to use the configuration. For example, LightDM has a `xserver-config` option in `/etc/lightdm/config` that can be set to point to the file.
