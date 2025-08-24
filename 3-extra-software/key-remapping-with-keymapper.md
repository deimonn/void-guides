# Key remapping with `keymapper`

The [keymapper](https://github.com/houmain/keymapper) daemon is a very neat utility that allows you to re-map any keys in your keyboard in all sorts of configurations.

It is similar to [keyd](https://github.com/rvaiya/keyd) (which I also recommend you check out if you use only Linux), but keymapper has the added benefit of working cross-platform in Linux, Windows and macOS, allowing you to share one configuration file across many systems. It also comes packed with several more features.

As keymapper isn't available in the official void repositories, it has to be built from source. I provide a package template in my [void-templates](https://github.com/deimonn/void-templates) repository to simplify the process; follow the usage instructions to install it, replacing `<package-name>` with `keymapper`.

## Installation

1.  To begin, build and install the `keymapper` package via [void-templates](https://github.com/deimonn/void-templates).

2.  Enable the system-wide service with:

    ```Shell
    sudo ln -s /etc/sv/keymapperd /var/service
    ```

3.  Write the following to `~/.config/keymapper.conf`:

    ```
    # Holding [CapsLk], [W] [A] [S] [D] keys become [↑] [←] [↓] [→].
    [modifier="CapsLock"]
    W >> ArrowUp
    A >> ArrowLeft
    S >> ArrowDown
    D >> ArrowRight

    # Tapping [CapsLk] just generates [Esc].
    [default]
    CapsLock >> Escape
    ```

    The above is just example content. You can replace it with your own later.

3.  You need to start the user-owned `keymapper` service yourself. If you followed [Running user scripts after login](../1-installation/guide.md#running-user-scripts-after-login), you can just append the following to your `~/.loginrc`:

    ```Bash
    # Start keymapper.
    (keymapper -u &>~/keymapper.log) &
    ```

## Further configuration

You'll want to take a look at the [keymapper repo](https://github.com/houmain/keymapper), which is where all the documentation for keymapper resides.

## Troubleshooting

### Keymapper and PCI passthrough

If you followed my [GPU passthrough in Void Linux](../5-gpu-passthrough/gpu-passthrough-in-void-linux.md), you may have run into an issue with keymapper intercepting input devices.

It is possible to keep keymapper running on the host and use its output as keyboard in the guest, but it requires a little bit of hacking.

1.  Firstly, for this hack keymapper should only intercept *one* device (presumably your keyboard). You can ensure this is the case by adding the following directives to the top of your `keymapper.conf`:

    ```
    @skip-device-id /.*/
    @grab-device-id /<YOUR_DEVICE_NAME>/
    ```

    Replace `<YOUR_DEVICE_NAME>` as appropiate; you can figure it out by looking at `/dev/input/by-id` contents and `cat`'ing devices with keymapper disabled as explained in [Creating the virtual machine](../5-gpu-passthrough/gpu-passthrough-in-void-linux.md#creating-the-virtual-machine). You don't need to write out the full name, just the identifying part is good enough.

2.  You should be able to use `sudo dmesg | grep "keymapper"` to figure out what ID the keymapper output got assigned. Look for a message like:

    ```
    elogind-daemon[1028]: Watching system buttons on /dev/input/event20 (keymapper)
    ```

    In my case, `/dev/input/event20` contains my keymapper output.

3.  To avoid having your virtual machine pointing to a device name that could change in the future (the `20` just means it was the 21st device to be added), you can use the following script:

    ```Bash
    #!/bin/bash

    device=$(dmesg | grep -m1 -o -E '/dev/input/event[0-9]+ \(keymapper\)')
    device=${device::-12}

    mkdir -p /tmp/dev/input && ln -s "$device" /tmp/dev/input/keymapper
    ```

    This will create `/tmp/dev/input/keymapper` as a symlink to the actual device, so your VM can point to that instead of `/dev/input/event<N>`.

    Note that this script must be run as root everytime you boot your computer. How you choose to do that is up to you (`/etc/rc.local` is unsuitable, as this must run *after* the keymapper service starts).
