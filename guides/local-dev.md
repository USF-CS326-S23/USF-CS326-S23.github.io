---
layout: default
title: Local Dev
nav_order: 3
parent: Guides
permalink: /guides/local-dev
---

# Setting up a Local Development Environment for xv6

It turns out that we are experiencing too much load on griffin for qemu/xv6 to work reliably. To ensure you can work on your labs and projects without disruption you need to get a local development working on your own computer. I provide detailed instructions for both macOS and Windows below.

# Contents
1. [macOS](#macos)
2. [Windows](#windows)


## macOS

## Windows

The instructions below are based on a Windows 10 installation. I assume the instructions will work for Windows 11, but there might be some minor differences.

### Download the Ubuntu 22.04.2 LTS Application

Go to the Windows Store and install the Ubuntu 22.04.2 LTS application:

![local-dev-win-ubuntu-app](/files/local-dev-win-ubuntu-app.png)

This is an easy to install version of WSL (Windows subsystem for Linux) running Ubuntu.

### Launch Ubuntu and create a Unix username

![local-dev-win-ubuntu-app](/files/local-dev-win-ubuntu-username.png)

### Update Ubuntu and install the build tools

```text
$ sudo apt update && sudo apt upgrade
$ sudo apt install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu
```

Note that you can access your Ubuntu file system using ```\\wsl$\Ubuntu-22.04\home\<username>```, where ```<username>``` is the username you created when you first start the Ubuntu app.

### Install micro

The vim editor comes installed by default. If you want micro, you can install it like this:

``` text
$ sudo apt install micro
```

### Setup ssh for GitHub access

You can copy your existing ssh key or create a new key. If you create a new key pair, you will need to add it to GitHub using the browswer interface.

In your ```~/.ssh``` directory you should have:
```
config your_key_name your_key_name.pub
```

You should make sure the ```~/.ssh``` directory and files inside this directory have the correct permissions:

```
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/*
```

If you need to create a new key, you follow these instructions. In your Ubuntu terminal, create ```~/.ssh``` if it does not exist:

```text
$ cd
$ mkdir .ssh
$ chmod 700 .ssh
```

Note that when you type cd with no arguments, this will put you into your home directory. If you don't have a ```~/.ssh``` directory, then you can create one:

The chmod command restricts access to the .ssh directory and is needed by ssh to work properly. Now you can cd into your .ssh directory and create a new keypair:

```text
% cd .ssh
% ssh-keygen -t ed25519 -C "your_username@dons.usfca.edu" -f id_ed25519_cs326_2023s
```

You will be asked to enter a passphrase, which is a just a password, but you should make it multple words or a sentence that you can remember.

This will create two files:

```text
id_ed25519_cs326_2023s
id_ed25519_cs326_2023s.pub
```

You should change the permissions for these new files:

```
% chmod 600 id_ed25519_cs326_2023s*
```

Now we need to configure your local ssh by editing the ```.ssh/config``` file. If you already have a ```.ssh/config``` file, then you can add to it, otherwise you can create it using a text editor. You want to put the following into your ```.ssh/config```:

```text
Host github.com
  AddKeysToAgent yes
  ForwardAgent yes
  IdentityFile ~/.ssh/id_ed25519_cs326_2023s
  User your_username
```

Notes:
- UseKeychain only applies to macOS
- The github.com entry will be used to securing access your GitHub repos
- Make sure to put your username where you see ```your_username```

Once you have edited or created ```.ssh/config```, set its permissions:

```text
% chmod 600 config
```

If you created a new key, you will have to add it to your settings on GitHub:

[Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)


### Test your ssh connection to github.com

You can test your ssh connection by cloning one of your labs or projects using the ssh link from your lab or project page on GitHub. You can also type the following:

```text
$ ssh -T git@github.com
```
This will tell you if you can successfully connect to GitHub using ssh:
```text

```text
benson@win10:~$ ssh -T git@github.com
Hi gdbenson! You've successfully authenticated, but GitHub does not provide shell access.
```

### Setup keychain

You can use the Debian keychain progam so that you don't have to type your passphrase:

[Use an ssh-agent in WSL with your ssh setup from windows 10](https://pscheit.medium.com/use-an-ssh-agent-in-wsl-with-your-ssh-setup-in-windows-10-41756755993e)

### Test xv6 and setup the Autograder

Try ```make qemu``` and then go to the lab01 spec to find instructions for setting up the Autograder.



