## PulseAudio

```sh
apk add dbus
rc-update add dbus
rc-service dbus start

apk add pulseaudio pulseaudio-utils sof-firmware
cat >> /etc/pulse/default.pa <<~~~
########################################
set-sink-volume @DEFAULT_SINK@ 0x10000
load-module module-switch-on-connect
~~~

reboot
```
