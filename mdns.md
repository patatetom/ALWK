# Zeroconf

```sh
# add edge/testing to repositories (avahi2dns)
if ! grep -q '^@testing http://mirrors' /etc/apk/repositories; then
  cat >> /etc/apk/repositories << 'xxxxxxxx'

@testing http://mirrors.ircam.fr/pub/alpine/edge/testing
xxxxxxxx
fi
apk update

# add mDNS tools
apk add avahi avahi-tools avahi2dns@testing

# add mDNS tools to default runlevel
rc-update add avahi-daemon
rc-update add avahi2dns

# start immediately mDNS tools
rc-service avahi-daemon start
rc-service avahi2dns start

# test avahi (mDNS) resolution (return print.local for example)
#avahi-browse -r -t _ipp._tcp

# test avahi2dns (mDNS to DNS) resolution
#apk add drill
#drill -p 5354 @127.0.0.1 print.local

# reconfigure dnsmasq (forward to avahi2dns for .local zone)
if ! grep -q '^server=/local/' /etc/dnsmasq.conf; then
  cat >> /etc/dnsmasq.conf << 'xxxxxxxx'

server=/local/127.0.0.1#5354
xxxxxxxx
fi

# add search directive (expand hosts / optional)
#chattr -i /etc/resolv.conf
#cat >> /etc/resolv.conf << 'xxxxxxxx'
#options ndots:1
#search local
#xxxxxxxx
#chattr +i /etc/resolv.conf

# restart dnsmasq
rc-service dnsmasq restart

# test name resolution (DNS and mDNS)
#apk add drill
#drill www.google.fr
#drill print.local
##drill print # if search directive added to /etc/resolv.conf
```
