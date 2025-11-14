# Mount USB Hard Drive

Steps copied from https://pimylifeup.com/raspberry-pi-mount-usb-drive/

## Check in which USB port is the hard drive connected

> lsblk

And it should say if it's /sda/sda1 or /sdb/sdb1

## Retrieving the Disk UUID and Type

Assuming the drive is sda1, then get the Disk UUID and Type

To find out more information about our drives filesystem, we can make use of the blkid tool.

1. Run the following command to retrieve information about your drive.

> sudo blkid /dev/sda1

Please note that you should replace /dev/sda1/ with the filesystem name you retrieved in the previous section.

2. From this command, you should have retrieved a result as we have below

> /dev/sda1: LABEL="My Passport" UUID="8A2CF4F62CF4DE5F" TYPE="ntfs" PTTYPE="atari" PARTUUID="00042ada-01"
Please make a note of the value for both the UUID and the TYPE.

3. Depending on the “type” of your filesystem, you may need to install additional drivers.

If you are using a drive that has a type of ntfs or exFAT, you will need to follow the appropriate steps below. Otherwise, you can continue to the next section.

### NTFS
To be able to use the NTFS format on your Raspberry Pi, you will need to install the NTFS-3g driver.

You can do this by running the following command.

> sudo apt install ntfs-3g

You can find out more about NTFS on the Raspberry Pi by following the guide.

### exFAT
To add support for the exFAT filesystem, we will need to install two packages.

> sudo apt install exfat-fuse

> sudo apt install exfatprogs

These two packages will allow the Raspberry Pi to read and interpret exFAT drives. You can learn more about exFAT on the Raspberry Pi by reading our guide.

## Mounting the Drive to the Raspberry Pi

With everything now prepared and the UUID and type of the drive on hand, we can now proceed to mount the drive.

1. To start, we need to make a directory where we will mount our drive to.

We can do this by running the following command. You can name the folder we are mounting anything, but for this tutorial, we will be using the name `usb1`.

> sudo mkdir -p /mnt/usb1

2. Let’s now give our pi user ownership of this folder by running the command below.

> sudo chown -R pi:pi /mnt/usb1

3. Next, we need to modify the fstab file by running the command below.

> sudo nano /etc/fstab

This file controls how drives are mounted to your Raspberry Pi.

4. The lines that we add to this file will tell the operating system how to load in and handle our drives.

For this step, you will need to know your drives UUID ([UUID]) and TYPE ([TYPE]).

Add the following line to the bottom of the file, replacing [UUID] and [TYPE] with their required values.

> UUID=[UUID] /mnt/usb1 [TYPE] defaults,auto,users,rw,nofail,noatime 0 0
Once done, save the file by pressing CTRL + X, followed by Y, then the ENTER key.

5. Now since the Pi will likely have automatically mounted the drive, we will need to unmount the drive. (if it didn't, it's ok too, after these changes + reboot will fix it)

A simple way to do this is to use the following command (Replace /dev/sda1 with the filesystem name you found earlier in this guide).

> sudo umount /dev/sda1

6. Once the drive has been unmounted, we can now go ahead and mount it again.

To mount the drive again, you can use the following command.

> sudo mount -a

The drive should now be mounted using the changes we made to the fstab file.

7. If you want to make sure the drives are restored after the Pi has been shut down, then run the following command:

> sudo reboot



## Extra steps
### Correcting Permissions

If you are using a drive with a type of ext4 or another native Linux format, you can also correct the file permissions.

This method won’t work for NTFS or exFAT as they do not support the same permission system that Linux uses.

For this step, make sure you have the mount directory you set on hand, for our example, we will be using the directory /mnt/usb1.

1. Start by changing to the superuser.

> sudo su

2. Now run the following two commands.

These commands will run through the directory, setting permissions for both files and directories.

> find /mnt/usb1/ -type d -exec chmod 755 {} \;

> ind /mnt/usb1/ -type f -exec chmod 644 {} \;

If you are using a different mount path, make sure you replace “/mnt/usb1/” with your own.

3. You can exit out of the superuser by using the command below

> exit

Drives Not Mounting on Boot
Another problem you may come across is the drive not being mounted on boot. There have been some changes to Raspberry Pi OS and the Raspberry Pi that might cause issues with mount the drive in time.

The best way to work around this would be to add the following lines before the exit 0 line in the /etc/rc.local file.

```
sleep 20
sudo mount -a
```