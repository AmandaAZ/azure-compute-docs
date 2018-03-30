---
title: SAP HANA availability within one Azure region | Microsoft Docs
description: Describes the operations of SAP HANA on Azure native VMs.
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''

ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/05/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017

---

# SAP HANA availability within one Azure region
This article describes several availability scenarios within one Azure region. Azure has many regions. Azure regions are spread throughout the world. For the list of Azure regions, see [Azure regions](https://azure.microsoft.com/regions/). For deploying SAP HANA on VMs within one Azure region, Microsoft offers the deployment of a single VM with a HANA instance. For increased availability, you can deploy two VMs with two HANA instances within an [Azure availability set](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-availability-sets) that uses HANA system replication for availability. 

Currently, Azure is offering a public preview of [Azure availability zones (preview)](https://docs.microsoft.com/azure/availability-zones/az-overview). This article doesn't describe availability zones in detail. But, it does include a general discussion about using availability sets versus availability zones.

What is the difference between an availability set and an availability zone in Azure? Azure regions where availability zones are offered have multiple datacenters. The datacenters are independent in the supply of power source, cooling, and network. The reason for offering different zones within a single Azure region is so that you can deploy applications across two or three availability zones that are offered. Assuming that power source or network issues would affect only one availability zone infrastructure, your application deployment within an Azure region is still fully functional if you use availability zones. This would be the case eventually, with some reduced capacity. Some VMs in one zone might still be lost. But, VMs in the other two zones would still be up and running. 
 
An Azure availability set is a logical grouping capability that you can use in Azure to ensure that the VM resources you place within the availability set are failure-isolated from each other when they are deployed within an Azure datacenter. Azure ensures that the VMs you place within an availability set run across multiple physical servers, compute racks, storage units, and network switches. In some Azure documentation, this configuration is referred to as placements in different [update and fault domains](https://docs.microsoft.com/azure/virtual-machines/windows/manage-availability). These placements usually are within an Azure datacenter. Assuming that power source and network issues would affect the datacenter that you are deploying, all your capacity in one Azure region would be affected.

The placement of datacenters that represent Azure availability zones is a compromise between delivering network latency between services deployed in different zones that's acceptable for most applications, and a specific distance between datacenters. Natural catastrophes ideally wouldn't affect the power and network supply and infrastructure for all availability zones in this region. However, as monumental natural catastrophes have shown, availability zones might not always be able to provide the availability that you want within one region. Think about Hurricane Maria that hit the island of Puerto Rico on August 20, 2017. The hurricane basically caused a near 100 percent blackout on the 90-mile-wide island.

## Single-VM scenario

In a single-VM scenario, you create an Azure virtual machine for the SAP HANA instance. You use Azure Premium Storage to host the operating system disk and all your data disks. The Azure uptime SLA of 99.9% and the SLAs of other Azure components is sufficient for you to fulfill your availability SLAs for your customers. In this scenario, you have no need to leverage an Azure availability set for VMs that run the DBMS layer. In this scenario, you rely on two different features:

- Azure VM auto restart (also referred to as Azure service healing)
- SAP HANA auto-restart

Azure VM auto restart, or service healing, is a functionality in Azure that works on two levels:

- The Azure server host checks the health of a VM that's hosted on the server host.
- Azure Fabric Controller monitors the health and availability of the server host.

A health check functionality monitors the health of every VM that's hosted on an Azure server host. If a VM falls into a non-healthy state, a reboot of the VM can be initiated by the Azure host agent that checks the health of the VM. Fabric Controller checks the health of the host by checking many different parameters that might indicate issues with the host hardware. It also checks on the accessibility of the host via the network. An indication of problems with the host can lead to the following actions:

- If the host signals a bad health state, reboot the host and restart the VMs that were running on the host.
- If the host is not in a healthy state after the reboot, reboot the host, and restart the VM(s) that were originally hosted on the host on a healthy host. In this case, the host is going to be marked as not healthy. It won't be used for further deployments until it's cleared or replaced.
- If the unhealthy host has problems during the reboot process, immediate restart of the VMs on a healthy host. 

With the host and VM monitoring provided by Azure, Azure VMs that experience host issues are automatically restarted on a healthy Azure host. 

The second feature that you rely on in this scenario is the fact that your HANA service that runs in a restarted VM starts automatically after the VM reboots. You can set up [HANA service auto-restart](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/cf10efba8bea4e81b1dc1907ecc652d3.html) through the watchdog services of the various HANA services.

This single-VM scenario might be improved by adding a cold failover node to an SAP HANA configuration. In the SAP HANA documentation, this setup is called [host auto-failover](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/ae60cab98173431c97e8724856641207.html). This configuration might make sense in an on-premises deployment situation where the server hardware is limited, and you dedicate a single server node as the host auto-failover node for a set of production hosts. But in Azure, where the underlying infrastructure of Azure provides a healthy target server for a successful VM restart, it doesn't make sense to deploy SAP HANA host auto-failover. Because of this, we have no reference architecture that foresees a standby node for HANA host auto-failover. This also applies for SAP HANA scale-out configurations.

## Availability scenarios for two different VMs

If you use two Azure VMs within an Azure availability set, you can increase the up-time between these two VMs if those VMs are placed in an Azure availability set within one Azure region. The base setup in Azure would look like this:

![Diagram of two VMs with all layers](./media/sap-hana-availability-one-region/two_vm_all_shell.PNG)

To illustrate the different availability scenarios, a few of the layers in the diagram are omitted. The diagram shows only layers that depict VMs, hosts, availability sets, and Azure regions. Azure VNets, resource groups, and subscriptions don't play a role in the scenarios described in this section.

### Replicate backups to second virtual machine

One of the most rudimentary setups is to have backups. In particular, you might have transaction log backups shipped from one VM to another Azure VM. You can choose the Azure Storage type. In this setup, you are responsible for scripting the copy of scheduled backups that are conducted on the first VM to the second VM. If you need to use the second VM instances, you must restore the full, incremental/differential, and transaction log backups to the point that you need. 

The architecture looks like this:

![Diagram of two VMs with storage replication](./media/sap-hana-availability-one-region/two_vm_storage_replication.PNG) 

This setup is not well suited for achieving great Recovery Point Objective (RPO) and Recovery Time Objective (RTO) times. RTO times especially would suffer due to the need to fully restore the complete database with the copied backups. However, this setup is useful to recover from unintended data deletion on the main instances. With this setup, at any time you can restore to a certain point in time, extract the data, and import the deleted data into your main instance. Hence, it might make sense to use a backup copy method in combination with other high-availability functionality. While  backups are being copied, you might be able to use a smaller VM than the main VM that the SAP HANA instance is running on. Keep in mind that smaller VMs have a lower number of VHDs that can be attached. For information about the limits of individual VM types, see [Sizes for Linux virtual machines in Azure](https://docs.microsoft.com/azure/virtual-machines/linux/sizes).

### SAP HANA system replication without automatic failover

The scenarios described in this section use SAP HANA system replication. For SAP documentation, see [System replication](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/b74e16a9e09541749a745f41246a065e.html). Two Azure VMs in a single Azure region have different configurations, so there are some differences in RTO. In general, scenarios without automatic failover might not apply specifically to VMs in one Azure region. This is because for most failures in the Azure infrastructure, the Azure service healing restarts the primary VM on another host. There are some edge cases where this configuration might help in terms of failure scenarios. Or, in some cases, you as a customer might want to realize more around efficiency.

#### SAP HANA system replication without auto failover and without data preload

In this scenario, you use SAP HANA system replication to move data in a synchronous manner to achieve an RPO of 0. On the other hand, you have a long enough RTO so that you don't need either failover or preloading the data into the HANA instance cache. In this case, it's possible to achieve further economy in your configuration by taking the following actions:

- You can run another SAP HANA instance in the second VM. The SAP HANA instance in the second VM takes most of the memory of the virtual machine. Usually, this is in the event of a failover to the second VM. The second VM can be shut down so that the replicated data can be loaded into the cache of the targeted HANA instance in the second VM.
- You can use a smaller VM size on the second VM. In case of a failover, you'd have a step before the manual failover where you would resize the VM to the size of the source VM. The scenario looks like this:

![Diagram of two VMs with storage replication](./media/sap-hana-availability-one-region/two_vm_HSR_sync_nopreload.PNG)

> [!NOTE]
> Even if you don't use data preload in the HANA system replication target, you need at least 64 GB of memory. You need enough memory in addition to 64 GB to keep the rowstore data in the memory of the target instance.

#### SAP HANA system replication without auto failover and with data preload

This scenario differs from the preceding scenario in that data that is replicated to the HANA instance in the second VM is preloaded. This eliminates the two advantages of not preloading data. In this case, you can't run another SAP HANA system on the second VM. You also can't use a smaller VM size. Hence, customers rarely implement this scenario.

### SAP HANA system replication with automatic failover

In the standard and most common availability configuration within one Azure region, two Azure VMs running SLES Linux have a failover cluster defined. The Linux cluster of SLES is based on the [Pacemaker](http://www.linux-ha.org/wiki/Pacemaker) framework, in conjunction with a [STONITH](http://linux-ha.org/wiki/STONITH) device. 

From an SAP HANA perspective, the replication mode that's used is synchronized and an automatic failover is configured. In the second VM, the SAP HANA instance acts as a hot standby node. The standby node receives a synchronous stream of change records from the primary SAP HANA instance. As transactions are committed by the application at the HANA primary node, the primary HANA node waits to confirm the commit to the application until the secondary SAP HANA node confirms that it received the commit record. SAP HANA offers two synchronous replication modes. For details and for a description of differences between these two synchronous replication modes, see the SAP article [Replication modes for SAP HANA system replication](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.02/en-US/c039a1a5b8824ecfa754b55e0caffc01.html)

The overall configuration looks like this:

![Diagram of two VMs with storage replication and failover](./media/sap-hana-availability-one-region/two_vm_HSR_sync_auto_pre_preload.PNG)

You might choose this solution because it enables you to achieve an RPO=0 and an extremely low RTO. Configure the SAP HANA client connectivity so that the SAP HANA clients use the virtual IP address to connect to the HANA system replication configuration. This eliminates the need to reconfigure the application if a failover to the secondary node occurs. In this scenario, the Azure VM SKUs for the primary and secondary VMs must be the same.

## Next steps

For step-by-step guidance on setting up these configurations in Azure, see these articles:

- [Set up SAP HANA system replication in Azure VMs](sap-hana-high-availability.md)
- [High availability for SAP HANA by using system replication](https://blogs.sap.com/2018/01/08/your-sap-on-azure-part-4-high-availability-for-sap-hana-using-system-replication/)

For more information about SAP HANA availability across Azure regions, see:

- [SAP HANA availability across Azure regions](https://docs.microsoft.com/en-us/azure/virtual-machines/workloads/sap/sap-hana-availability-across-regions) 

