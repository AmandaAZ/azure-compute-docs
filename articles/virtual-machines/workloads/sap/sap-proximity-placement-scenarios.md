---
title: Azure proximity placement groups for optimal network latency with SAP applications | Microsoft Docs
description: Describes SAP deployment scenarios with Azure proximity placement groups
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''

ms.service: virtual-machines-linux

ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/15/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017

---

# Azure proximity placement groups for optimal network latency with SAP applications
SAP applications based on the SAP NetWeaver or SAP S/4HANA architecture are sensitive to network latency between the SAP application tier and the SAP database tier. This sensitivity is the result of most of the business logic running in the application layer. Because the SAP application layer executes the business logic, it issues queries to the database tier at a high frequency, at a rate of thousands or tens of thousands per second. In most cases, the nature of these queries is simple. They can often be run on the database tier in 500 microseconds or less.

The time spent on the network to send such a query from the application tier to the database tier and receive the result set back from the database tier has a major impact on the time it takes to run business processes. This sensitivity to network latency is why you need to achieve optimal network latency in SAP deployment projects. See [SAP Note #1100926 - FAQ: Network performance](https://launchpad.support.sap.com/#/notes/1100926/E) for guidelines on how to classify the network latency.

In many Azure regions, the number of datacenters has grown. This growth has also been triggered by the introduction of Availability Zones. At the same time, customers, especially for high-end SAP systems, are using more special VM SKUs in the M-Series family, or HANA Large Instances. These Azure virtual machine types aren't available in all the datacenters in a specific Azure region. Because of these two tendencies, customers have experienced situations in which network latency wasn't in the optimal range. In some cases, this latency resulted in suboptimal performance of their SAP systems.

To prevent these problems, Azure offers [proximity placement groups](https://docs.microsoft.com/azure/virtual-machines/linux/co-location). This new functionality has already been used to deploy various SAP systems. For restrictions on proximity placement groups, see the article referred to at the start of this paragraph. This article will cover the SAP scenarios in which Azure proximity placement groups can or should be used.

## What are proximity placement groups? 
An Azure proximity placement group is a logical construct. When one is defined, it's bound to an Azure region and an Azure resource group. When VMs are deployed, a proximity placement group is referenced by:

- The first Azure VM deployed in the datacenter. The first virtual machine can be seen as an anchor VM that's deployed in a datacenter based on Azure allocation algorithms that are eventually combined with user definitions for a specific Availability Zone.
- All subsequent VMs deployed that reference the proximity placement group, to place all subsequently deployed Azure VMs in the same datacenter as the first virtual machine.

> [!NOTE]
> If there is no host hardware deployed that could run a specific VM type in the datacenter that the first VM was placed in, the deployment of the requested VM type won’t succeed. You’ll get a failure message. This situation can occur with VMs that aren’t mainstream, like VMs with GPUs or HPC VM types, which should  be used with, for example, an M-Series VM that has been deployed as the first VM type.

A single [Azure resource group](https://docs.microsoft.com/azure/azure-resource-manager/manage-resources-portal) can have multiple proximity placement groups assigned to it. But a proximity placement group can be assigned to only one Azure resource group.

When you use proximity placement groups, keep these considerations in mind:

- When you aim for optimal performance for your SAP system and limit yourself to a single Azure datacenter for the system by using proximity placement groups, you might not be able to combine all types of VM families within the placement group. These limitations occur because the host hardware that’s needed to run a certain VM type might not be present in the datacenter to which the “anchor VM” of the placement group was deployed.
- During the life cycle of such an SAP system, you could be forced to move the system to another datacenter. This move could be required if you decide your scale-out HANA DBMS layer should, for example, move from four nodes to 16 nodes, and there's not enough capacity to get an additional 12 VMs of the type you used in the datacenter.
- Because of decommissioning of hardware, Microsoft might build up capacities for a VM type you used in a different datacenter, rather than the one that you initially used. In that scenario, you might need to move the all the proximity placement group's VMs into another datacenter.

## Proximity placement groups with SAP systems that use only Azure VMs
Most SAP NetWeaver and S/4HANA system deployments on Azure don't use [HANA Large Instances](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture). For deployments of these systems, it's important to provide optimal performance between the SAP application layer and the DBMS tier. To do so, define an Azure proximity placement group just for the system.

In most customer deployments, customers build a single [Azure resource group](https://docs.microsoft.com/azure/azure-resource-manager/manage-resources-portal) for SAP systems. In that case, there's a one-to-one relationship between, for example, the production ERP system resource group and its proximity placement group. In other cases, customers organize their resource groups horizontally and collect all production systems in a single resource group. In this case, you'd have a one-to-many relationship between your resource group for production SAP systems and several proximity placement groups of your production SAP ERP, SAP BW, and so on.

Avoid bundling several SAP production or non-production systems in a single proximity placement group. When a small number of SAP systems or an SAP system and some surrounding applications need to have low latency network communication, you might consider moving these systems into one proximity placement group. You should avoid bundles of systems because the more systems you group in a proximity placement group, the higher the chances:

- That you require a VM type that can't be run in the specific datacenter into which the proximity placement group was anchored.
- That resources of non-mainstream VMs, like M-Series VMs, could eventually be unfulfilled when you need more because you're adding software to a proximity placement group over time.

Here's what the ideal configuration, as described, looks like:

![Proximity placement groups with only Azure VMs](./media/sap-proximity-placement-scenarios/ppg-for-all-azure-vms.png)

In this case, single SAP systems are grouped in one resource group each, with one proximity placement group each. There's no dependency on whether you use HANA scale-out or DBMS scale-up configurations.

## Proximity placement groups and HANA Large Instances
If some of your SAP systems rely on [HANA Large Instances](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture) for the application layer, you can experience significant improvements in network latency between the HANA Large Instances unit and Azure VMs when you're using HANA Large Instances units that have been deployed in [Revision 4 rows or stamps](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-network-architecture#networking-architecture-for-hana-large-instance). One improvement is that HANA Large Instances units, as they're deployed, deploy with a proximity placement group. You can use that proximity placement group to deploy your application layer VMs. As a result, those VMs will be deployed in the same datacenter that hosts your HANA Large Instances unit.

In order to detect whether your HANA Large Instance unit is deployed in a Revision 4 stamp or row, check the article [Azure HANA Large Instances control through Azure portal](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-li-portal#look-at-attributes-of-single-hli-unit). In the attributes overview of your HANA Large Instance unit, you also can find out the name of the proximity placement group as it has been created at deployment time for your HANA Large Instance unit. The name displayed in the Attributes overview,  is the name of the proximity placement group, you should use to deploy your application layer VMs into.

In opposite to SAP systems that use Azure virtual machines only, the decision how many [Azure resource groups](https://docs.microsoft.com/azure/azure-resource-manager/manage-resources-portal) should be used is taken away from you to a degree when using HANA Large Instances. All the HANA Large Instance units of a [HANA Large Instance tenant](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-know-terms) are grouped in a single Azure resource group as described [here](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-li-portal#display-of-hana-large-instance-units-in-the-azure-portal). Unless you desired a deployment into different tenants to separate, for example, production and non-production or certain systems, all your HANA Large Instance units will be deployed in one HANA Large Instance tenant, which again has a 1:1 relationship with an Azure resource Group. Whereas all the single units will have a separate proximity placement group defined.

As a result, the grouping between Azure resource groups and proximity placement groups for a single tenant will look like:

![Proximity placement groups for all Azure VMs](./media/sap-proximity-placement-scenarios/ppg-for-hana-large-instance-units.png)


## Short example of deploying with Azure proximity placement groups
To demonstrate, how you can use Azure proximity placement groups to deploy your VM, here a list of PowerShell commands that demonstrate its usage for a first little exercise with Azure proximity placement groups

First step after being logged in with your [Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/) is to check whether you are in the correct Azure subscription you want to use to deploy with the command:

<pre><code>
Get-AzureRmContext
</code></pre>

If you need to change to a different subscription, you can do so by executing this command:

<pre><code>
Set-AzureRmContext -Subscription "my PPG test subscription"
</code></pre>

As a third step, you want to create a new Azure resource group with this command:

<pre><code>
New-AzResourceGroup -Name "myfirstppgexercise" -Location "westus2"
</code></pre>

You can create the new proximity placement group now with:

<pre><code>
New-AzProximityPlacementGroup -ResourceGroupName "myfirstppgexercise" -Name "letsgetclose" -Location "westus2"
</code></pre>

You now can start deploying your first VM into this proximity placement group with a command like:

<pre><code>
New-AzVm -ResourceGroupName "myfirstppgexercise" -Name "myppganchorvm" -Location "westus2" -OpenPorts 80,3389 -ProximityPlacementGroup "letsgetclose" -Size "Standard_DS11_v2"
</code></pre>

With the command above, a Windows based VM is deployed. After this VM deployment succeeded, the datacenter scope of the proximity placement group is defined within the Azure region. All subsequent VM deployments that reference the proximity placement group as in the last command, will be deployed in the same Azure datacenter as long as the VM type can be hosted on hardware placed in that datacenter and/or capacity for that VM type is available.

## Combine Availability Sets and Availability Zones with proximity placement groups 
Using Availability Zones for SAP System deployments, one of the disadvantages is the fact that the SAP application layer can't be controlled deployed using availability sets within the specific zone. Since you want to have the SAP application layer deployed in the same zones as the DBMS layer, and referencing an Availability Zone and an availability set when deploying a single VM is not supported, you were forced to deploy your application layer by referencing a zone and thereby lose the capability of making sure that the application layer VMs got spread across different update and failure domains. 
With the help of proximity placement groups, you can overcome this restriction. The sequence of deployments would look like:

- Create a proximity placement group
- Deploy your 'anchor VM', usually the DBMS server by referencing a certain Azure Availability Zone
- Create an availability set that references the Azure proximity group (see below)
- Deploy the application layer VMs by referencing the availability set and the proximity placement group

Instead of deploying the first VM as demonstrated above, you reference an Azure Availability Zone and the proximity placement group when deploying the VM like:

<pre><code>
New-AzVm -ResourceGroupName "myfirstppgexercise" -Name "myppganchorvm" -Location "westus2" -OpenPorts 80,3389 -Zone "1" -ProximityPlacementGroup "letsgetclose" -Size "Standard_E16_v3"
</code></pre>

A successful deployment of this virtual machine that would host the database instance of my SAP system in one Azure Availability Zone, the scope of the proximity placement group is fixed to one of the datacenters that are representing the Availability Zone you defined.

We assume that you deploy the central services VMs in the same way as the DBMs VMs by referencing the same zone(s) as for the DBMS VMs and the same proximity placement groups. In the next step, you need to create the availability set(s) that you want to use for the application layer of your SAP system.
The proximity placement group needs to be defined and created. The command for creating the availability set requires an additional reference to the proximity placement group ID (not the name). You can get the ID of the proximity placement group with:



<pre><code>
Get-AzProximityPlacementGroup -ResourceGroupName "myfirstppgexercise" -Name "letsgetclose"
</code></pre>

When you create the availability set, you need to consider additional parameters when you are using managed disks (default unless specified differently) and proximity placement groups:

<pre><code>
New-AzAvailabilitySet -ResourceGroupName "myfirstppgexercise" -Name "myppgavset" -Location "westus2" -ProximityPlacementGroupId "/subscriptions/my very long ppg id string" -sku "aligned" -PlatformUpdateDomainCount 3 -PlatformFaultDomainCount 2 
</code></pre>

Ideally, you should use three fault domains. However, the number of supported fault domains can vary from region to region. In this case, the maximum fault domain count possible for the specific regions was two. For deploying your application layer VMs, you need to add a reference to your availability set name and the proximity placement group name, as demonstrated here:

<pre><code>
New-AzVm -ResourceGroupName "myfirstppgexercise" -Name "myppgavsetappvm" -Location "westus2" -OpenPorts 80,3389 -AvailabilitySetName "myppgavset" -ProximityPlacementGroup "letsgetclose" -Size "Standard_DS11_v2"
</code></pre>

The end result of this sequence will be a DBMS layer and central services of your SAP system that is located in a specific Availability Zone(s) and an SAP application layer that is located through availability sets in the same Azure datacenters as the DBMS VM(s) got deployed.

> [!NOTE]
> As you deploy one DBMS VM into one zone and the second DBMS VM into another zone to create a high availability configuration, you are going to require different proximity placement groups for each of the zones. Same si true for the availability set you might use

## Get an existing system into Azure proximity placement groups
As you have SAP systems deployed already, you might want to optimize the network latency of some of your critical systems and locate the application layer and the DBMS layer in the same datacenter. In the stage of public preview of proximity placement group functionality, a deletion of the VMs and a new creation of the VMs is necessary to perform such a move into proximity placement groups. At this stage of the functionality, it is not good enough to shut down the VMs to be able to assign those shutdown virtual machines to proximity placement groups.


## Next Steps
Consult the documentation:

- [SAP workload on Azure planning and deployment checklist](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-deployment-checklist)
- [Preview: Deploy VMs to proximity placement groups using Azure CLI](https://docs.microsoft.com/azure/virtual-machines/linux/proximity-placement-groups)
- [Preview: Deploy VMs to proximity placement groups using PowerShell](https://docs.microsoft.com/azure/virtual-machines/windows/proximity-placement-groups)
- [Considerations for Azure Virtual Machines DBMS deployment for SAP workload](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/dbms_guide_general)

