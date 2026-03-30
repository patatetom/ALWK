![Alpine Linux Web Kiosk](logo.webp)
# Alpine Linux Web Kiosk

ALWK is a web kiosk based on [Alpine Linux](https://www.alpinelinux.org/) (3.23) and [Chromium](https://www.chromium.org/Home/) (146.0).



## installation

> changes may be necessary (keyboard `fr`, disk `sda`, …) : **be sure to select correct disk if there are multiple available**<br/>
> **Secure Boot must be disabled on EFI/UEFI platforms**<br/>
> prefer installation under EFI/UEFI, especially if it is performed to removable media<br/>
> see also https://docs.alpinelinux.org/ and https://wiki.alpinelinux.org/wiki/Installation for more informations

- boot Alpine Linux ISO image on PC containing media storage for the future web kiosk
- login as `root` without a password (empty password)
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
  - ssh key or URL `none`
  - ssh server `openssh`
  - allow root ssh `prohibit-password`
  - ssh key or URL `none`
  - disk `sda`
  - type `sys`
  - erase `y`
- update system `apk update && apk upgrade`
- reboot `reboot`



## configuration

> changes may be necessary (SSH public key, disk `sda`, …) : **be sure to select correct disk if there are multiple available**

- boot ALWK from media storage selected during installation
- login as `root` with defined password
- modify EFI System Partition (ESP)
> installation is assumed to have been performed under EFI/UEFI<br/>
> if this is not case, only run `dosfslabel …` command
```sh
apk add gptfdisk
gdisk /dev/sda <<~~~
t
1
0700
c
1
ALWK
c
2
SWAP
c
3
ROOT
w
y
~~~
dosfslabel /dev/sda1 ALWK &> /dev/null
apk add mtools
echo 'drive e: file="/dev/sda1"' > /etc/mtools.conf
mattrib +h +s e:/efi 2> /dev/null
```
- make few minor changes
```sh
echo -n | tee /etc/issue > /etc/motd
sed -i 's/^wheel:x:10:root,browser/wheel:x:10:root/' /etc/group
```
- add widest hardware support (ALWK on a USB device) `apk add linux-firmware`
> see https://wiki.alpinelinux.org/wiki/Kernels#Firmware for more informations
- modify GRUB loader (quiet boot)
```sh
chmod -x /etc/grub.d/*
chmod +x /etc/grub.d/00_header
chmod +x /etc/grub.d/10_linux
echo "
GRUB_TIMEOUT_STYLE=hidden
GRUB_DISABLE_OS_PROBER=true
" >> /etc/default/grub
grub-mkconfig |
sed -e 's/Loading Linux lts/Loading kiosk/' \
    -e '/Loading initial ramdisk/d' > /boot/grub/grub.cfg
```
- make dynamic hostname (MAC based / multiple kiosks on same LAN)
```sh
cat > /etc/init.d/machostname <<\~~~
#!/sbin/openrc-run
depend()
{
	after hostname
}
start()
{
	hostname kiosk-$(
	  ip link show dev eth0 |
	  awk '/link\/ether/{print$2}' |
	  tr -d ':'
	)
}
~~~
chmod +x /etc/init.d/machostname
rc-update add machostname boot
service machostname start
```
- configure remote access (remote administration)
> generate an SSH key pair from administration workstation `ssh-keygen -t ed25519 -C comment -f ./kiosk.key`<br/>
> use content of public key `cat ./kiosk.key.pub` or copy public key to `~/.ssh/authorized_keys` at ALWK<br/>
> use alternative ports (`Port 88`, `Port 389`, `Port 445`, `Port 636`, …) if kiosk is operating in filtered environment
```sh
mkdir ~/.ssh/
echo "ssh-ed25519 AA … … … Sp comment" > ~/.ssh/authorized_keys
```
- install graphics server
```sh
setup-xorg-base
cat > /etc/X11/xorg.conf <<~~~
Section "ServerFlags"
	 Option "DontVTSwitch" "true"
	 Option "DontZap" "true"
EndSection
~~~
```
- add graphics server extensions `apk add xf86-input-synaptics setxkbmap font-dejavu ttf-freefont`
- add simple window manager `apk add jwm`
- add Chromium browser `apk add chromium chromium-lang`
- setup default web page
```sh
rc-update add local default
cat > /etc/local.d/default.web.page.start <<\~~~
mkdir -p "${root:=/boot/efi/www}"
[ -f "${index:=$root/index.html}" ] ||
echo '<html style="background-color:#0E5980">
<h1 style="color:#FFFFFF;text-align:center">
<br/><br/><br/>Alpine Linux Web Kiosk</h1></html>' > "$index"
~~~
chmod +x /etc/local.d/default.web.page.start
service local start
```
- configure system initialization (minimum, silent, and auto-login for `browser` user)
> **no console access with this `/etc/inittab` configuration**<br/>
> uncomment `#tty2::respawn:/sbin/getty 38400 tty2` for console access<br/>
> and/or uncomment `#ttyS0::respawn:/sbin/getty -L 0 ttyS0 vt100` for serial console access (Qemu)<br/>
> and/or access ALWK via secure shell
```sh
cat > /etc/inittab <<~~~
::sysinit:clear
::sysinit:echo Starting kiosk ...
::sysinit:/sbin/openrc sysinit   -q > /dev/null
::sysinit:/sbin/openrc boot      -q > /dev/null
::wait:/sbin/openrc default      -q > /dev/null
tty1::respawn:/bin/login -f browser
#tty2::respawn:/sbin/getty 38400 tty2
#ttyS0::respawn:/sbin/getty -L 0 ttyS0 vt100
::ctrlaltdel:clear
::ctrlaltdel:/sbin/reboot        -q > /dev/null
::shutdown:clear
::shutdown:/sbin/openrc shutdown -q > /dev/null
~~~
```
- configure login for `browser` user (graphics server automatic startup at login)
```sh
cat > /home/browser/.profile <<~~~
clear
echo "Starting browser ..."
export LANG=fr
export LC_COLLATE=C
rm -f \
  ~/.Xauthority \
  ~/.serverauth.* \
  ~/.cache/chromium \
  ~/.config/chromium \
  &> /dev/null
exec startx &>/dev/null
~~~
```
- configure window manager's startup
```sh
cat > /home/browser/.xinitrc <<~~~
setxkbmap fr
xset -dpms
xset s off
xset s noblank
exec jwm
~~~
```
- set up web browser auto-start
```sh
cat > /home/browser/.jwmrc <<\~~~
<?xml version="1.0" encoding="UTF-8"?>
<JWM>
<Key mask="A" key="Tab">nextstacked</Key>
<Key mask="AS" key="Tab">prevstacked</Key>
<StartupCommand>
chromium \
  --kiosk \
  --no-first-run \
  --autoplay-policy=no-user-gesture-required \
  --disable-infobars \
  --disable-session-crashed-bubble \
  --disable-restore-session-state \
  --disable-component-update \
  --check-for-update-interval=315360000 \
  --disable-pinch \
  --disable-features=TranslateUI \
  --disable-extensions \
  --disable-background-networking \
  --disable-sync \
  --disable-default-apps \
  --process-per-site \
  --disk-cache-size=0 \
  --password-store=basic \
  --noerrdialogs \
  $(
  [ -f "${urls:=/boot/efi/urls.txt}" ] &&
  grep -E '^(file|http(s)?)://' "$urls" ||
  echo file:///boot/efi/www/index.html
  )
jwm -exit
</StartupCommand>
</JWM>
~~~
```



## Chromium configuration

- disable `file://` scheme (except for default web page)
```sh
mkdir -p /etc/chromium/policies/managed/
cat > /etc/chromium/policies/managed/block_file.json <<~~~
{
  "URLAllowlist": ["file:///boot/efi/www/"],
  "URLBlocklist": ["file://"]
}
~~~
```



## web kiosk customization

### `%part1%/urls.txt`

> `/boot/efi/urls.txt` on ALWK

`urls.txt` file, located in root directory of first partition, tells browser which web page(s) to open and can be easily installed and configured

```ini
# keep default updated web page for user informations (kiosk guide)
file:///boot/efi/www/index.html
# DuckDuckGo
https://duckduckgo.com/
# ALWK ;-)
https://github.com/patatetom/ALWK/
```

### `%part1%/www/index.html`

> `/boot/efi/www/index.html` on ALWK

`index.html` file, stored in `/www/` folder located in root of first partition, is default web page opened by browser and can be easily updated and expanded



## screencast

[![boot screencast](boot.webp)](boot.large.webp)
