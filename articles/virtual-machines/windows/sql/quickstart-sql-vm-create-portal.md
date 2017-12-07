---
title: Create SQL Server Windows VM in the Azure portal | Microsoft Docs
description: This tutorial shows how to create a Windows SQL Server 2017 virtual machine in the Azure portal.
services: virtual-machines-windows
documentationcenter: na
author: rothja
manager: jhubbard
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: infrastructure-services
ms.date: 12/05/2017
ms.author: jroth
---

# Create a SQL Server 2017 Windows virtual machine in the Azure portal

> [!div class="op_single_selector"]
> * [Windows](quickstart-sql-vm-create-portal.md)
> * [Linux](../../linux/sql/provision-sql-server-linux-virtual-machine.md)

This quickstart steps through creating a SQL Server virtual machine in the Azure portal.

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

## <a id="select"></a> Select a SQL VM image from the gallery

1. Log in to the [Azure portal](https://portal.azure.com) using your account.

1. On the Azure portal, click **New**. The portal opens the **New** window.

1. In the search field, type **SQL Server 2017 Developer on Windows Server 2016**, and press ENTER.

1. Select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server 2016** image.

   ![New search window](./media/quickstart-sql-vm-create-portal/newsearch.png)

   > [!TIP]
   > The Developer edition is used in this tutorial because it is a full-featured edition of SQL Server that is free for development testing purposes. You pay only for the cost of running the VM. For complete pricing considerations, see [Pricing guidance for SQL Server Azure VMs](virtual-machines-windows-sql-server-pricing-guidance.md).

1. Click **Create**.

## <a id="configure"></a> Provide basic details

On the **Basics** window, provide the following information:

1. In the **Name** field, enter a unique virtual machine name. 

1. In the **User name** field, enter a name for the local administrator account on the VM.

1. Provide a strong **Password**.

1. Enter a new **Resource group** name. This group helps to manage all resources associated with the virtual machine.

1. Verify the other default settings, and click **OK** to continue.

   ![SQL Basics window](./media/quickstart-sql-vm-create-portal/azure-sql-basic.png)

## Choose virtual machine size

On the **Size** step, choose a virtual machine size in the **Choose a size** window. The window initially displays recommended machine sizes based on the image you selected. 

1. Click **View all** to see all available machine sizes.

1. For this quickstart, select **D2S_V3**.

   Note the estimated monthly machine cost for continuous use. This is only the cost of the VM and not the [SQL Server VM licensing costs](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). But for this quickstart, using the Developer Edition means that there is no additional SQL Server licensing costs.

   > [!TIP]
   > This machine size saves money during testing. But for production workloads, see the recommended machine sizes and configuration in [Performance best practices for SQL Server in Azure Virtual Machines](virtual-machines-windows-sql-performance.md).

1. Click **Select** to continue.

## Configure optional features

On the **Settings** window, click **OK** to select the defaults.

## SQL Server settings

On the **SQL Server settings** window, configure the following options.

1. In the **SQL connectivity** drop-down, select **Public (Internet)**. This allows SQL Server connections over the internet.

1. Under **SQL Authentication**, click **Enable**. The SQL Login is set to the same user name and password that you configured for the VM.

1. Change any other settings if needed, and click **OK** to complete the configuration of the SQL Server VM.

   ![SQL Server settings](./media/quickstart-sql-vm-create-portal/sql-settings.png)

## Review the summary

On the **Summary** window, review the summary and click **Purchase** to create SQL Server, resource group, and resources specified for this VM.

You can monitor the deployment from the Azure portal. The **Notifications** button at the top of the screen shows basic status of the deployment.

> [!TIP]
> Deploying a Windows SQL Server VM can take several minutes.

## <a id="remotedesktop"></a> Open the VM with Remote Desktop

Use the following steps to connect to the SQL Server virtual machine with Remote Desktop:

> [!INCLUDE [Connect to SQL Server VM with remote desktop](../../../../includes/virtual-machines-sql-server-remote-desktop-connect.md)]

After you connect to the SQL Server virtual machine, you can launch SQL Server Management Studio and connect with Windows Authentication using your local administrator credentials. If you enabled SQL Server Authentication, you can also connect with SQL Authentication using the SQL login and password you configured during provisioning.

Access to the machine enables you to directly change machine and SQL Server settings based on your requirements. For example, you could configure the firewall settings or change SQL Server configuration settings.

## Enable TCP/IP for internet connections

When provisioning a new SQL Server VM, Azure does not automatically enable the TCP/IP protocol for SQL Server Developer and Express editions. The steps below explain how to manually enable TCP/IP so that you can connect remotely by IP address.

The following steps use **SQL Server Configuration Manager** to enable the TCP/IP protocol for SQL Server Developer and Express editions.

> [!INCLUDE [Connect to SQL Server VM with remote desktop](../../../../includes/virtual-machines-sql-server-connection-tcp-protocol.md)]

### Connect to SQL Server remotely

1. In the portal, find the **Public IP address** of your VM in the **Overview** section of your virtual machine's properties.

1. On a different computer connected to the Internet, open SQL Server Management Studio (SSMS).

   > [TIP] 
   > If you do not have SQL Server Management Studio, you can download it [here](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms).

1. In the **Connect to Server** or **Connect to Database Engine** dialog box, edit the **Server name** value. Enter your VM's public IP address.

1. In the **Authentication** box, select **SQL Server Authentication**.

1. In the **Login** box, type the name of a valid SQL login.

1. In the **Password** box, type the password of the login.

1. Click **Connect**.

    ![ssms connect](./media/quickstart-sql-vm-create-portal/ssms-connect.png)

## Next Steps

In this quickstart, you created a SQL Server 2017 virtual machine using Azure PowerShell. To learn more about how to migrate your data to the new SQL Server, see the following article.

> [!div class="nextstepaction"]
> [Migrate a database to a SQL VM](virtual-machines-windows-migrate-sql.md)
