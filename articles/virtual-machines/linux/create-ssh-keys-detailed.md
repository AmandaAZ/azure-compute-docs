---
title: Detailed steps - SSH key pair for Azure Linux VMs | Microsoft Docs
description: Learn detailed steps to create and manage an SSH public and private key pair for Linux VMs in Azure.
services: virtual-machines-linux
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: ''

ms.assetid:
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 03/30/2018
ms.author: danlep

---

# Create and use an SSH key pair for a Linux VM in Azure - background and steps
With an SSH key pair, you can create a Linux virtual machine on Azure that defaults to using SSH keys for authentication, eliminating the need for passwords to log in. VMs created with the Azure portal, Azure CLI, Resource Manager templates, or other tools can include your SSH public key as part of the deployment, which allows you to to use SSH key authentication for SSH connections. 

This article provides detailed background and steps to create and manage an SSH RSA public and private key file pair (also referred to as *ssh-rsa* keys) for SSH connections from a client compute. If you want quick steps to create and use an SSH key pair, see [How to create an SSH public and private key pair for Linux VMs in Azure](mac-create-ssh-keys.md).

## Overview of SSH and keys

SSH is an encrypted connection protocol that allows secure logins over unsecured connections. It is the default connection protocol for Linux VMs hosted in Azure. Although SSH itself provides an encrypted connection, using passwords with SSH connections still leaves the VM vulnerable to brute-force attacks or guessing of passwords. A more secure and preferred method of connecting to a VM using SSH is by using public and private keys, also known as SSH keys. 

* The *public key* is placed on your Linux VM, or any other service that you wish to use with public-key cryptography.

* The *private key* is what you present to your Linux VM when you log in, to verify your identity. Protect this private key. Do not share it.

These public and private keys can be used on multiple VMs and services. You do not need a pair of keys for each VM or service you wish to access. 

Your public key can be shared with anyone; but only you (or your local security infrastructure) possess your private key. 

The SSH private key should have a very secure passphrase to safeguard it. This passphrase is just to access the private SSH key file and *is not* the user account password. When you add a passphrase to your SSH key, it encrypts the private key using 128-bit AES, so that the private key is useless without the passphrase to decrypt it. If an attacker stole your private key and that key did not have a passphrase, they would be able to use that private key to log in to any servers that have the corresponding public key. If a private key is protected by a passphrase, it cannot be used by that attacker, providing an additional layer of security for your infrastructure on Azure.

[!INCLUDE [virtual-machines-common-ssh-support](../../../includes/virtual-machines-common-ssh-support.md)]

## SSH keys use and benefits

When you create an Azure VM by specifying the public key, Azure copies the public key (in the `.pub` format) to the `~/.ssh/authorized_keys` folder on the VM. SSH keys in `~/.ssh/authorized_keys` are used to challenge the client to match the corresponding private key on an SSH login connection. In an Azure Linux VM that uses SSH keys for authentication, Azure configures the SSHD server to not allow password logins, only SSH keys. Therefore, by creating an Azure Linux VM with SSH keys, you can help secure the VM deployment and save yourself the typical post-deployment configuration step of disabling passwords in the `sshd_config` file.

If you do not wish to use SSH keys, you can set up your Linux VM to use password authentication. If your VM is not exposed to the Internet, using passwords may be sufficient. However, you still need to manage your passwords for each Linux VM and maintain healthy password policies and practices, such as minimum password length and regular updates. Using SSH keys reduces the complexity of managing individual credentials across multiple VMs.


## Generate keys with ssh-keygen

