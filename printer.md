# Cups

```sh
apk add cups cups-filters hplip
rc-update add cupsd
rc-service cupsd start
sed -i 's/^lpadmin:x:105:$/lpadmin:x:105:root/' /etc/group
```
