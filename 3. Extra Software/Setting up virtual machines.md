# Setting up virtual machines

Virtual machines are an incredibly useful tool both to developers, in order to test their applications on clean systems, and to users, to run applications only available for other operating systems.

To begin, install the `libvirt`, `virt-manager`, `virt-manager-tools` and `qemu` packages:

```Shell
sudo xbps-install libvirt virt-manager virt-manager-tools qemu
```

Enable the `libvirtd`, `virtlockd` and `virtlogd` services:

```Shell
sudo ln -s /etc/sv/libvirtd /var/service
sudo ln -s /etc/sv/virtlockd /var/service
sudo ln -s /etc/sv/virtlogd /var/service
```

Lastly, add yourself to the `libvirt` and `kvm` groups:

```Shell
sudo usermod -a -G libvirt,kvm <USER>
```

## Creating a virtual machine

Open Virtual Machine Manager and press on the computer icon directly below "File" to create a new virtual machine. Set up the parameters to your liking; do consider giving a decent amount of RAM and CPU cores to your OS.

After creating the virtual machine, it will open up and run the ISO image you selected. Just install the OS as you normally would.

### VirtIO for Windows

For Windows guests, you might want to install the VirtIO drivers for improved performance and integration.

1.  First, download the [VirtIO Drivers ISO](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso).

2.  Inside the open VM, press on the lightbulb on the toolbar to open hardware details, then go to "SATA CDROM 1".

3.  In "Source path", press "Browse" and select the drivers ISO. This will load the ISO as a CDROM. Afterwards simply run the "virtio-win-gt-*arch*" and "virtio-win-guest-tools" found inside the CDROM in the Windows VM.

4.  You can now optionally switch to using SCSI for I/O, which vastly improves virtual disk performance.

    1.  Shutdown the virtual machine and go back to hardware details.

    2.  Click "Add Hardware", select "Controller" and add a controller of type "SCSI" and model "VirtIO SCSI".

    3.  Click "Add Hardware" again, this time add "Storage" with bus type "SCSI", any size. We won't really use the disk, we just need it attached.

    4.  Boot the machine again. Check or wait until Windows' Device Manager shows the SCSI disk is detected (should appear as "QEMU QEMU HARDDISK SCSI Disk Device").

    5.  Shutdown the machine, then detach and delete the temporary SCSI disk you added in step 6. Afterwards, detach the SATA disk and reattach it as an SCSI disk.

5.  You can also optionally switch the NIC interface in hardware details to "virtio", similarly increasing performance for network I/O.

    If your connection goes out after making the change, it is most likely a Windows issue. A reboot of the virtual machine usually fixes it.
