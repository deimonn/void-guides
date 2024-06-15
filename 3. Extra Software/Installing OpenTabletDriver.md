# Installing OpenTabletDriver

If you have a PenTablet, you might have realized that driver support for it on Linux is likely to be lackluster at best, or completely missing at worst.

Fortunately, there's an alternative - [OpenTabletDriver](https://github.com/OpenTabletDriver/OpenTabletDriver) - that works pretty well on Linux for a wide range of tablets.

The downside is that it is not packaged by Void Linux at the moment, so the installation instructions aren't as simple as invoking `xbps-install`.

## Installing dependencies

OpenTabletDriver requires .NET 6 to run.

I maintain templates for .NET 5 through 8 in my [void-templates](https://github.com/deimonn/void-templates) repository; simply follow the usage instructions once per package, replacing `<package-name>` as appropiate. You'll need `dotnet` and then `dotnet6`.

Note that after installing .NET for the first time, you will have to restart your computer for it become available system-wide.

## Installing the package

I maintain a template for OpenTabletDriver in my [void-templates](https://github.com/deimonn/void-templates) repository; simply follow the usage instructions, replacing `<package-name>` with `OpenTabletDriver`.

## Setting up the daemon

The template comes with a daemon that must be run by the user, similarly to PipeWire. If you did the [Running user scripts after login](../1.%20Installation/Guide.md#running-user-scripts-after-login) step, simply append the following to your `~/.loginrc` file:

```Bash
# Start the OpenTabletDriver daemon.
(otd-daemon &>~/otd-daemon.log) &
```

The daemon also requires permission to access `/dev/uinput` and `/dev/hidraw*`, which you don't have by default, so you'll have to do some setup:

1.  To begin, add yourself to the `input` and `plugdev` groups (replace `<USER>` as appropiate):

    ```Bash
    sudo usermod -a -G input,plugdev <USER>
    ```

2.  This does not include `/dev/hidraw*` devices by default, but we can make it so by creating the file `/etc/udev/rules.d/99-hidraw-permissions.rules` with the following contents:

    ```
    KERNEL=="hidraw*", SUBSYSTEM=="hidraw", MODE="0660", GROUP="plugdev"
    ```

3.  You'll have to restart your computer for the changes to enter into effect.
