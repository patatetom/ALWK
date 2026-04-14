# Branch upgrade

switch AWK to new Alpine branch

```sh
# upgrade current branch
apk update
apk upgrade

# change branche number
sed -i 's/3.23/x.yy/g' /etc/apk/repositories

# upgrade to new branch
apk update
apk add --upgrade apk-tools
apk upgrade --available

# reboot AWK
reboot
```
