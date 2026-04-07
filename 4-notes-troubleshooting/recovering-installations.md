# Recovering bricked installations

If your Void Linux boots but gets stuck at some point afterwards, perhaps due to bad services or configuration, it is still possible to recover from it.

1.  First, you'll have to flash a USB with the Void Linux installation medium, the same way you presumably did to install Void originally; then boot into it.

2.  Once inside the installation medium, mount your system's root partition to `/mnt`, like:

    ```Shell
    mount /dev/<ROOT_PART> /mnt
    ```

    You can find the name of the partition by running `fdisk -l`.

3.  Now chroot into it with:

    ```Shell
    xchroot /mnt
    ```

    You're now back in your system, but without any services running or bad configuration to get in your way.

4.  At this point, you can just go ahead and revert whichever configuration changes broke the install in the first place.

    You can also remove packages via `xbps-remove` as you would normally.

    If you need to disable services, keep in mind that `/var/service` is not available on chroot; remove them from `/etc/runit/runsvdir/default` instead.