To create the keys, a preferred command is `ssh-keygen`, which is available in the Azure Cloud Shell, a macOS or Linux host, the [Windows Subsystem for Linux](https://docs.microsoft.com/windows/wsl/about), and other tools. `ssh-keygen` asks a series of questions and then writes a private key and a matching public key. 

SSH keys are by default kept in the `~/.ssh` directory.  If you do not have a `~/.ssh` directory, the `ssh-keygen` command creates it for you with the correct permissions.

The following command creates a passphrase-secured (encrypted) SSH key pair using 2048-bit RSA, and it is commented for easy identification.  

```bash
ssh-keygen \
    -t rsa \
    -b 2048 \
    -C "azureuser@myserver" \
    -f ~/.ssh/id_rsa \
    -N mypassphrase
```

**Command explained**

`ssh-keygen` = the program used to create the keys

`-t rsa` = type of key to create which is the RSA format

`-b 2048` = the number of bits in the key

`-C "azureuser@myserver"` = a comment appended to the end of the public key file to easily identify it.  Normally an email is used as the comment, but use whatever works best for your infrastructure.

### Example of ssh-keygen

```bash
ssh-keygen -t rsa -b 2048 -C "azureuser@myserver"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/azureuser/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/azureuser/.ssh/id_rsa.
Your public key has been saved in /home/azureuser/.ssh/id_rsa.pub.
The key fingerprint is:
14:a3:cb:3e:78:ad:25:cc:55:e9:0c:08:e5:d1:a9:08 azureuser@myserver
The keys randomart image is:
+--[ RSA 2048]----+
|        o o. .   |
|      E. = .o    |
|      ..o...     |
|     . o....     |
|      o S =      |
|     . + O       |
|      + = =      |
|       o +       |
|        .        |
+-----------------+
```

#### Saved key files

`Enter file in which to save the key (/home/azureuser/.ssh/id_rsa): ~/.ssh/id_rsa`

The key pair name for this article. Having a key pair named `id_rsa` is the default; some tools might expect the **id_rsa** private key file name, so having one is a good idea. The directory `~/.ssh/` is the default location for SSH key pairs and the SSH config file.  If not specified with a full path, `ssh-keygen` creates the keys in the current working directory, not the default `~/.ssh`.

#### List of the `~/.ssh` directory

```bash
ls -al ~/.ssh
-rw------- 1 azureuser staff  1675 Aug 25 18:04 id_rsa
-rw-r--r-- 1 azureuser staff   410 Aug 25 18:04 id_rsa.pub
```

#### Key passphrase

`Enter passphrase (empty for no passphrase):`

It is *strongly* recommended to add a passphrase to your private key. Without a passphrase to protect the key file, anyone with the file can use it to log in to any server that has the corresponding public key. Adding a passphrase offers more protection in case someone is able to gain access to your private key file, giving you time to change the keys.




## Generate SSH keys during deployment

If you use the [Azure CLI 2.0](/cli/azure) to create your VM, you can optionally generate SSH public and private key files (if they don't already exist) by using the `--generate-ssh-keys` option. The keys are stored in the ~/.ssh directory. 

## Provide SSH public key for VM deployment

To create a Linux VM that uses SSH keys for authentication, provide your SSH public key when creating the VM using the Azure portal, CLI, Resource Manager templates, or other methods. When using the portal, you enter the public key itself. If you use the [Azure CLI 2.0](/cli/azure) to create your VM with an existing public key, specify the value or location of this public key by running the [az vm create](/cli/azure/vm#az_vm_create) command with the `--ssh-key-value` option. 

If you're not familiar with the format of an SSH public key, you can see your public key by running `cat` as follows, replacing `~/.ssh/id_rsa.pub` with your own public key file location:

```bash
cat ~/.ssh/id_rsa.pub
```

Output is similar to the following (here redacted):

```
ssh-rsa XXXXXXXXXXc2EAAAADAXABAAABAXC5Am7+fGZ+5zXBGgXS6GUvmsXCLGc7tX7/rViXk3+eShZzaXnt75gUmT1I2f75zFn2hlAIDGKWf4g12KWcZxy81TniUOTjUsVlwPymXUXxESL/UfJKfbdstBhTOdy5EG9rYWA0K43SJmwPhH28BpoLfXXXXXG+/ilsXXXXXKgRLiJ2W19MzXHp8z3Lxw7r9wx3HaVlP4XiFv9U4hGcp8RMI1MP1nNesFlOBpG4pV2bJRBTXNXeY4l6F8WZ3C4kuf8XxOo08mXaTpvZ3T1841altmNTZCcPkXuMrBjYSJbA8npoXAXNwiivyoe3X2KMXXXXXdXXXXXXXXXXCXXXXX/ azureuser@myserver
```

If you copy and paste the contents of the public key file to use in the Azure portal or a Resource Manager template, make sure you don't copy any additional whitespace or introduce additional linebreaks. For example, if you use macOS, you can pipe the public key file (by default, `~/.ssh/id_rsa.pub`) to **pbcopy** to copy the contents (there are other Linux programs that do the same thing, such as **xclip**).

If you prefer to use a public key that is in a multiline format, you can generate an RFC4716 formatted key in a pem container from the public key you previously created.

To create a RFC4716 formatted key from an existing SSH public key:

```bash
ssh-keygen \
-f ~/.ssh/id_rsa.pub \
-e \
-m RFC4716 > ~/.ssh/id_ssh2.pem
```

## SSH to your VM with an SSH client
With the public key deployed on your Azure VM, and the private key on your local system, SSH to your VM using the IP address or DNS name of your VM. Replace *azureuser* and *myvm.westus.cloudapp.azure.com* in the following command with the administrator user name (if configured) and the fully qualified domain name (or IP address):

```bash
ssh azureuser@myvm.westus.cloudapp.azure.com
```

If you provided a passphrase when you created your key pair, enter the passphrase when prompted during the login process. (The server is added to your `~/.ssh/known_hosts` folder, and you won't be asked to connect again until the public key on your Azure VM changes or the server name is removed from `~/.ssh/known_hosts`.)

## Use ssh-agent to store your private key passphrase

To avoid typing your private key file passphrase with every SSH login, you can use `ssh-agent` to cache your private key file passphrase. If you are using a Mac, the macOS Keychain securely stores the private key passphrase when you invoke `ssh-agent`.

Verify and use `ssh-agent` and `ssh-add` to inform the SSH system about the key files so that you do not need to use the passphrase interactively.

```bash
eval "$(ssh-agent -s)"
```

Now add the private key to `ssh-agent` using the command `ssh-add`.

```bash
ssh-add ~/.ssh/id_rsa
```

The private key passphrase is now stored in `ssh-agent`.

## Use ssh-copy-id to copy the key to an existing VM
If you have already created a VM, you can install the new SSH public key to your Linux VM with a command similar to the following:

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub ahmet@myserver
```

## Create and configure an SSH config file

We recommend that you create and configure an `~/.ssh/config` file to speed up log-ins and to optimize your SSH client behavior.

The following example shows a standard configuration.

### Create the file

```bash
touch ~/.ssh/config
```

### Edit the file to add the new SSH configuration

```bash
vim ~/.ssh/config
```

### Example configuration

Add the following configuration settings to the file.

```bash
# Azure Keys
Host fedora22
  Hostname 102.160.203.241
  User ahmet
# ./Azure Keys
# Default Settings
Host *
  PubkeyAuthentication=yes
  IdentitiesOnly=yes
  ServerAliveInterval=60
  ServerAliveCountMax=30
  ControlMaster auto
  ControlPath ~/.ssh/SSHConnections/ssh-%r@%h:%p
  ControlPersist 4h
  IdentityFile ~/.ssh/id_rsa
```

This SSH config gives you sections for each server to enable each to have its own dedicated key pair. The default settings (`Host *`) are for any hosts that do not match any of the specific hosts higher up in the config file.

### Config file explained

`Host` = the name of the host being called on the terminal. `ssh fedora22` tells `SSH` to use the values in the settings block labeled `Host fedora22`.  The label associated with `Host` can be any value that is logical for your usage and does not represent the actual hostname of any server.

`Hostname 102.160.203.241` = the IP address or DNS name for the server being accessed.

`User ahmet` = the remote user account to use when logging in to the server.

`PubKeyAuthentication yes` = tells SSH you want to use an SSH key to log in.

`IdentityFile /home/ahmet/.ssh/id_id_rsa` = the SSH private key and corresponding public key to use for authentication.

## SSH into Linux without a password

Now that you have an SSH key pair and a configured SSH config file, you are able to log in to your Linux VM quickly and securely. The first time you log in to a server using an SSH key, the command prompts you for the passphrase for that key file.

```bash
ssh fedora22
```

### Command explained

When `ssh fedora22` is executed, SSH first locates and loads any settings from the `Host fedora22` block, and then loads the remaining settings from the last block, `Host *`.

## Next Steps

Next up is to create Azure Linux VMs using the new SSH public key. Azure VMs that are created with an SSH public key as the login are better secured than VMs created with the default login method, passwords.

* [Create a Linux virtual machine with the Azure portal](quick-create-portal.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Create a Linux virtual machine with the Azure CLI](quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Create a Linux VM using an Azure template](create-ssh-secured-vm-from-template.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
