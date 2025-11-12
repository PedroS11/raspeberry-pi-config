# Configure Home Assistant

Steps copied from https://pimylifeup.com/home-assistant-docker-compose

## Docker-compose.yaml

In the folder where HA will sit, create the following docker-compose.yaml file

```
services:
  homeassistant:
    container_name: homeassistant
    image: "ghcr.io/home-assistant/home-assistant:stable"
    volumes:
      - ./CONFIG_FOLDER:/config
      - /etc/localtime:/etc/localtime:ro
      - /run/dbus:/run/dbus:ro
    restart: unless-stopped
    privileged: true
    network_mode: host
    environment:
      TZ: Europe/Lisbon
```

Then, run

> docker compose up -d
