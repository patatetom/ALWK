# WiFi

WiFi support can be added to AWK (laptop for example)

```sh
# add Intel Wireless daemon
apk add iwd

# add iwd to boot runlevel
rc-update add iwd boot

# start iwd immediatly
rc-service iwd start

# connect access point
iwctl station wlan0 scan
iwctl station wlan0 get-networks
# Available networks                              
# -----------------------------
# Network name Security Signal
# -----------------------------
# SSID         psk      ****
iwctl station wlan0 connect SSID psk
# Type the network passphrase for SSID psk.
# Passphrase: *********
```

> `/var/lib/iwd/SSID.psk` will be created to enable next re-associations
