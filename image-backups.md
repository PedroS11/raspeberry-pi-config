# Backup PI Image to External HDD + Google Drive

1. Install rclone
> sudo apt update

> sudo apt install rclone -y

2. Configure rclone with Google Drive

Run:

> rclone config

Follow the prompts:

- Type n → New remote

- Name it e.g., `gdrive`

- Select drive (Google Drive)

- Follow instructions to authorize rclone (it will give a URL to open in your browser on another device and a code to paste back), don't put any client ID/Secret as the browser redirect will do it.

- Choose default options for the rest unless you want to customize

- Finish with q to quit

You now have a remote called `gdrive`.


1. Create this script 

```
#!/bin/bash

# -----------------------------
# Configuration
# -----------------------------
BACKUP_MOUNT_PATH="/mnt/usb1"            # Mount path
BACKUP_DIR="$BACKUP_MOUNT_PATH/pi_backups"        # Local HDD backup folder
REMOTE="gdriveimage:/pi_image_backup"    # Google Drive folder via rclone
MAX_LOCAL_BACKUPS=3                       # Number of local backups to keep
MAX_DRIVE_BACKUPS=2                       # Number of backups to keep on Google Drive
LOGS_FOLDER="$BACKUP_DIR/logs"
LOG_FILE="$LOGS_FOLDER/backup.log"
DATE=$(date +%Y-%m-%d)
IMAGE_NAME="pi_backup_$DATE.img.gz"

# -----------------------------
# Logging function
# -----------------------------
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# -----------------------------
# Create Logs folder if not exists
# -----------------------------
mkdir -p "$LOGS_FOLDER"

# -----------------------------
# Check if HDD is mounted
# -----------------------------
if ! mount | grep "$BACKUP_MOUNT_PATH"; then
    log "ERROR: HDD not mounted at $BACKUP_MOUNT_PATH. Backup aborted."
    exit 1
fi

# -----------------------------
# Start backup
# -----------------------------
log "Starting backup: $IMAGE_NAME"
if sudo dd if=/dev/nvme0n1p1 bs=4M status=progress | gzip > "$BACKUP_DIR/$IMAGE_NAME"; then
    log "Backup completed successfully: $IMAGE_NAME"
else
    log "ERROR: Backup failed!"
    exit 1
fi

# -----------------------------
# Keep only last $MAX_LOCAL_BACKUPS backups locally
# -----------------------------
cd "$BACKUP_DIR" || exit
log "Cleaning up old local backups, keeping last $MAX_LOCAL_BACKUPS"
ls -1t pi_backup_*.img.gz | tail -n +$((MAX_LOCAL_BACKUPS+1)) | xargs -r rm --
log "Old local backups cleaned up."

# -----------------------------
# Upload to Google Drive
# -----------------------------
log "Uploading $IMAGE_NAME to Google Drive..."
if rclone copy "$BACKUP_DIR/$IMAGE_NAME" "$REMOTE" --log-file="$LOGS_FOLDER/gdrive_upload.log" --log-level INFO; then
    log "Upload successful."

    # -----------------------------
    # Keep only last $MAX_DRIVE_BACKUPS backups on Drive
    # -----------------------------
    log "Cleaning old backups on Google Drive, keeping last $MAX_DRIVE_BACKUPS..."

    # List files rclone can see (created by this app)
    FILES=$(rclone lsf --files-only "$REMOTE" --log-level INFO)
    TOTAL_FILES=$(echo "$FILES" | wc -l)

    if [ "$TOTAL_FILES" -gt "$MAX_DRIVE_BACKUPS" ]; then
        DELETE_COUNT=$((TOTAL_FILES - MAX_DRIVE_BACKUPS))
        FILES_TO_DELETE=$(echo "$FILES" | sort | head -n "$DELETE_COUNT")

        IFS=$'\n'
        for file in $FILES_TO_DELETE; do
            log "Deleting old backup from Google Drive: $file"
            rclone delete "$REMOTE/$file" --log-file="$LOGS_FOLDER/gdrive_upload.log" --log-level INFO
        done
        unset IFS
    fi

    log "Old backups removed from Google Drive."

else
    log "ERROR: Upload failed. Check $LOGS_FOLDER/gdrive_upload.log"
fi

log "Backup script finished."
```

Where `/mnt/usb1` is the mount path to your HDD and `/dev/nvme0n1p1` is the mount path to your SD Card/Nvme used for booting. The HDD should be always automatically mounted for this to work, that part was already done in [Mount Hardrive](./mount-hardrive.md).

The script will keep the last two updates in the folder (defined by the `tail -n +3` part). To save space, the img file is compressed.

2. Make the script executable

> chmod +x /home/pi/backup_pi.sh

3. Schedule weekly backups with cron

Edit the crontab:

> crontab -e


Add the line to run every Sunday at 2 AM:

> 0 2 * * 0 /home/pi/backup_pi.sh


## Troubleshoot

### Wrong number of files return from `rclone lsf --files-only "$REMOTE" --log-level INFO`
If while configuring the remote, you set the permissions to only files created by rclone, the command won't return files created by anyone but rclone