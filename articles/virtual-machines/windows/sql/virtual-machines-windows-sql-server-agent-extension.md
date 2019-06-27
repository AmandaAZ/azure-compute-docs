---
title: Automate management tasks on Azure Virtual Machines with the SQL Server IaaS Agent Extension | Microsoft Docs
description: This article describes how to manage the SQL Server agent extension, which automates specific SQL Server administration tasks. These include Automated Backup, Automated Patching, and Azure Key Vault Integration.
services: virtual-machines-windows
documentationcenter: ''
author: MashaMSFT
manager: jroth
editor: ''
tags: azure-resource-manager
ms.assetid: effe4e2f-35b5-490a-b5ef-b06746083da4
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/24/2019
ms.author: mathoma
ms.reviewer: jroth
---
# Automate management tasks on Azure Virtual Machines with the SQL Server IaaS Agent Extension
> [!div class="op_single_selector"]
> * [Resource Manager](virtual-machines-windows-sql-server-agent-extension.md)
> * [Classic](../sqlclassic/virtual-machines-windows-classic-sql-server-agent-extension.md)

The SQL Server IaaS Agent Extension (SqlIaasExtension) runs on Azure virtual machines to automate administration tasks. This article provides an overview of the services supported by the extension as well as instructions for installation, status, and removal.

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

To view the classic version of this article, see [SQL Server Agent Extension for SQL Server VMs Classic](../sqlclassic/virtual-machines-windows-classic-sql-server-agent-extension.md).

## Supported services
The SQL Server IaaS Agent Extension supports the following administration tasks:

