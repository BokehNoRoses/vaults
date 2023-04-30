# ISSUE:
- ## Proton Mail Bridge fails to connect to the \`org.freedesktop.secrets\` DBus session and cannot retrieve password
```bash
Apr 30 11:45:33 localhost.localdomain dbus-daemon[3022]: [session uid=1000 pid=3020] Activating service name='org.freedesktop.secrets' requested by ':1.15' (uid=1000 pid=3174 comm="/usr/libexec/xdg-desktop-portal")

Apr 30 11:45:33 localhost.localdomain org.freedesktop.secrets[3365]: discover_other_daemon: 1

Apr 30 11:45:58 localhost.localdomain xdg-desktop-por[3174]: Failed to create secret proxy: Error calling StartServiceByName for org.freedesktop.secrets: Timeout was reached

Apr 30 11:47:33 localhost.localdomain dbus-daemon[3022]: [session uid=1000 pid=3020] Failed to activate service 'org.freedesktop.secrets': timed out (service_start_timeout=120000ms)
```


# SOLUTION:
- ## Force reload the gnome-keyring-daemon
```bash
$ gnome-keyring-daemon -dr
```
