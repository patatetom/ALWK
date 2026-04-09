# WiFi

WiFi support can be added to AWK (laptop for example)

```sh
# add Intel Wireless daemon
apk add iwd

# add iwd to boot runlevel
rc-update add cupsd boot

# start cups immediatly
rc-service iwd start

# connect access point
iwctl station wlan0 scan
iwctl station wlan0 get-networks
# Available networks                              
# -----------------------------
# Network name Security Signal
# -----------------------------
# AP           psk      ****
iwctl station wlan0 connect AP psk
# Type the network passphrase for AP psk.
# Passphrase: *********
```

> `/var/lib/iwd/AP.psk` will be created to enable next re-associations