| Administration feature | Description |
| --- | --- |
| **SQL Automated Backup** |Automates the scheduling of backups for all databases for either the default instance or a [properly installed](virtual-machines-windows-sql-server-iaas-faq.md#administration) named instance of SQL Server on the VM. For more information, see [Automated backup for SQL Server in Azure Virtual Machines (Resource Manager)](virtual-machines-windows-sql-automated-backup.md). |
| **SQL Automated Patching** |Configures a maintenance window during which important Windows updates to your VM can take place, so  you can avoid updates during peak times for your workload. For more information, see [Automated patching for SQL Server in Azure Virtual Machines (Resource Manager)](virtual-machines-windows-sql-automated-patching.md). |
| **Azure Key Vault Integration** |Enables you to automatically install and configure Azure Key Vault on your SQL Server VM. For more information, see [Configure Azure Key Vault Integration for SQL Server on Azure VMs (Resource Manager)](virtual-machines-windows-ps-sql-keyvault.md). |

Once installed and running, the SQL Server IaaS Agent Extension makes these administration features available on the SQL Server panel of the virtual machine in the Azure portal and through Azure PowerShell for SQL Server marketplace images, and through Azure PowerShell for manual installations of the extension. 

## Prerequisites
Requirements to use the SQL Server IaaS Agent Extension on your VM:

**Operating System**:

* Windows Server 2008 R2
* Windows Server 2012
* Windows Server 2012 R2
* Windows Server 2016
* Windows Server 2019 

**SQL Server versions**:

* SQL Server 2008 
* SQL Server 2008 R2
* SQL Server 2012
* SQL Server 2014
* SQL Server 2016
* SQL Server 2017

**Azure PowerShell**:

* [Download and configure the latest Azure PowerShell commands](/powershell/azure/overview)

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

> [!IMPORTANT]
> At this time, the [SQL Server IaaS Agent Extension](virtual-machines-windows-sql-server-agent-extension.md) is not supported for SQL Server FCI on Azure. We recommend that you uninstall the extension from VMs that participate in an FCI. The features supported by the extension are not available to the SQL VMs after the agent is uninstalled.

## Modes
There are two mode types for the SQL IaaS extension: **Full** and **Lightweight**. 

- **Full** mode delivers all functionality, but requires a restart of the SQL Server and SA permissions. This is the option that is installed by default.
- **Lightweight** does not require the restart of SQL Server, but only supports changing the license type and edition of SQL Server. 


It is possible to upgrade from **Lightweight**, to **Full**, but it is not possible to downgrade. To downgrade, you will need to completely uninstall the SQL IaaS extension and install it again. Upgrading to **Full** mode will restart the SQL Server service. 

You can view the current mode of your SQL IaaS agent from the Azure portal, or by using PowerShell: 


  ```powershell-interactive
     //Get the SqlVirtualMachine
     $sqlvm = Get-AzResource -Name $vm.Name  -ResourceGroupName $vm.ResourceGroupName  -ResourceType Microsoft.SqlVirtualMachine/SqlVirtualMachines
     $sqlvm.Properties 
  ```



## Installation
The SQL Server IaaS Agent Extension is automatically installed when you provision one of the SQL Server virtual machine gallery images. The SQL IaaS extension offers manageability for a single instance on the SQL Server VM. If there is a default instance, then the extension will work with the default instance, and it will not support managing other instances. If there is no default instance but only one named instance, then it will manage the named instance. If there is no default instance and there are multiple named instances, then the extension will fail to install. 

If you need to reinstall the extension manually on one of these SQL Server VMs, use the following PowerShell command:

```powershell
Set-AzVMSqlServerExtension -ResourceGroupName "resourcegroupname" -VMName "vmname" -Name "SqlIaasExtension" -Version "2.0" -Location "East US 2"
```

Install SQL IaaS agent with lightweight mode using PowerShell:

  ```powershell-interactive
     // Get the existing  Compute VM
     $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
          
     // Create a new SQL VM with the 'Lightweight' SQL IaaS agent
     New-AzResource -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
        -ResourceType Microsoft.SqlVirtualMachine/SqlVirtualMachines `
        -Properties @{virtualMachineResourceId=$vm.Id;sqlManagement='LightWeight'} -Force 
  
  ```

> [!WARNING]
> If the extension is not already installed, installing the **Full** extension restarts the SQL Server service. However, using the **Lightweight** mode, and updating the SQL IaaS extension does not restart the SQL Server service. 

> [!NOTE]
> While it is possible to install the SQL Server IaaS Agent extension to custom SQL Server images, the functionality is currently limited to [changing the license type](virtual-machines-windows-sql-ahb.md). Other features provided by the SQL IaaS extension will only work on [SQL Server VM gallery images](virtual-machines-windows-sql-server-iaas-overview.md#get-started-with-sql-vms) (pay-as-you-go or bring-your-own-license).

### Use a single named instance
The SQL IaaS extension will work with a named instance on a SQL Server image if the default instance is uninstalled properly, and if the IaaS extension is reinstalled.

To use a named instance of SQL Server, do the following:
   1. Deploy a SQL Server VM from the marketplace. 
   1. Uninstall the IaaS extension from within the [Azure portal](https://portal.azure.com).
   1. Uninstall SQL Server completely within the SQL Server VM.
   1. Install SQL Server with a named instance within the SQL Server VM. 
   1. Install the IaaS extension from within the Azure portal.  

## Status
One way to verify that the extension is installed is to view the agent status in the Azure portal. Select **All settings** in the virtual machine window, and then click on **Extensions**. You should see the **SqlIaasExtension** extension listed.

![SQL Server IaaS Agent Extension in Azure portal](./media/virtual-machines-windows-sql-server-agent-extension/azure-rm-sql-server-iaas-agent-portal.png)

You can also use the **Get-AzVMSqlServerExtension** Azure PowerShell cmdlet.

   ```powershell-interactive
   Get-AzVMSqlServerExtension -VMName "vmname" -ResourceGroupName "resourcegroupname"
   ```

The previous command confirms the agent is installed and provides general status information. You can also get specific status information about Automated Backup and Patching with the following commands.

   ```powershell-interactive
    $sqlext = Get-AzVMSqlServerExtension -VMName "vmname" -ResourceGroupName "resourcegroupname"
    $sqlext.AutoPatchingSettings
    $sqlext.AutoBackupSettings
   ```

## Removal
In the Azure portal, you can uninstall the extension by clicking the ellipsis on the **Extensions** window of your virtual machine properties. Then click **Delete**.

![Uninstall the SQL Server IaaS Agent Extension in Azure portal](./media/virtual-machines-windows-sql-server-agent-extension/azure-rm-sql-server-iaas-agent-uninstall.png)

You can also use the **Remove-AzVMSqlServerExtension** PowerShell cmdlet.

   ```powershell-interactive
    Remove-AzVMSqlServerExtension -ResourceGroupName "resourcegroupname" -VMName "vmname" -Name "SqlIaasExtension"
   ```

## Next steps
Begin using one of the services supported by the extension. For more information, see the articles referenced in the [Supported services](#supported-services) section of this article.

For more information about running SQL Server on Azure Virtual Machines, see [SQL Server on Azure Virtual Machines overview](virtual-machines-windows-sql-server-iaas-overview.md).

