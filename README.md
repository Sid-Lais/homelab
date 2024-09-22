# OS Installation
- Use Rapsberry pi Imager to burn os to sd card
- Make sure to install `Raspberry Pi OS Lite 64-bit`
- Username - `Sid_Lais`

# Install necessary tools

```zsh
sudo apt install -y neovim networkd-dispatcher
```

# Setup ssh keys
- Go to ~/.shh folder on your pc and copy the public ssh key you want to use
- Ssh to raspberrypi using password, then run the following commands

```zsh
mkdir .ssh
cd .ssh
touch authorized_keys
vim authorized_keys
```

# First Update
```zsh
sudo apt update && sudo apt upgrade
```

# Docker Install
## Uninstall any conflicting versions
Run the following command to uninstall all conflicting packages:

```zsh
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt remove $pkg; done
```

## Installing Docker through apt

1. Set up Docker's apt repository.
```zsh
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
```

2. Install the Docker packages.
```zsh
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3. Added User to Docker Group -
```zsh
sudo groupadd docker
sudo usermod -aG docker $USER
```

Refrence - https://docs.docker.com/engine/install/debian/

# Configure `mount-ssd.service` before proceeding

https://www.zdnet.com/article/how-to-permanently-mount-a-drive-in-linux-and-why-you-should/

# Static IP Config

Edit the current connection and use static ip.
```zsh
sudo nmtui
```
Use the arrow key to scroll down to the IPv4 CONFIGURATION option. Change it from Automatic to Manual.

Refrence - https://itsfoss.com/raspberry-pi-static-ip/

# ZSH4HUMANS Setup

```zsh
if command -v curl >/dev/null 2>&1; then
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/romkatv/zsh4humans/v5/install)"
else
  sh -c "$(wget -O- https://raw.githubusercontent.com/romkatv/zsh4humans/v5/install)"
fi
```

Refrence - https://github.com/romkatv/zsh4humans

# Setting Up New Github SSH Keys

## Generating a new SSH Key
```shell
ssh-keygen -t ed25519 -C "sidlais.ls@gmail.com"
```
Use `homelab_gh` for key name.

## Adding your SSH key to the ssh-agent

```shell
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/homelab_gh
```

Refrences - [Ref 1](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

## Adding a new SSH key to your GitHub account

Follow this guide - https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account

# Tailscale Setup -

## Install Tailscale -

```zsh
curl -fsSL https://tailscale.com/install.sh | sh
```

## Login -

```zsh
sudo tailscale login
```

## Subnets Setup -

1. To enable local network access / exit node from raspberry pi -

```zsh
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
```

2. Now run the following command on the pi -

```zsh
sudo tailscale up --advertise-routes=192.168.0.0/24,192.168.1.0/24 --advertise-exit-node
```

3. Now, Enable subnet routes from the admin console.

## Subnets Optimisation for Ethernet

```zsh
NETDEV=$(ip route show 0/0 | cut -f5 -d' ')
sudo ethtool -K $NETDEV rx-udp-gro-forwarding on rx-gro-list off
```

### Enable on each boot

Changes made via ethtool are not persistent and will be lost after the machine shuts down. On Linux distributions using networkd-dispatcher (which you can verify with systemctl is-enabled networkd-dispatcher), copy and run the following commands to create a script that will configure these settings on each boot.

```zsh
printf '#!/bin/sh\n\nethtool -K %s rx-udp-gro-forwarding on rx-gro-list off \n' "$(ip route show 0/0 | cut -f5 -d" ")" | sudo tee /etc/networkd-dispatcher/routable.d/50-tailscale
sudo chmod 755 /etc/networkd-dispatcher/routable.d/50-tailscale
```

Test the created script to ensure it runs successfully on your machine:

```zsh
sudo /etc/networkd-dispatcher/routable.d/50-tailscale
test $? -eq 0 || echo 'An error occurred.'
```

Ref - https://tailscale.com/kb/1019/subnets , https://tailscale.com/kb/1320/performance-best-practices

# Cloudflare Tunnels

Go to `Cloudflare Dashboard` > `Zero Trust` > `Networks` > `Tunnels` then configure.

# Synchting Setup

## Install Syncthing

```zsh
sudo mkdir -p /etc/apt/keyrings
sudo curl -L -o /etc/apt/keyrings/syncthing-archive-keyring.gpg https://syncthing.net/release-key.gpg
echo "deb [signed-by=/etc/apt/keyrings/syncthing-archive-keyring.gpg] https://apt.syncthing.net/ syncthing stable" | sudo tee /etc/apt/sources.list.d/syncthing.list

# Update and install syncthing:
sudo apt update
sudo apt install syncthing

```

Refrence - https://apt.syncthing.net/

# Enable and Start the Service

```zsh
sudo systemctl enable syncthing@Sid_Lais.service
sudo systemctl start syncthing@Sid_Lais.service
```

# Make Syncthing Accessible from Anywhere

```zsh
vim /.local/state/syncthing/config.xml
```

Change

```xml
<address>127.0.0.1:8384</address>
```

to

```xml
<address>0.0.0.0:8384</address>
```
