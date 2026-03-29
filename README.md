![Alpine Linux Web Kiosk](logo.webp)
# Alpine Linux Web Kiosk

ALWK is a web kiosk based on [Alpine Linux](https://www.alpinelinux.org/) (3.23) and [Chromium](https://www.chromium.org/Home/) (146.0).



## installation

> prefer installation under EFI/UEFI, especially if it is performed to removable media<br/>
> see also https://docs.alpinelinux.org/ and https://wiki.alpinelinux.org/wiki/Installation for more informations

- boot Alpine Linux ISO image on PC containing media for the future web kiosk
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

- login as `root` with defined password
- modify EFI System Partition (ESP)
> installation is assumed to have been performed under EFI/UEFI<br/>
> if this is not the case, only run `dosfslabel …` command
```
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
```
echo -n | tee /etc/issue > /etc/motd
sed -i 's/^wheel:x:10:root,browser/wheel:x:10:root/' /etc/group
```
- add widest hardware support (ALWK on a USB device) `apk add linux-firmware`
> see https://wiki.alpinelinux.org/wiki/Kernels#Firmware for more informations
- modify GRUB loader (quiet boot)
```
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
```
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
> use content of public key `cat ./kiosk.key.pub` or copy public key to `~/.ssh/authorized_keys` at ALWK
```
mkdir ~/.ssh/
echo "ssh-ed25519 AA … … … Sp comment" > ~/.ssh/authorized_keys
echo "
Port 22
Port 88
Port 389
Port 445
Port 636
" >> /etc/ssh/sshd_config
service sshd restart
```
- install graphics server
```
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
- set up local HTTP server
> this server will serve `index.html` file located in `/www/` folder on first partition (mounted at `/boot/efi/` by ALWK)
```
rc-update add local default
cat > /etc/local.d/python.httpd.start <<\~~~
mkdir -p "${root:=/boot/efi/www}"
[ -f "${index:=$root/index.html}" ] ||
echo '<html style="background-color:#0E5980">
<h1 style="color:#FFFFFF;text-align:center">
<br/><br/><br/>Alpine Linux Web Kiosk</h1></html>' > "$index"
su -p browser -c "python -m http.server -b localhost -d '$root' 8888 &> /dev/null &"
~~~
chmod +x /etc/local.d/python.httpd.start
service local start
cat > /etc/local.d/python.httpd.stop <<\~~~
kill $( ps | awk '$2~/browser/ && $4~/python/ {print $1}' ) &> /dev/null
~~~
chmod +x /etc/local.d/python.httpd.stop
```
- configure system initialization (minimum, silent, and auto-login for `browser` user)
> **no console access with this `/etc/inittab` configuration**<br/>
> uncomment `#tty2::respawn:/sbin/getty 38400 tty2` for console access<br/>
> and/or uncomment `#ttyS0::respawn:/sbin/getty -L 0 ttyS0 vt100` for serial console access (Qemu)<br/>
> and/or access ALWK via secure shell

```
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
```
cat > /home/browser/.profile <<~~~
clear
echo "Starting browser ..."
export LANG=fr
export LC_COLLATE=C
exec startx &>/dev/null
~~~
```
- configure window manager's startup
```
cat > /home/browser/.xinitrc <<~~~
setxkbmap fr
xset -dpms
xset s off
xset s noblank
exec jwm
~~~
```
- set up web browser auto-start
```
cat > /home/browser/.jwmrc <<\~~~
<?xml version="1.0" encoding="UTF-8"?>
<JWM>
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
  grep -E '^http(s)?://' "$urls" ||
  echo http://localhost:8888
  )
rm -rf ~/.config/chromium
jwm -exit
</StartupCommand>
</JWM>
~~~
```


## Chromium configuration

> _TODO_


## web kiosk customization

> _TODO_
