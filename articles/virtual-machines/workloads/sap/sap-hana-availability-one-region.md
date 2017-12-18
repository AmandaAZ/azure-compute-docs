title: SAP HANA Availability within one Azure Region | Microsoft Docs
description: Operations of SAP HANA on Azure native VMs
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: juergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''

ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 11/17/2017
ms.author: msjuergent
ms.custom: H1Hack27Feb2017

---

# Availability within one Azure Region
In this section we are working through several scenarios which describes availability scenarios within one Azure Region. Azure has many regions which are spread all over the world. For the the list of Azure regions consult the [Azure Regions](https://azure.microsoft.com/en-us/regions/) article. Deploying SAP HANA on VMs within one Azure Region, Microsoft offers the deployment of a single VM with a HANA instance. Or for increased availability you can deploy two VMs with two HANA instances within an Azure Availability set that are using HANA System Replication for availability purposes. Though Azure has a public preview of [Azure Availability Zones](https://docs.microsoft.com/en-us/azure/availability-zones/az-overview), parts that are necessary for a complete availability scenario are still missing and hence will not be described as a valid scenario in this guide yet.

What is the difference between Azure Availability sets and Availability Zones? For Azure Regions where Availability Sets are offered, the regions will have multiple data centers which can be in a distance far enough to be considered as valid disaster recovery location. Hence the Azure data centers would be grouped into so called 'Availability Zones'. You as a customer are going to have the possibility to deploy into a specific Availability Zone. Think Azure Availability Sets as a grouping which works either in Azure Region that do not support Availability Zones or a construct within an Azure Availability Zone where you can make sure that two or multiple VMs are getting deployed into different fault and update zones, so, that not all VMs are affected by eventual issues or updates to host nodes. So far for SAP HANA, we will focus on deployment scenarios in Availability Sets only.   


## Single VM scenario
In this scenario you have created an Azure Virtual machine for the SAP HANA instance. You used Azure Premium Storage for hosting the operating System disk and all the data disks. The up-time SLA of 99.9% by Azure and the SLAs of other Azure components is sufficient for you to fulfill your availability SLAs towards your customers. In this scenario, you will not have the need to leverage an Azure Availability Set for VMs that run the DBMS layer. In this scenario you rely on two different features:

- Azure VM Auto Restart (also referenced as Azure Service Healing)
- SAP HANA Auto-Restart

Azure VM Auto Restart or 'service healing' is a functionality in azure that works on two levels:

- Azure server host checking the health of a VM hosted on the server host
- Azure Fabric Controller monitoring the health and availability of the server host

For every VM hosted on an Azure server host, a health check functionality is monitoring the health of the hosted VM(s). In case VMs fall into a non-healthy state a reboot of the VM can be initiated by the host agent that checks the health of the VM. The Azure Fabric Controller is checking the health of the host by checking many different parameters indicating issues with the host hardware, but also checks on the accessibility of host via the network. An indication of problems with the host can lead to actions like:

- Reboot of the host and restart of the VMs that were running on the host if the host reaches a health state
- Reboot of the host and restart of the VM(s) that were originally hosted on the host on a healthy host in case the host is not in a healthy state after the reboot. In this case the host will be marked as not healthy an not used for further deployments until cleared or replaced.
- Immediate restart of the VMs on a healthy host in cases where the unhealthy host has problems in the reboot process. 

With the host and VM monitoring provided by Azure, Azure VMs that suffer on host issues issues are Auto Restarted on a healthy Azure host 

The second feature you rely on in such a scenario is the fact that your HANA service that runs within such a restarted VM is starting automatically after the reboot of the VM. The [HANA Service Auto-Restart](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/cf10efba8bea4e81b1dc1907ecc652d3.html) can be configured through the watchdog services of the different HANA services.

This single VM scenario could get improved by adding a cold failover node to a SAP HANA configuration. Or as it is called out in the SAP HANA documentation as [Host Auto-Failover](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/ae60cab98173431c97e8724856641207.html). this scenario can make sense in on-premise deployment situations where the server hardware is limited and you dedicate a single server node as Host Auto-Failover node for a set of production hosts. However for situations like Azure where the underlying infrastructure of Azure is providing a healthy target server for a successful restart of a VM, the SAP HANA Host Auto-Failover scenario does not make sense to deploy. 

Beyond that, Azure does not provide NFS storage that would be able to the storage latency requirements of high-end DBMS with SAP workload. hence the SAP HANA Host Auto-Failover as described by SAP can't be deployed.


## Availability Scenarios involving two different VMs
Using two Azure VMs within Azure Availability Sets enable you to increase the up-time between these two VMs if those are placed in an Azure Availability Set within one Azure region. The base setup in Azure would look like the graphics shown here:
![Two VMs with all layers](./media/sap-hana-availability-one-region/two_vm_all_shell.PNG)

In order to illustrate the different availability scenarios, we will leave a few of the layers above away and only will limit the graphics to the layers of VMs, hosts, Availability Sets and Azure Regions. Azure VNets , Resource Groups and subscriptions don't play a role for the following scenarios.

### Replicating Backups to second Virtual Machine
One of the most rudimentary setups is to have backups, especially transaction log backups shipped from one VM to another Azure Virtual machine. You have the choice of any Azure Storage type. You would be responsible for scripting the copy of scheduled backups conducted on the first VM to the second VM. You also would need to be responsible for scheduling and scripting a regular restore of the (transaction log) backups to the HANA instance in the second Azure VMs.

The architecture would look like:
![Two VMs with storage replication](./media/sap-hana-availability-one-region/two_vm_storage_replication.PNG) 

Advantage of this scenario is that you can restore back to a certain point in time in case you need to recover specific data within a specific table that got deleted by a user or administrator error. The disadvantage is that you need to have a full VM running and that you need to script quite a lot around a solution like this.

### Using SAP HANA System Replication without Automatic Failover
For the following scenarios you are using SAP HANA System replication. SAP issued documentation can be found starting with the article [System Replication](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/b74e16a9e09541749a745f41246a065e.html). Between two Azure VMs in a single Azure region we have two different configurations that have some differences in the Recovery Time Objective. In general, the scenarios with not having automatic failover might not too relevant for scenarios within one region. Reason for that is that in most failure situations in Azure infrastructure, the Azure Service healing is going to restart the primary VM on another host. There are only some edge cases where such a configuration might help in terms of failure scenarios. Or some cases you as a customer want to realize, especially around efficiency.

#### Using HANA System Replication without auto failover and without data pre-load 
This is a scenario where you use SAP HANA System Replication for the purpose of moving data in a synchronous manner to achieve a Recovery Point Objective (RPO) of 0. On the other side, you have a long enough Recovery Time Objective (RTO), so, that you don't need either failover or pre-load of the data into the HANA instance cache. In such a case, you have the possibilities to drive further economics into your configuration by:

- You can run another SAP HANA instance in the second VM that takes most of the memory of the virtual machine. Usually such an instance would be an instance that in case of a fail-over to the second VM could be shut down. So, that the replicated data can be loaded into the cache of the targeted HANA instance in the second VM.
- You could use a smaller VM size as the second VM. In case of a failover, you'd have a step before the manual failover where you would re-size the VM to the size of the source VM. The scenario basically looks like:

![Two VMs with storage replication](./media/sap-hana-availability-one-region/two_vm_HSR_sync_nopreload.PNG)

#### Using HANA System Replication without auto failover and with data pre-load
The difference to the scenario introduced before is that the data which gets replicated to the HANA Instance in the second VM is pre-loaded. This would eliminate the two advantages you can have with the scenario




 










 