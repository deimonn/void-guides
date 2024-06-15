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

3.  You need to start the user-owned `keymapper` service yourself. If you followed [Running user scripts after login](../1.%20Installation/Guide.md#running-user-scripts-after-login), you can just append the following to your `~/.loginrc`:

    ```Bash
    # Start keymapper.
    (keymapper -u &>~/keymapper.log) &
    ```

## Further configuration

You'll want to take a look at the [keymapper repo](https://github.com/houmain/keymapper), which is where all the documentation for keymapper resides.
