# Can't boot into Void Linux

If you can't boot into Void Linux, either post-installation or due to some other system modification, you can try to repair the bootloader to see if it helps.

1.  First, you'll have to flash a USB with the Void Linux installation medium, the same way you presumably did to install Void originally; then boot into it.

2.  Once inside the installation medium, mount your actual system to `/mnt`, like:

    ```Shell
    mount /dev/<ROOT_PART> /mnt
    mount /dev/<EFI_PART> /mnt/boot/efi
    ```

    - Replace `<ROOT_PART>` with the partition containing your installation root.
    - Replace `<EFI_PART>` with the partition containing your EFI files.

    You can find the names of the partitions by running `fdisk -l`.

3.  Now chroot into it with:

    ```Shell
    xchroot /mnt
    ```

4.  If you think the partition may have had its contents modified, I recommend removing existing files from it first with:

    ```Shell
    rm -rf /boot/efi/*
    ```

5.  Finally, to re-install GRUB, simply run:

    ```Shell
    grub-install
    ```

    If no errors ocurred, you can try booting again. If booting still fails despite the lack of GRUB errors, you can retry the procedure with the following command instead:

    ```Shell
    grub-install && grub-install --removable
    ```

    This sets up GRUB the way it usually would for removable storage such as USB drives, which may allow it to start. If it works like this, leaving it this way shouldn't cause any issues.
