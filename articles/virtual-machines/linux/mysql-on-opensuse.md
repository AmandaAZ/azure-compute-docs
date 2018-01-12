---
title: Install MySQL on an OpenSUSE VM in Azure | Microsoft Docs
description: Learn to install MySQL on an OpenSUSE Linux VMirtual machine in Azure.
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager

ms.assetid: 1594e10e-c314-455a-9efb-a89441de364b
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 01/10/2018
ms.author: cynthn

---
# Install MySQL on a virtual machine running OpenSUSE Linux in Azure

[MySQL](http://www.mysql.com) is a popular, open-source SQL database. This tutorial shows you how to create a virtual machine running OpenSUSE Linux, then install MySQL.


## Create a virtual machine running OpenSUSE Linux

First, create a resource group. In this example, we are naming the resource group *mySQSUSEResourceGroup* and creating it in the **East US** region.

```azurecli-interactive
az group create --name mySQLSUSEResourceGroup --location eastus
```

Create the VM. In this example, we are naming the VM *myVM*. We are also going to use a VM size *Standard_D2s_v3*, but you should choose the [VM size](sizes.md) you think is most appropriate for your workload.

```azurecli-interactive
az vm create --resource-group mySQLSUSEResourceGroup \
   --name myVM \
   --image openSUSE-Leap \
   --size Standard_D2s_v3 \
   --generate-ssh-keys
```


## Connect to the Virtual Machine

You'll use SSH connect to the virtual machine. In this example, the public IP address of the VM is *10.111.112.113*. You can see the IP address in the output of the previous command.

```azurecli-interactive  
ssh 10.111.112.113
```

 
## Update the VM
 
After you're connected to the virtual machine, you can install system updates and patches. 
   
```bash
sudo zypper update
```

Follow the prompts to update your VM.

## Install and run MySQL on the virtual machine


Install the MySQL Community Server edition.

```bash
sudo zypper install mysql-community-server
```
 
Output:

You just installed MySQL server for the first time.

You can start it using:
 rcmysql start

During first start empty database will be created for your automatically.

PLEASE REMEMBER TO SET A PASSWORD FOR THE MariaDB root USER !
To do so, start the server, then issue the following commands:

'/usr/bin/mysqladmin' -u root password 'new-password'
'/usr/bin/mysqladmin' -u root -h <hostname> password 'new-password'

Alternatively you can run:
'/usr/bin/mysql_secure_installation'

which will also give you the option of removing the test
databases and anonymous user created by default. This is
strongly recommended for production servers.

 
 
 
 
Set MySQL to start when the system boots.

```bash
insserv mysql

sudo insserv /usr/lib/systemd/system/mysql.service
```

Start the MySQL daemon (mysqld) manually.

```bash   
rcmysql start
```
   
Check the status of the MySQL daemon.
 
```bash 
rcmysql status
```
   
Stop the MySQL daemon.

```bash   
rcmysql stop
```

## mySQL password

After installation, the MySQL root password is empty by default. We recommended that you run **mysql\_secure\_installation**, a script that helps secure MySQL. The script prompts you to change the MySQL root password, remove anonymous user accounts, disable remote root logins, remove test databases, and reload the privileges table. We recommended that you answer yes to all of these options and change the root password.

Run the script MySQL installation script.

```bash
mysql_secure_installation
```

Log in to MySQL:

```bash  
mysql -u root -p
```
Enter the MySQL root password (which you changed in the previous step) and you are presented with a prompt where you can issue SQL statements to interact with the database.


To create a new MySQL user, run the following command at the **mysql>** prompt:

```   
CREATE USER 'mysqluser'@'localhost' IDENTIFIED BY 'password';
```
   
The semi-colons (;) at the end of the line is crucial for ending the command.


## Create a database


Create a database and grant the `mysqluser` user permissions on it.

```   
CREATE DATABASE testdatabase;
GRANT ALL ON testdatabase.* TO 'mysqluser'@'localhost' IDENTIFIED BY 'password';
```
   
Note that database user names and passwords are only used by scripts connecting to the database.  Database user account names do not necessarily represent actual user accounts on the system.

To log in from another computer, type:

```   
GRANT ALL ON testdatabase.* TO 'mysqluser'@'<ip-address>' IDENTIFIED BY 'password';
```
   
Where `ip-address` is the IP address of the computer from which you connect to MySQL.


To exit the MySQL database administration utility, type:

```    
quit
```

## Add an endpoint

```azurecli-interactive
az vm open-port --port 3306 --resource-group mySQSUSEResourceGroup --name myVM
```

To remotely connect to the virtual machine from your computer, type:

```   
mysql -u mysqluser -p -h <yourservicename>.cloudapp.net
```   
    
For example, using the virtual machine you created in this tutorial, type this command:

```   
mysql -u mysqluser -p -h testlinuxvm.cloudapp.net
```


## Next steps
For details about MySQL, see the [MySQL Documentation](http://dev.mysql.com/doc/index-topic.html).




