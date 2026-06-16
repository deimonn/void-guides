# Installing apps not in the repos

While native is usually the best for most apps (provided they're set up correctly), many applications are simply not available in the official void repositories and thus won't be available to `xbps-install`.

## AppImage

The best alternative to native are applications in AppImage format. In my experience, these usually 'just work' without requiring much setup; just download the app and run it.

However, AppImage applications won't show up in launchers by default, and file types corresponding to the application won't be recognized either. Integration isn't that difficult though:

1.  Extract the AppImage's contents:

    ```Shell
    ./<APPIMAGE> --appimage-extract
    ```

    They will be found in a folder called `squashfs-root`.

2.  For launcher integration, you need to copy and adjust the desktop file, which ends in `.desktop`.

    Once you've found it, simply copy it to `~/.local/share/applications`, then in every occurrence of the `Exec` field, replace the binary name with the full path to the AppImage file.

3.  If you also want the app icon to show up properly, look for a `.png` or `.svg` file in the same directory. Copy the file (not the symlink) to some folder somewhere, then simply modify the `Icon` field of the desktop file from the previous step to point to the copy.

4.  To register or associate relevant MIME types with the application, look inside the extracted `usr/share/mime` folder, then copy its contents to `~/.local/share/mime/packages`.

    Refresh the MIME type database with:

    ```Shell
    update-mime-database ~/.local/share/mime
    ```

5.  Lastly, if some applications keep throwing `EPIPE` errors when trying to launch them, you will have to edit the `Exec` fields of the desktop file to set the standard output and error streams. Something like so:

    ```
    Exec=/bin/bash -c '<THE_ACTUAL_COMMAND>' &>/dev/null
    ```

## Flatpak

Another choice is Flatpak. Flatpak sandboxes the application, which while it helps in keeping the installation clean, also tends to restrict applications quite heavily.

Some applications run extremely well (with [Bitwig Studio](https://flathub.org/apps/com.bitwig.BitwigStudio) you can't even tell the difference with its native counterpart). Others are either very buggy or so severely handicapped its a hassle to use them (e.g. [Visual Studio Code](https://flathub.org/apps/com.visualstudio.code)).

Still, Flatpak is rather simple in terms of usage and has a wide array of available applications. If you wish to give it a chance, simply run the following two commands:

```Shell
sudo xbps-install flatpak
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

Note that you may have to restart the session for Flatpak to start working correctly.

The [flathub](https://flathub.org/) is where you'll find applications to install. For example, to install [Vesktop](https://flathub.org/apps/dev.vencord.Vesktop) (a much better maintained alternative to Discord on linux):

```Shell
flatpak install flathub dev.vencord.Vesktop
```

## Hacking it

Lastly, some applications may provide 'portable' versions, which usually just means a compressed archive that can be extracted and the binary run from directly. These are rather inconvenient though, as they usually do not provide desktop entries or MIME types.

If you're already willing to hack it with raw binaries anyway, I suggest you get the `.deb` package equivalent instead. These can just be extracted, with the application files available under `data/`.

Provided the dependencies are satisfied, this also allows you to run the binary directly as well, but now you may also have desktop and MIME files available.

You can copy the desktop entries from `data/usr/share/applications` to `~/.local/share/applications`, but note that all of the entries will need their `Exec` field updated to point to the correct paths.

For MIME types, they can similarly be copied from `data/usr/share/mime` to `~/.local/share/mime`.
