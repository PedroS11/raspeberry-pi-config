# Docker configuration

Got from https://pimylifeup.com/raspberry-pi-docker/

## Update dependencies

> sudo apt update

> sudo apt upgrade -y

## Get docker

> curl -sSL https://get.docker.com | sh

## Add current user to Docker group
Once Docker has finished installing on your Raspberry Pi, there are a couple more things we need to do.

For another user to be able to interact with Docker, it needs to be added to the docker group.

So, our next step is to add our current user to the docker group by using the usermod command as shown below. By using “$USER” we are inserting the environment variable that stores the current users name.

> sudo usermod -aG docker $USER

> sudo reboot

## Check if it was added
Once you have either logged back in or your device has restarted, we can verify that the Docker group has been added successfully by using the following command within the terminal.

This command lists the groups the current user is a member of.

> groups

If everything has worked correctly, you should see “$USER” in the list of groups.