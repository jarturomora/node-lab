---
sidebar_label: 'Getting Started'
sidebar_position: 1
---

# Getting Started

### Welcome to the Cardano node workshop!

## Connecting to your server via SSH

The first thing we need to do is connect to your server via secure shell (SSH). 

Mac OSX and Linux (most distros) have the SSH client already installed. If you have a Windows machine, you'll need to install [putty](https://www.putty.org/)

You will be handed a card with your server IP, user and password printed on it. You will use these credentials to login to your Raspberry Pi server via SSH. 

:::warning

We are not going to use a best practice for SSH today as we are relaxing security for ease of use for this lab exercise. In a production environment you will need to harden your SSH server connection methods. 

The following examples are ways to harden SSH: 
- Change default SSH port from 22 to a much higher number. 22 is commonly scanned and attacked via brute-force attacks and scanners typically start and ascend when looking for opened ports. 
- Disable password authentication and use generated SSH keys only. 
- Configure firewall to only allow SSH connections from specific IPs that you own.
- Install `fail2ban` or similar to limit authentication attempts to your server.
- Configure a tunnel between devices using something like `wireguard` or `openvpn` between client and server.

:::

First, we need to connecdt to your device with SSH. 

For Mac OSX and Linux, use the following command. 

```
ssh n@10.20.20.X
```

For putty users, make sure port is `22`, protocol or connection type is `ssh` and that your IP is correct. Then `open` the connection. 

![putty1](/img/putty1.png)

:::tip

Replace the `X`, or whatever is in the last octet in `10.20.20.X` with the correct information on your credential card. 

::: 

## Time to get started!

Once you have connected it is time to get started!

Before we get rolling, let's check for updates/upgrades to our server.

```
sudo apt-get update -y && sudo apt-get upgrade -y
```

Next, let's make a few working directories

```
cd
mkdir -p preview/{node,config,socket,test-db,scripts,bin,logs}
```

Now, let's add a few path items to our `.bashrc` file

Open your `.bashrc` file with a text editor

```
nano ~/.bashrc
```
Then, copy the following export lines and paste them at the end of the `.bashrc` file

```
export PATH="/home/n/preview/bin:$PATH"
export CARDANO_NODE_SOCKET_PATH="/home/n/preview/socket/node.socket"
```
Save with `ctrl + o` and exit the editor with `ctrl + x`


Let's source our bash file and then check our paths

```
cd
. .bashrc
echo $PATH $CARDANO_NODE_SOCKET_PATH
```

You should see the `/home/n/preview/bin:/home/n/` and `/home/n/preview/socket/node.socket` paths as part of the output




