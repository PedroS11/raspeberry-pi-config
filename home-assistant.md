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

## Back up to Google Drive

Access HA page https://www.home-assistant.io/integrations/google_drive/

In the Google Dashboard when asked to, there are a few steps to do:
- Create a project
- In the Audience, publish to production
- Create credentials 
- Enable Google Drive api - https://console.cloud.google.com/apis/library/drive.googleapis.com?


Enable the back up Google Drive in the HomeAssistant backup page http://raspberrypi.local:8123/config/backup/settings

![alt text](./ha-drive-backup.png)

## Add HACS

To add HACS, follow the instructions [here](https://hacs.xyz/docs/use/download/download/#to-download-hacs-container).

<details>
<summary>In case it goes down</summary>

1. Open a terminal.
2. Go inside the container with docker exec -it <name of the container running homeassistant> bash.
3. Run the HACS download script.

> wget -O - https://get.hacs.xyz | bash -

4. Restart Home Assistant

5. In Home Assistant, go to Settings > Devices & services.

5. Clear your browser cache.

HACS won't show up in the list unless you clear your browser cache or perform a hard refresh.
In the bottom right corner, select + Add integration.


6.  Search for HACS and select it.

7. Acknowledge the statements and select Submit.

8. Authenticate the integration:

HACS uses a device OAuth flow for authentication with GitHub.
Copy the device code and select the link https://github.com/login/device.

9. Sign in to GitHub.

If you are not signed in to GitHub in your browser, you need to sign up or sign in now to continue the setup.
If you are already signed in, you can skip this part.


10. Enter the device code you copied in the previous step and select Continue.

11. Select Authorize HACS.

12. Once you see the confirmation screen, you can close the tab and go back to Home Assistant.

13. Assign HACS to an area and select Finish.

14. Congrats! You have installed the HACS integration in Home Assistant.

Now, go checkout the configuration options.
Don't forget to setup a backup process.
</details>