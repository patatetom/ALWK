## PulseAudio

sound support can be added to AWK using the following commands

```sh
# add dbus
apk add dbus
rc-update add dbus
rc-service dbus start

# add sound open firmware
apk add sof-firmware

# add pulseaudio, set default volume to 100% on all output
# and switch on new device
apk add pulseaudio pulseaudio-utils
cat >> /etc/pulse/default.pa <<~~~
########################################
set-sink-volume @DEFAULT_SINK@ 0x10000
load-module module-switch-on-connect
~~~

# add output/volume control and bind [Window]-[S] (sound)
# or [Window]-[V] (volume) to pavucontrol
apk add pavucontrol xbindkeys
cat > /home/browser/.xbindkeysrc <<~~~
"pavucontrol"
    Mod4 + s
"pavucontrol"
    Mod4 + v
~~~

reboot
```
