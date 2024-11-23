---
sidebar_label: 'Getting Started'
sidebar_position: 1
---

# Getting Started

## Welcome to the Cardano node workshop!

Let's get started!

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




