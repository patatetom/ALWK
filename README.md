![Alpine Linux Web Kiosk](logo.webp)
# Alpine Linux Web Kiosk

ALWK is a web kiosk based on [Alpine Linux](https://www.alpinelinux.org/) (3.23) and [Chromium](https://www.chromium.org/Home/) (146.0).


## installation

- boot ISO Alpine Linux image on the future web kiosk
- log in as `root` without a password (empty password)
- start installation with `KERNELOPTS="quiet mitigations=off" ROOTFS=btrfs setup-alpine`
  - keymap `fr`
  - keyboard layout `fr`
  - hostname `kiosk`
  - initialize interface `eth0`
  - ip address `dhcp`
  - root password `**********`
  - timezone `Europe`
  - sub timezone `Paris`
  - proxy `none`
  - ntp client `busybox`
  - apk mirror `c` (community) `22` (mirros.ircam.fr)
  - user `browser`
  - full name `Browser`
  - password `chromium`
  - retype password `chromium`
  - ssh/url `none`
  - ssh server `openssh`
  - allow root ssh `prohibit-password`
  - ssh key or URL `none`
  - disk `sda`
  - type `sys`
  - erase `y`
- update system `apk update && apk upgrade`
- reboot `reboot`
