# Setup Samba


## Setup docker-compose.yml

### docker-compose.yml example from https://github.com/ServerContainers/samba

```
services:
  samba:
    container_name: samba
    image: ghcr.io/servercontainers/samba
    restart: unless-stopped
    # note that this network_mode makes it super easy (especially for zeroconf) but is not as safe as exposing ports directly
    # more about that here: https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/docker-security/docker-breakout-privilege-escalation/index.html#hostnetwork
    network_mode: host
    # uncomment to solve bug: https://github.com/ServerContainers/samba/issues/50 - wsdd2 only - not needed for samba
    #cap_add:
    #  - CAP_NET_ADMIN
    environment:
      # uncomment to enable fail fast (currently only fails fast if there are conflicts/errors during user/group creation)
      #FAIL_FAST: 1

      MODEL: 'TimeCapsule'
      AVAHI_NAME: StorageServer

      SAMBA_CONF_LOG_LEVEL: 3

      # uncomment to disable optional services
      #WSDD2_DISABLE: 1
      #AVAHI_DISABLE: 1
      #NETBIOS_DISABLE: 1

      # Can't use 1000 group as it collides with the Pi OS user
      GROUP_family: 1500

      # After the _ sits the user name account to use, replace username with the name of the user you want to create
      ACCOUNT_username: password
      UID_username: 1000
      GROUPS_username: family

      SAMBA_VOLUME_CONFIG_aliceonly: "[Admin Share]; path=/share; valid users = username; guest ok = no; read only = no; browseable = yes"
      
      SAMBA_VOLUME_CONFIG_guest: |
        [Guest Share]
         path = /share/Guest
         guest ok = yes
         browseable = yes

    volumes:
      - /etc/avahi/services/:/external/avahi
      - /mnt/usb1:/share
      - /mnt/usb1/Guest:/share/Guest
      
```


## Accessing it

On your OS, access it via Explorer. You can use the pi account or just the guest one

> smb://<YOUR_PI_ADDRESS>

you can use it like this if configured this way on Pi OS install

> smb://raspberrypi.local

If you log in as Guest, you can access Guest Share volume
If you log in with your user, you should select Admin Share volume9