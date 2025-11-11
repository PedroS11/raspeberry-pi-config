# Configure Git

Steps copied from https://mathias.rocks/blog/2025/github-ssh-keys-raspberry-pi

## Generate SSH Key

> ssh-keygen -t ed25519 -C "your.email@example.com"

With filename like "~/.ssh/github"

## Start SSH Agent and Add Your Key

> eval "$(ssh-agent -s)"

ssh-add ~/.ssh/github

## Copy SSH pub key

> cat ~/.ssh/github.pub


## Add the Key to GitHub
Now we need to add your public key to your GitHub account:

- Open GitHub in your web browser
- Sign in to your account
- Click your profile picture in the top-right corner
- Select "Settings" from the dropdown menu
- Click "SSH and GPG keys" in the left sidebar
- Click "New SSH key"
    - Fill in the form:
    - Title: Give it a descriptive name (e.g., "Raspberry Pi")
    - Key type: Select "Authentication Key"
    - Key: Paste the public key you copied in Step 4
- Click "Add SSH key"