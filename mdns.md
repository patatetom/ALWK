# Zeroconf

```sh
# add edge/testing to repositories (avahi2dns)
echo "@testing http://mirrors.ircam.fr/pub/alpine/edge/testing" >> /etc/apk/repositories
apk update

# add mDNS tools
apk add avahi avahi-tools avahi2dns

# add mDNS tools to default runlevel
rc-update add avahi-daemon
rc-update add avahi2dns

# start immediatly mDNS tools
rc-service avahi-daemon start
rc-service avahi2dns start

# test avahi (mDNS) resolution (return print.local for example)
#avahi-browse --resolve --terminate  _ipp._tcp

# test avahi2dns (mDNS to DNS) resolution
#apk add drill
#drill -p 5354 @127.0.0.1 print.local

# reconfigure dnsmasq (forward to avahi2dns for .local zone)
cat >> /etc/dnsmasq.conf <<~~~
server=/local/127.0.0.1#5354
~~~

# restart dnsmasq
rc-service dnsmasq restart

# test name resolution (DNS and mDNS)
#apk add drill
#drill www.google.fr
#drill printer.local
```
