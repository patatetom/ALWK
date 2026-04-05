# Zeroconf

```sh
# add edge/testing to repositories (avahi2dns)
echo "@testing http://mirrors.ircam.fr/pub/alpine/edge/testing" >> /etc/apk/repositories
apk update

# add mdns tools
apk add avahi avahi-tools avahi2dns

# add mdns tools to default runlevel
rc-update add avahi-daemon
rc-update add avahi2dns

# start immediatly mdns tools
rc-service avahi-daemon start
rc-service avahi2dns start

# test avahi (mdsn) resolution (return print.local for example)
#avahi-browse --resolve --terminate  _ipp._tcp

# test avahi2dns resolution
#apk add drill
#drill -p 5354 @127.0.0.1 print.local

# add dns tools
apk add openresolv unbound

# add dns tools to default runlevel
rc-update add unbound

# configure name resolution
cat > /etc/resolvconf.conf <<~~~
name_servers=127.0.0.1
unbound_conf=/etc/unbound/unbound.conf.d/resolvconf.conf
~~~

# configure forward to avahi2dns for .local zone
cat > /etc/unbound/unbound.conf.d/avahi-local.conf <<~~~
forward-zone:
  name: "local"
  forward-addr: 127.0.0.1@5354
server:
  do-not-query-localhost: no
  domain-insecure: "local"
~~~

# start immediatly dns tools
rc-service unbound start

# link udhcpc (DHCP client) with dns tools
mkdir -p /etc/udhcpc/post-bound
cat > /etc/udhcpc/post-bound/resolvconf <<~~~
#!/bin/sh
echo nameserver 127.0.0.1 > /etc/resolv.conf
~~~
chmod +x /etc/udhcpc/post-bound/resolvconf

# restart networking service
service networking restart

# test name resolution
#apk add drill
#drill www.google.fr
#drill printer.local
```
