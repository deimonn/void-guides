# Final notes

## Automating SSD trimming

You can easily automate SSD [trimming](https://en.wikipedia.org/wiki/Trim_(computing)), which will definitely help in boosting the performance of your disk. To begin, check if your SSD supports trimming in the first place:

```Shell
lsblk --discard
```

If your SSD's `DISC-GRAN` and `DISC-MAX` columns are non-zero, then the trim command is supported.

For the trimming script, you need to be able to run scripts periodically as root; see [Running superuser scripts periodically](../2-extra-setup/running-superuser-scripts-periodically.md). Create file `/etc/cron.weekly/fstrim` with contents:

```Shell
#!/bin/sh
fstrim /
```

Then make it executable with `sudo chmod +x /etc/cron.weekly/fstrim`.

Replace `/` with the mountpoint of your SSD partition; leave as is if your root is in an SSD itself.

You can add more `fstrim` commands if you have more than one partition in an SSD, but remember to make sure they're always mounted if you don't want the command to fail.

## Keeping up-to-date

It is worth setting up some scripts to check and notify about updates, so your system doesn't become quickly outdated.

My setup is relatively simple:

1.  First, you need to be able to run scripts periodically as root; see [Running superuser scripts periodically](../2-extra-setup/running-superuser-scripts-periodically.md). Create the file `/etc/cron.daily/xbps-sync` with contents:

    ```Shell
    #!/bin/sh
    xbps-install -S
    xbps-install -nu > /var/log/xbps-sync
    ```

    Make it executable with `sudo chmod +x /etc/cron.daily/xbps-sync`. This will sync the repository daily and log a list of available updates to `/var/log/xbps-sync`.

2.  Then, on your `~/.bashrc` file, add the following:

    ```Bash
    xbps_check_updates=$(cat /var/log/xbps-sync)

    if [[ -n $xbps_check_updates ]]; then
        echo "There are updates available! Run sudo xbps-install -Su to update the system."
        echo "The following packages can be updated:"
        echo "$xbps_check_updates"
        echo ""
    fi

    function sudo {
        /bin/sudo "$@"
        code=$?

        if [[ $1 == "xbps-install" ]]; then
            xbps-install -nu | /bin/sudo tee /var/log/xbps-sync &>/dev/null
        fi

        return $code
    }
    ```

    The first part just checks whether `/var/log/xbps-sync` contains anything to determine if updates are available. The `sudo` part aliases the command to detect invokations of `xbps-install`, updating `/var/log/xbps-sync` after every call.

## Managing runit services

Void Linux uses `runit` to run services. If you followed the guides preceding this one, you've already interacted with it plenty of times.

- Enabling a service is done by creating a symlink to it in `/var/service`, like so:

  ```Shell
  sudo ln -s /etc/sv/<SERVICE> /var/service
  ```

- Enabling a service automatically starts running it, so you don't need to "start" it. To prevent this from happening for risky services, create file `down` inside its folder before enabling it:

  ```Shell
  sudo touch /etc/sv/<SERVICE>/down
  sudo ln -s /etc/sv/<SERVICE> /var/service
  ```

- To disable a service, simply remove the symlink. This will stop the service too if it is running:

  ```Shell
  sudo rm /var/service/<SERVICE>
  ```

- You can manage an enabled service's status via `sv`:

  ```Shell
  sudo sv status <SERVICE>
  sudo sv start <SERVICE>
  sudo sv stop <SERVICE>
  sudo sv restart <SERVICE>
  ```

## Managing packages with xbps

The package manager in Void Linux is quite powerful. Here is a cheatsheet of commands that may be useful in keeping the installation clean:

- `xbps-query -m`: Gives you a list of every package that you installed explicitly or is part of the base system. Useful to backtrack on what you've done.

- `xbps-remove -R <pkg>`: Removes a package as well all dependencies it pulled that would be orphaned otherwise.

- `xbps-remove -o`: Removes all orphan packages from the system. Orphans are packages that got installed as a dependency of another but are no longer needed, thus they can be removed.

- `xbps-remove -O`: Removes *obsolete* packages from the cache. This gets rid of older versions of packages that are normally only kept for downgrading purposes.

- `xbps-pkgdb -m manual <pkg>`: Switches a package installed as a dependency of another package from auto mode to manual mode. This means that, if the parent package is removed, this package won't be considered an orphan (and thus won't get accidentally removed by `xbps-remove -o` or `xbps-remove -R`).

- If you install the `xtools` package, `xlocate` can be used to find which package provides a particular file or command in the void repositories:

  ```
  xlocate -S
  xlocate <FILE>
  ```

## More guides

You should definitely rummage through the [Void Linux Handbook](https://docs.voidlinux.org/); it is more thorough than these guides and includes information about things I didn't bother to cover.

Beyond that, definitely feel free to look at all of the other guides in the repository if you feel like they're relevant to you.
