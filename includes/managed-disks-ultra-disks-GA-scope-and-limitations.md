---
title: include file
description: include file
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 04/08/2020
ms.author: rogarana
ms.custom: include file
---
For now, ultra disks have additional limitations, they are as follows:

The only infrastructure redundancy options currently available to ultra disks are availability zones, VMs using any other redundancy options cannot attach an ultra disk.  though ultra disks are not offered in every availability zone of these regions:

The following table outlines the available regions and their corresponding availability options:

|Regions  |No infrastructure Redundancy  |Availability zones  |
|---------|---------|---------|
|West US     |         |         |
|West US 2    |         |         |
|East US     |         |         |
|East US 2     |         |         |
|SouthEast Asia     |         |         |
|North Europe     |         |         |
|West Europe     |         |         |
|UK South     |         |         |

- Are only supported on the following VM series:
    - [ESv3](https://azure.microsoft.com/blog/introducing-the-new-dv3-and-ev3-vm-sizes/)
    - [DSv3](https://azure.microsoft.com/blog/introducing-the-new-dv3-and-ev3-vm-sizes/)
    - FSv2
    - [M](../articles/virtual-machines/workloads/sap/hana-vm-operations-storage.md)
    - [Mv2](../articles/virtual-machines/workloads/sap/hana-vm-operations-storage.md)
- Not every VM size is available in every supported region with ultra disks
- Are only available as data disks and only support 4k physical sector size. Due to the 4K native sector size of Ultra Disk, there are some applications that won't be compatible with ultra disks. One example would be Oracle Database, which requires release 12.2 or later in order to support ultra disks.  
- Can only be created as empty disks  
- Doesn't currently support disk snapshots, VM images, availability sets, Azure Dedicated Hosts, or Azure disk encryption
- Doesn't currently support integration with Azure Backup or Azure Site Recovery
- The current maximum limit for IOPS on GA VMs is 80,000.

Azure ultra disks offer up to 16 TiB per region per subscription by default, but ultra disks support higher capacity by request. To request an increase in account limits, contact Azure Support.