# Setup Samba


## Setup docker-compose.yml

```
version: "3"

services:
  samba:
    image: dperson/samba
    container_name: samba
    restart: unless-stopped
    volumes:
      - /mnt/usb1:/mount # /mnt/usb1 is the folder created in the Mount Hard Drive steps
    ports:
      - "137:137/udp"
      - "138:138/udp"
      - "139:139/tcp"
      - "445:445/tcp"
    command: >
      -p
      -u "pi;raspberry"
      -s "Public;/mount;yes;no;yes"
```

Command explanation:
```
-u "pi;raspberry" → Samba user pi with password raspberry

-s "Public;/mount;yes;no;yes" → Share name Public

browsable = yes

read only = no

guest = yes

```

## Accessing it

On your OS, access it via Explorer. You can use the pi account or just the guest one

> smb://<YOUR_PI_ADDRESS>/Public

you can use it like this if configured this way on Pi OS install

> smb://raspberrypi.local/Public
