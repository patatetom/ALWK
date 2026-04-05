# Cups

> 

```sh
# add edge/community to repositories
echo "@edge http://mirrors.ircam.fr/pub/alpine/edge/community" >> /etc/apk/repositories
apk update

# hplip actually return error
apk add hplip
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

apk add cups cups-filters
rc-update add cupsd
rc-service cupsd start
sed -i 's/^lpadmin:x:105:$/lpadmin:x:105:root/' /etc/group
```
