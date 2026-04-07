## PulseAudio

```sh
apk add dbus
rc-update add dbus
rc-service dbus start

apk add sof-firmware

apk add pulseaudio pulseaudio-utils
cat >> /etc/pulse/default.pa <<~~~
########################################
set-sink-volume @DEFAULT_SINK@ 0x10000
load-module module-switch-on-connect
~~~

apk add pavucontrol xbindkeys
cat > /home/browser/.xbindkeysrc <<~~~
"pavucontrol"
    Mod4 + s
"pavucontrol"
    Mod4 + v
~~~

reboot
```
