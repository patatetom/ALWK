# Cups

printing support can be added to AWK using the following commands

> see [mdns.md](mdns.md) to add/use Zeroconf with Avahi

```sh
# add edge/community to repositories (hplip)
if ! grep -q '^@edge http://mirrors' /etc/apk/repositories; then
  cat >> /etc/apk/repositories << 'xxxxxxxx'

@edge http://mirrors.ircam.fr/pub/alpine/edge/community
xxxxxxxx
fi
apk update

# hplip actually return error
apk add hplip@edge
#ERROR: unable to select packages:
#  hplip-libs-3.25.8-r2:
#    masked in: @edge
#    satisfies: hplip-3.25.8-r2[hplip-libs=3.25.8-r2]
#               hplip-3.25.8-r2[so:libhpdiscovery.so.0]
#               hplip-3.25.8-r2[so:libhpipp.so.0]
#               hplip-3.25.8-r2[so:libhpmud.so.0]
#  so:libnetsnmp.so.45 (no such package):
#    required by: hplip-libs-3.25.8-r2[so:libnetsnmp.so.45]
#  python3-3.12.12-r0:
#    breaks: hplip-3.25.8-r2[python3~3.14]
#  hplip-3.25.8-r2:
#    masked in: @edge
#    satisfies: world[hplip]

# install cups and filters
apk add cups cups-filters

# add cups to default runlevel
rc-update add cupsd

# start cups immediately
rc-service cupsd start

# let root manage cups
sed -i -E 's/^(lpadmin:x:[^:]+):/\1:root/' /etc/group

# let root redirect local port
sed -i -E 's/^(AllowTcpForwarding) no$/\1 yes\npermitopen="127.0.0.1:*"/' /etc/ssh/sshd_config
rc-service sshd restart
```

> adding a printer and configuring it can then be done directly from AWK (`http://localhost:631/`) or remotely via SSH and browser (`ssh -NC -i …/AWK.key -L 1631:localhost:631 root@AWK-%macaddr%` then `http://localhost:1631/`)
