# Setting up a swap file

It is possible to set up a swap file if you want swap space but decided to skip on creating a dedicated partition for it.

Using a file has the added benefit of being flexible enough to be deleted and re-created as needed if your swap space requirements change.

1.  To begin, allocate the desired amount of swap space as a file (replace `<SIZE>` on the following command):

    ```Shell
    sudo fallocate -l <SIZE> /swapfile
    ```

    E.g. `8GiB`.

2.  Update its permissions so only the root user can write and read to it:

    ```Shell
    sudo chmod 600 /swapfile
    ```

3.  Format and enable the swap file as you would a partition:

    ```Shell
    sudo mkswap /swapfile
    sudo swapon /swapfile
    ```

    You'll likely want to update your `/etc/fstab` too so the swap comes on automatically when the system starts:

    ```
    /swapfile swap swap defaults 0 0
    ```

You can verify your swap is active by running `sudo swapon --show`.
