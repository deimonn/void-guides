# Running superuser scripts periodically

Running user-provided scripts as root periodically is a common occurrence in Linux. This is possible by just installing and configuring any conforming cron implementation.

If you do not need the flexibility or don't want to go through the effort, however, the simplest way to achieve the same thing is with the `snooze` package:

```Shell
sudo xbps-install snooze
```

Enable its services with:

```Shell
sudo ln -s /etc/sv/snooze-hourly /var/service
sudo ln -s /etc/sv/snooze-daily /var/service
sudo ln -s /etc/sv/snooze-weekly /var/service
sudo ln -s /etc/sv/snooze-monthly /var/service
```

The scripts go in `/etc/cron.<PERIOD>/`; e.g. `/etc/cron.weekly/` to run a script once a week.
