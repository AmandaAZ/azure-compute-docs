---
title: Use Azure Site Recovery to move a SQL Server virtual machine to another region within Azure | Microsoft Docs
description: Learn how you can migrate your SQL Server virtual machine from one region to another within Azure.  
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
manager: jroth
tags: azure-resource-manager
ms.assetid: aa5bf144-37a3-4781-892d-e0e300913d03
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/25/2019
ms.author: mathoma
ms.reviewer: jroth

---
# Use Azure Site Recovery to move a SQL Server VM to another region within Azure

This article teaches you how to use Azure Site Recovery to migrate your SQL Server virtual machine (VM) from one region to another within Azure. 

## Verify the prerequisites 

- Confirm that moving from your source region to your target region [is supported](../../../site-recovery/azure-to-azure-support-matrix.md#region-support).  
- Review the [scenario architecture and components](../../../site-recovery/azure-to-azure-architecture.md) as well as the [support limitations and requirements](../../../site-recovery/azure-to-azure-support-matrix.md). 
- Verify account permissions. If you created your free Azure account, you're the administrator of your subscription. If you're not the subscription administrator, work with the administrator to assign the permissions that you need. To enable replication for a VM and copy data using Azure Site Recovery, you must have: 
    - Permissions to create a VM. The *Virtual Machine Contributor* built-in role has tehse permissions, which cinlude: 
        - Permissions to create a VM in the selected resource group. 
        - Permissions to create a VM in the selected virtual network. 
        - Permissions to write to the selected storage account. 
      - Permissions to manage Azure Site Recovery operations. The *Site Recovery Contributor* role has all the permissions that are required to manage Site Recovery operations in a Recovery Services vault.  

## Prepare to move

### Prepare the source SQL Server VM

- Ensure that all the latest root certificates are on the SQL Server VM taht you want to move. If the latest root certificates are not there, security constraints will prevent data copy to the target region. 
- For Windows VMs, install all of the latest Windows updates on the VM, so that all the trusted root certificates are on the machine. In a disconnected environment, follow the standard Windows UPdate and certificate update process for your orginization. 
- For Linux VMs, follow the guidance provided by your Linux distributor to get the latest trusted root certificates and certificate revocation list on the VM. 
- Make sure you're not using an authentication proxy to control network connectivity for the VMs that you want to move. 
- If the VM that you're trying to move doesn't have access to the internet, or it's using a firewall proxy to control outbound access, check the reuqirements. 
- Identify the source networking layout and all the resources that your'e currently using. This includes but isn't limited to load balancers, network security groups (NSGs), and public IPs. 

### Prepare the target SQL Server VM 

- Verify that your Azure subscription allows you to create VMs in the target region that's used for disaster recovery. Contact support to enable the required quota. 
- Make sure that your subscription has enough resources to support VMs with size that match your source VMs. If you're using Site Recovery to copy data to the target, site Recovery chooses the same size, or the closest possible size for the target VM. 
- Make sure that you create a target resource for every component that's identified in the source networking layout. This step is important to ensure that your VMs have all the functionality and features in the target region that you had in the source region. 
    - Azure Site Recovery automatically discovers and creates a virtual network when you enable replication for the source VM. You can also pre-create a network and assign it to the VM in the user flow for enabling replication. You need to manually create any other resources in the target region.
- To create the most commonly used network resources that are relevant for you based on the source VM configuration, see the following documentation: 
    - [Network security groups](../../../virtual-network/tutorial-filter-network-traffic.md) 
    - [Load balancer](../../../load-balancer/tutorial-load-balancer-basic-internal-portal.md)
    - [Public IP address](../../../virtual-network/virtual-network-public-ip-address.md)
    - For any additional networking components, see the [networking documentation](../../../virtual-network/virtual-networks-overview.md).
- Manually create a non-production network in the target region if you want to test the configuration before you perform the final move to the target region. We recommend this step because it ensures minimal interference with the production network. 

## Copy data to the target region 

The following steps show you how to use Azure Site Recovery to copy data to the target region. Create the Recovery Services vault in any region other than the source region. 

1. Sign into the [Azure portal](https://portal.azure.com). 
1. Choose to **Create a resource** from the upper-left hand corner of the navigation pane. 
1. Select **IT & Management tools** and then select **Backup and Site recovery**. 
1. On the **Basics** tab, under **Project details**, either create a new resource group in the target region, or select an existing resource group in the target region. 
1. Under **Instance Details**, specify a name for your vault, and then select your target **Region** from the drop-down. 
1. Select **Review + Create** to create your Recovery Services vault. 
1. Select **All services** from the upper-left hand corner of the navigation pane and in the search box type `recovery services`. 
1. (Optionally) Select the star next to **Recovery Services vaults** to add it to your quick navigation bar. 
1. Select **Recovery services vaults** and then select the Recovery Services vault you created. 
1. On the **Overview** pane, select **Replicate**. 
1. Select **Source** and then select **Azure** as the source. Select the appropriate values for the other drop-down fields, such as the location for your source VMs. Only resources groups located in the **Source location** region will be visible in the **Source resource group** field. 
1. Select **Virtual machines** and then choose the virtual machines you want to migrate. Select **OK** to save your VM selection. 
1. Select **Settings**, and then choose your **Target location** from the drop-down. You can leave the rest of the fields default, or you can customize other settings here too, such as a different subscription, or a specific resource group. By default, a new resource group is created in the target region with `-asr` appended to your existing resource group name. 
1. Once you have customized replication, select **Create target resources** to create the resources in the new location. 
1. Once resource creation is complete, select **Enable replication** to start replication your SQL Server VM from the source to the target region. 

## Test the move process
The following steps show you how to use Azure Site Recovery to test the move process. 

1. Navigate to your **Recovery Services vault** in the [Azure portal](https://portal.azure.com) and select **Replicated items**. 
1. Select the SQL Server VM you would like to move, verify that the **Replication Health** shows as **Healthy** and then select **Test Failover**. 

  ![Test failover for your VM](media/virtual-machines-windows-sql-move-to-new-region/test-failover-of-replicated-vm.png)

1. Select the virtual network under **Azure virtual network** and then select **OK** to test failover. 

## Next steps

For more information, see the following articles: 

* [Overview of SQL Server on a Windows VM](virtual-machines-windows-sql-server-iaas-overview.md)
* [SQL Server on a Windows VM FAQ](virtual-machines-windows-sql-server-iaas-faq.md)
* [SQL Server on a Windows VM pricing guidance](virtual-machines-windows-sql-server-pricing-guidance.md)
* [SQL Server on a Windows VM release notes](virtual-machines-windows-sql-server-iaas-release-notes.md)


