---
uid: admin-troubleshoot
title: Troubleshooting
---

# Troubleshooting

This page outlines some solutions to common issues beginners may encounter when running a Jellyfin server.

## Connection Issues

If you can access the web interface over HTTP but not HTTPS, then you likely have an error with the certificate. Jellyfin uses a PFX file to handle HTTPS traffic. If you created the file with a password, then you will have to enter that value on the **Networking** page in the settings.

If you can access the server locally but not outside of your LAN, then you likely have an issue with the router configuration. Check the port forwarding settings on your router to ensure the server is visible from outside your local network. You can also enable the "Enable automatic port mapping" option on the  **Networking** page of the server settings to have the server attempt to configure port forwarding on the router automatically if your router supports it.

The easiest way to check for issues is by checking the logs, which can be accessed through the settings on the web client or in the log directory on your server. If there are no logs at all relating to web traffic, even over a LAN connection, then the server hasn't been reached at all yet. This would indicate either an incorrect address or an issue somewhere else on the network.

## Debug Logging

To enable debug (much more verbose) logging, it is currently required to manually edit config files since no options exist yet on the frontend. Go to the Jellyfin configuration directory, find the `logging.json` file, and change the minimum level to debug as seen below.

```json
{
    "Serilog": {
        "MinimumLevel": {
            "Default": "Debug"
        }
    }
}
```

Jellyfin 10.4.1 and above will automatically reload the new configuration. The debug messages show up in the log with the `DBG` tag.

## Real Time Monitoring

This will let Jellyfin automatically update libraries when files are added or modified. Unfortunately this feature is only supported on certain filesystems.

For Linux systems, this is performed by [inotify](https://en.wikipedia.org/wiki/Inotify). NFS and rclone do not support inotify, but support can be provided by using a union file system such as [mergerfs](https://github.com/trapexit/mergerfs) with your networked file systems.

Due to the library size, you can receive an error such as this:

```log
[2019-12-31 09:11:36.652 -05:00]  [ERR] Error in Directory watcher for: "/media/movies"  System.IO.IOException: The configured user limit (8192) on the number of  inotify watches has been reached.
```

If you are running Debian, RedHat, or another similar Linux distribution, run the following in a terminal:

```sh
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
```

If you are running ArchLinux, run the following command instead:

```sh
echo fs.inotify.max_user_watches=524288 | sudo tee /etc/sysctl.d/40-max-user-watches.conf && sudo sysctl --system
```

Then paste it in your terminal and press on enter to run it. For Docker, this needs to be done on the host, not the container. See [here](https://github.com/guard/listen/wiki/Increasing-the-amount-of-inotify-watchers) for more information.
