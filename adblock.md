# AD blocker

simple AD blocker based on [StevenBlack](https://github.com/StevenBlack/hosts) block list

```sh
# install startup script
cat > /etc/local.d/hosts.block.start << 'xxxxxxxx'
#!/bin/sh
# see https://github.com/StevenBlack/hosts and change url as needed
block=/etc/hosts.blocked
url=https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
[ -f "$block" ] || touch -t 190101010000 "$block"
if [ "$( find "$block" -mtime +7 )" ]; then
  wget -q -O "$block" "$url" 2> /dev/null &&
  rc-service -q dnsmasq restart
fi
xxxxxxxx

# make script executable
chmod +x /etc/local.d/hosts.block.start

# configure dnsmasq
if ! grep -q '^addn-hosts=' /etc/dnsmasq.conf; then
  cat >> /etc/dnsmasq.conf << 'xxxxxxxx'

addn-hosts=/etc/hosts.blocked
xxxxxxxx
fi

# start script
/etc/local.d/hosts.block.start
```

> use `chmod -x /etc/local.d/hosts.block.start && rm /etc/hosts.blocked` to stop AD blocker
