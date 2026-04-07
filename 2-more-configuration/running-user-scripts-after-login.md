# Running user scripts after login

The need to run programs as soon as the desktop session starts arises quite often in Void Linux.

As user services are a bit of a hassle to get right, I prefer to use a startup script due both to its simplicity and because it allows easy access to standard output & error streams.

## Setup

1.  Create file `~/.config/autostart/loginrc.desktop` with the following contents:

    ```
    [Desktop Entry]
    Type=Application
    Name=loginrc
    Comment=Run commands on login
    Exec=/home/<USER>/.loginrc
    ```

    Replace `<USER>` with your username as appropiate.

2.  Create file `~/.loginrc` with the following contents:

    ```Shell
    #!/bin/bash
    sleep 5
    notify-send "HEY!"
    ```

3.  Make it executable with:

    ```Shell
    chmod +x ~/.loginrc
    ```

If you did everything correctly, on restarting your session a notification saying "HEY!" should appear after a while.

## Starting user services

To fork a user service from the `~/.loginrc` script, you can just do:

```Shell
(YOUR_SERVICE &>/dev/null) &
```

If you want to leave a log behind to check for errors, just replace `/dev/null` with the name of a file.
