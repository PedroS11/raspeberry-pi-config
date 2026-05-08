# iCloud Photos Backup

To regularly back up all your photos and videos from iCloud using a cron-like system, you first need to create a [mounted drive](https://github.com/PedroS11/raspeberry-pi-config/blob/main/mount-hardrive.md). Optionally, you can also [configure Samba](https://github.com/PedroS11/raspeberry-pi-config/blob/main/samba.md) to access the backup folder from other devices.

Once these steps are complete, you should have a mounted folder available at `/mnt/usb1`.

## Backup Flow

For the backup process, we will use [docker-icloudpd](https://github.com/boredazfcuk/docker-icloudpd), a Docker container that can be configured to back up your iCloud photos into a local folder.

Every day (or at any configured interval), the container will check your iCloud account and download any new photos or videos. You can also configure whether files deleted from iCloud should also be removed from the local backup.

## Authentication

To allow the container to access your account, you will need to log in with your Apple ID email and password the first time the container runs. The authentication token is valid for 30 days, after which re-authentication is required.

You have two options for handling re-authentication:

- Manually access the container and rerun the authentication command
- Use a Telegram bot to manage the authentication flow for you

---

## Configuration

### Create the Backup Folder

```bash
mkdir /mnt/usb1/icloud_backup
```

Or choose any other name, as long as it is inside the mounted folder.

### Give Docker Permissions to the Folder

```bash
sudo chown -R 1000:1000 /mnt/usb1/icloud_backup
```

### Create the `.mounted` File

This file is required by the container.

```bash
touch /mnt/usb1/icloud_backup/.mounted
```

---

## Telegram Bot Setup

If you want to use Telegram to handle the re-authentication flow, follow these steps.

### 1. Create a Telegram Bot

1. Open Telegram
2. Search for `BotFather`
3. Start a chat with BotFather
4. Send:

```text
/newbot
```

5. Follow the prompts:
   - Choose a bot name
   - Choose a username ending in `bot`

Example:

```text
icloudpd_backup_bot
```

---

### 2. Get the Bot Token

After creating the bot, BotFather will send a token similar to:

```text
123456789:AAAbbbbCCCCddddEEEE
```

Save this value.

---

### 3. Start a Chat With the Bot

Search for your bot in Telegram and press:

```text
START
```

Or send any message.

This step is required before the bot can message you.

---

### 4. Get Your Telegram Chat ID

Open this URL in your browser:

```text
https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
```

Example:

```text
https://api.telegram.org/bot123456789:AAAbbbbCCCCddddEEEE/getUpdates
```

You will receive JSON output similar to:

```json
{
  "result": [
    {
      "message": {
        "chat": {
          "id": 123456789
        }
      }
    }
  ]
}
```

Your chat ID is:

```text
123456789
```

---

## docker-compose.yml

All environment variables can be changed. The full list of available options can be found [here](https://github.com/boredazfcuk/docker-icloudpd/blob/master/CONFIGURATION.md#configuration-items).

```yaml
version: "3.8"

services:
  icloudpd:
    container_name: icloudpd
    image: boredazfcuk/icloudpd:latest

    volumes:
      - ./config:/config
      - /mnt/usb1/icloud_backup:/home/user/iCloud # CHANGE THIS PATH IF YOU CHOSE A DIFFERENT ONE

    environment:
      - TZ=Europe/Lisbon

      - apple_id=YOUR_EMAIL

      - user_id=1000 # SAME ID USED WHEN GIVING DOCKER PERMISSIONS
      - group_id=1000 # SAME ID USED WHEN GIVING DOCKER PERMISSIONS

      # --- Telegram integration ---
      - telegram_token=YOUR_TELEGRAM_TOKEN
      - telegram_chat_id=YOUR_TELEGRAM_CHAT_ID
      - notification_type=Telegram
      - telegram_polling=true
      - notification_days=7

      # Synchronisation interval. 21600=6hr, 43200=12hr, 86400=24hr
      - download_interval=86400
      - download_path=/home/user/iCloud

      - folder_structure={:%Y/%m}

      - auto_delete=false
```

---

## Start the Container

```bash
docker compose up -d
```

---

## First-Time Authentication

```bash
docker exec -it icloudpd sync-icloud.sh --Initialise
```

You will be prompted for:

- Your Apple ID password
- The 2FA code sent to your iPhone

Enter the requested information and press Enter.

---

## Restart the Container

```bash
docker compose restart
```

Now the container will automatically back up your photos every day.

The first sync may take a while because the container needs to scan and download your entire library.

You can check the logs with:

```bash
docker compose logs
```

And you should get something like this

<img width="692" height="1228" alt="image" src="https://github.com/user-attachments/assets/700dd69a-1e03-47aa-8c26-bf54c4183edd" />

\
Be aware the log line where you can see the username that you will require for re authenticating via Telgram. Whatever is after `Sync user: ` will be username you will use.

---

## Re-Authentication Process

### Using Telegram

If you configured Telegram integration, you should see in the logs that a `user` was created.

In your Telegram bot chat:

1. Send:

```text
user auth
```

2. Apple will send a 2FA code to your iPhone
3. Reply to the bot with:

```text
user THE_CODE
```

Example:

```text
user 123456
```

The container should authenticate successfully.

---

### Manual Re-Authentication

Go to the folder containing your `docker-compose.yml` file and run:

```bash
docker exec -it icloudpd sync-icloud.sh --Initialise
```

Then repeat the same steps used during the initial authentication.
