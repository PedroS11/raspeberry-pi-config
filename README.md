# Configuration page for Raspberry Pi

Basic configs from the [official website](https://www.raspberrypi.com/documentation/computers/configuration.html#external-storage).

On RaspberryPi Imager
- Use latest Pi OS for Raspberry Pi 5
- Enable SSH
- Select your WIFI network

Access your router and consider setting a static IP address to you raspberry so it doesn't change every time your router reboots.

## Enable VNC access

After installation and boot, run
> sudo raspi-config

Select `Interface Options`, enable VNC and reboot the Pi.

## Docker configuration

Can be found [here](./docker.md).

## Git configuration

Can be found [here](./git.md).

## Home Assistant configuration

Can be found [here](./home-assistant.md).

## USB Hard drive and Samba

The steps to mount the hard drive can be found [here](./mount-hardrive.md).

Then, the steps to setup sambda can be found [here](./samba.md).

## Create remove access with DuckDNS + Wireguard

All the steps can be found [here](./vpn-wireguard.md).

## Backup Pi images to HDD + Google drive

All the steps can be found [here](./image-backups.md).
