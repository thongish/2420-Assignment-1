# Setting up a DigitalOcean droplet using doctl and cloud-init

# Table of contents
- [Introduction](#introduction)
- [Installing and setting up doctl](#installing-and-setting-up-doctl)
- [Uploading a custom image to DigitalOcean](#uploading-a-custom-image-to-digitalocean)
- [Setting up SSH keys](#setting-up-ssh-keys)
- [Configuring cloud-init](#configuring-cloud-init)
- [Deploying the droplet](#deploying-the-droplet)
- [Verify everything worked](#verify-everything-worked)

# Introduction
This tutorial will walk you through the process of using the tools `doctl`, and `cloud-init` to set up an Arch Linux droplet on DigitalOcean. 

**Things we'll need:**
<!-- - A local machine running the cloud version of Arch Linux -->
<!-- - A local machine running Microsoft Windows 10/11 -->
- A DigitalOcean account
- An existing Arch Linux droplet
- Neovim (or your preferred text editor)

>[!TIP]
>You will be copying and pasting into your terminal frequently throughout this tutorial. If you're using Command Prompt or Windows Powershell, you can enable copy/paste by clicking the icon on the top left of your terminal and choosing the **Properties** option. Once inside the **Properties** window, make sure to check the **Use Ctrl+Shift+C/V as Copy/Paste** checkbox.
---

# Installing and setting up doctl

---

# Uploading a custom image to DigitalOcean


---

<!--add part about uploading pub key to DigitalOcean and rewrite from droplet perspective not windows-->
# Setting up SSH keys
Secure shell (SSH) is a network protocol used to initiate secure connections over an unsecured network. Through the secure connection, you can do things such as sending commands or transferring files, and more. SSH will be essential to accessing your DigitalOcean droplets.

We'll get started by creating an SSH public/private key-pair on your Arch Linux droplet using this command::
```
ssh-keygen -t ed25519 -f ~/.ssh/<key name> -C <youremail@email.com>
```
You will be prompted to enter a passphrase. This passphrase will be used every time you connect to your droplet via SSH.

Once you've generated the public/private key-pair, you'll be able to find both keys in the .ssh folder within your user folder. They will look something like this: `bobs-key` and `bobs-key.pub`. `bobs-key` (the private key) will stay on your Arch Linux droplet, while `bobs-key.pub` will be uploaded to DigitalOcean to be used by your droplets.

You can do that by running this command:
```
doctl compute ssh-key import <key identifier> --public-key-file ~/.ssh/<key name>.pub
```

SSH will use this pair of keys to send back-and-forth encrypted messages from your local machine to the DigitalOcean droplet. The messages can only be decrypted if the public and private keys match. 


**Now that you have new a new pair of SSH keys, you can head on over to your Arch Linux DigitalOcean droplet.**

---

# Configuring cloud-init
`Cloud-init` is a tool used to automate the initializtion of cloud instances such as the Arch Linux droplet you'll be deploying at the end of this tutorial. `Cloud-init` will allow your droplet to automatically create users, install software, authorize SSH keys, run scripts, and more. 

To get started, let's create and open a `cloud-init` configuration file in neovim:
```
nvim ~/cloud-config.yml
```
>[!NOTE]
>If that command doesn't work, you might not have neovim installed on your system. Run the command:
>```
>sudo pacman -S neovim
>```

Next, copy and paste this code into your file, changing the necessary fields:
```
#cloud-config
users:
  - name: <user name>
    primary_group: <user group>
    groups: wheel
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - ssh-ed25519 <your ssh public key string>

packages:
  - ripgrep
  - rsync
  - neovim
  - fd
  - less
  - man-db
  - bash-completion
  - tmux

disable_root: true
```
>[!NOTE]
>You can run the command:
>```
>cat ~/.ssh/<your ssh key ending in .pub>
>```
>And then select all of the displayed content and press Ctrl+Shift+C to copy it to your clipboard.

**Now that you have a cloud-init configuration file, we can move on to deploying the droplet!**

---

# Deploying the droplet
Before running the command to deploy the droplet, you'll need to run some preliminary commands to get the IDs of your uploaded SSH key, custom Arch Linux image ID, and optionally, the region slug.

1. This command will list your custom images with ID being the first piece of text:
```
doctl compute image list-user
```

2. This command will list your SSH keys and their IDs:
```
doctl compute ssh-key list
```

3. This command will give you a list of valid region slugs:
```
doctl compute region list
```
Once you have all the necessary information you need, you can run this command:

```
doctl compute droplet create --image <image ID> --size s-1vcpu-1gb --ssh-keys <SSH key ID> --region <your preferred region slug> --user-data-file ~/cloud-config.yml --wait <droplet name>
```
>[!NOTE]
>It may seem like your terminal froze, but just let it work and you'll eventually be greeted with an output detailing the information about the droplet you just created.

**Congrats! You just deployed a droplet using `doctl` and `cloud-init`! Now you just need to make sure everything worked.**

---

# Verify everything worked
Now that your droplet is up and running, you should try and connect to it via SSH. 

First, you'll need to get the IPv4 address your droplet is running on. You can use this command to list your running droplets and find the IPv4 address:
```
doctl compute droplet list
```
Once you have the IPv4 address you can immediately try using SSH to connect to your droplet with this command:
```
ssh -i ~/.ssh/<private key> <username>@<IPv4 address>
```
But you can make this easier by creating an SSH configuration file and entering your droplet's information into it. Run this command to create your own SH configuration file:
```
nvim ~/.ssh/config
```
Paste this text into your config file, replacing the necessary fields:
```
Host <droplet alias>
  HostName <droplet IPv4 address>
  User <username>
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/<private key>
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
```
Now you should be able to connect to your by simply running this command:
```
ssh <droplet alias>
```
**Congrats! You've made it to the end of the tutorial!**
<!--
[^1]: doctl is a command-line interface tool used to interact with DigitalOcean's cloud services.
[^2]: cloud-init is an industry standard tool used for cloud instance initialization. 
-->