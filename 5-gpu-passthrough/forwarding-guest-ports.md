# Forwarding guest ports

You may ocassionally find the need to expose VM ports to the internet, as would be the case when hosting some game server that only runs on Windows. The easiest way to get this done is to just use [libvirt-hook-qemu](https://github.com/saschpe/libvirt-hook-qemu).

## Installing the hook

1.  Begin by installing the prerequisites:

    ```Shell
    sudo xbps-install git make
    ```

2.  Clone the repo and change directory into it:

    ```Shell
    git clone https://github.com/saschpe/libvirt-hook-qemu
    cd libvirt-hook-qemu
    ```

3.  Install the hook with:

    ```Shell
    sudo make install
    ```

4.  You will have to restart the libvirt daemon next. Ensure no VM is running, then run:

    ```Shell
    sudo sv restart libvirtd
    ```

## Configuring the ports

The file with the port forwarding configuration for the VM can be found at `/etc/libvirt/hooks/hooks.json`, so you can open it with e.g.:

```Shell
sudo nano /etc/libvirt/hooks/hooks.json
```

Make sure to read the comments, they document the configuration pretty well. The usual steps it takes for configuring the ports are:

1.  Replacing `example-guest-vm-name-please-change` with name of the target VM.

2.  Replacing `private_ip` with the IP shown by the NIC interface of the VM.

3.  Removing or commenting out `source_ip`.

4.  Finally, setting the `port_map` or `port_range` to your desired configuration.

Note that the configuration is only applied when the VM starts, so it is necessary to restart it for changes to take effect.

## Uninstalling the hook

If at any point you wish to uninstall the hook, simply do:

```Shell
sudo make uninstall
```

This will remove it while leaving configuration files intact.
