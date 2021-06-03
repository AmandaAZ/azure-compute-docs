---
title: Flexible orchestration for virtual machine scale sets in Azure
description: Learn how to use Flexible orchestration mode for virtual machine scale sets in Azure.
author: fitzgeraldsteele
ms.author: fisteele
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: flexible-scale-sets
ms.date: 05/24/2021
ms.reviewer: jushiman
ms.custom: mimckitt, devx-track-azurecli, vmss-flex
---

# Preview: Flexible orchestration for virtual machine scale sets in Azure

**Applies to:** :heavy_check_mark: Flexible scale sets

Virtual machine scale sets with Flexible orchestration allows you to combine the scalability of [virtual machine scale sets](../virtual-machine-scale-sets/overview.md) with the regional availability guarantees of [availability sets](availability-set-overview.md).

With Flexible orchestration, Azure provides a unified experience across the Azure VM ecosystem. Flexible orchestration offers high availability guarantees (up to 1000 VMs) by spreading VMs across fault domains in a region or within an Availability Zone. This enables you to scale out your application while maintaining fault domain isolation that is essential to run quorum-based or stateful workloads, including:
- Quorum-based workloads
- Open-Source databases
- Stateful applications
- Services that require High Availability and large scale
- Services that want to mix virtual machine types, or leverage Spot and on-demand VMs together
- Existing Availability Set applications

Learn more about the differences between Uniform scale sets and Flexible scale sets in [Orchestration Modes](../virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes.md).


> [!IMPORTANT]
> Virtual machine scale sets in Flexible orchestration mode is currently in public preview. An opt-in procedure is needed to use the public preview functionality described below.
> This preview version is provided without a service level agreement and is not recommended for production workloads. Certain features might not be supported or might have constrained capabilities.
> For more information, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


> [!CAUTION]
> The orchestration mode is defined when you create the scale set and cannot be changed or updated later.


## What has changed with Flexible orchestration mode?
One of the main advantages of Flexible orchestration is that it provides orchestration features over standard Azure IaaS VMs, instead of scale set child virtual machines. This means you can use all of the standard VM APIs when managing Flexible orchestration instances, instead of the virtual machine scale set VM APIs you use with [Uniform orchestration](.\virtual-machine-scale-sets\virtual-machine-scale-sets-orchestration-modes.md). During the preview period, there are several differences between managing instances in Flexible orchestration versus Uniform orchestration. In general, we recommend that you use the standard Azure IaaS VM APIs when possible. In this section, we highlight examples of best practices for managing VM instances with Flexible orchestration.

### Assign fault domain during VM creation
You can choose the number of fault domains for the Flexible orchestration scale set. By default, when you add a VM to a Flexible scale set, Azure evenly spreads instances across fault domains. While it is recommended to let Azure assign the fault domain, for advanced or troubleshooting scenarios you can override this default behavior and specify the fault domain where the instance will land.

```azurecli-interactive
az vm create –vmss "myVMSS"  –-platform_fault_domain 1
```

### Instance naming
When you create a VM and add it to a Flexible scale set, you have full control over instance names within the Azure Naming convention rules. When VMs are automatically added to the scale set via autoscaling, you provide a prefix and Azure appends a unique number to the end of the name. 

### List scale sets VM API changes
Virtual Machine Scale Sets allows you to list the instances that belong to the scale set. With Flexible orchestration, the list Virtual Machine Scale Sets VM command provides a list of scale sets VM IDs. You can then call the GET Virtual Machine Scale Sets VM commands to get more details on how the scale set is working with the VM instance. To get the full details of the VM, use the standard GET VM commands or [Azure Resource Graph](../governance/resource-graph/overview.md).

### Query instances for power state
The preferred method is to use Azure Resource Graph to query for all VMs in a Virtual Machine Scale Set. Azure Resource Graph provides efficient query capabilities for Azure resources at scale across subscriptions.

```
| where type =~ 'Microsoft.Compute/virtualMachines'
| where properties.virtualMachineScaleSet contains "demo"
| extend powerState = properties.extended.instanceView.powerState.code
| project name, resourceGroup, location, powerState
| order by resourceGroup desc, name desc
```

Querying resources with [Azure Resource Graph](../governance/resource-graph/overview.md) is a convenient and efficient way to query Azure resources and minimizes API calls to the resource provider. Azure Resource Graph is an eventually consistent cache where new or updated resources may not be reflected for up to 60 seconds. You can:
- List VMs in a resource group or subscription.
- Use the expand option to retrieve the instance view (fault domain assignment, power and provisioning states) for all VMs in your subscription.
- Use the Get VM API and commands to get model and instance view for a single instance.

### Scale sets VM Batch operations
Use the standard VM commands to start, stop, restart, delete instances, instead of the Virtual Machine Scale Set VM APIs. The Virtual Machine Scale Set VM Batch operations (start all, stop all, reimage all, etc.) are not used with Flexible orchestration mode.

### Monitor application health
Application health monitoring allows your application to provide Azure with a heartbeat to determine whether your application is healthy or unhealthy. Azure can automatically replace VM instances that are unhealthy. For Flexible scale set instances, you must install and configure the Application Health Extension on the virtual machine. For Uniform scale set instances, you can use either the Application Health Extension, or measure health with an Azure Load Balancer Custom Health Probe.

### Retrieve boot diagnostics data
Use the standard VM APIs and commands to retrieve instance Boot Diagnostics data and screenshots. The Virtual Machine Scale Sets VM boot diagnostic APIs and commands are not used with Flexible orchestration mode instances.

### VM extensions
Use extensions targeted for standard virtual machines, instead of extensions targeted for Uniform orchestration mode instances.


## Features
The following table lists the Flexible orchestration mode features and links to the appropriate documentation.

| Feature | Supported by Flexible orchestration (Preview) |
|-|-|
| Virtual machine type | Standard Azure IaaS VM (Microsoft.compute /virtualmachines) |
| SKUs supported | D series, E series, F series, A series,   B series, Intel, AMD |
| Availability Zones | Optionally specify all instances land in   a single availability zone |
| Full control over VM, NICs, Disks | Yes |
| Automatic Scaling | No |
| Assign VM to a   Specific Fault Domain | Yes |
| Remove NICs and Disks when deleting   VM instances | No |
| Upgrade Policy (VM scale sets) | No |
| Automatic OS Updates (VM scale sets) | No |
| In Guest Security Patching | Yes |
| Terminate Notifications (VM scale sets) | No |
| Instance Repair (VM scale sets) | No |
| Accelerated networking | Yes |
| Spot instances and pricing  | Yes, you can have both Spot and Regular priority instances |
| Mix operating systems | Yes, Linux and Windows can reside in the same Flexible scale set |
| Monitor Application Health | Application health extension |
| UltraSSD Disks  | Yes |
| Infiniband  | No |
| Write Accelerator  | No |
| Proximity Placement Groups  | Yes |
| Azure Dedicated Hosts  | No |
| Basic SLB  | No |
| Azure Load Balancer Standard SKU | Yes |
| Application Gateway | No |
| Maintenance Control  | No |
| List VMs in Set | Yes |
| Azure Alerts | No |
| VM Insights | No |
| Azure Backup | Yes |
| Azure Site Recovery |  No |
| Add/remove existing VM to the group | No |


## Register for Flexible orchestration mode
Before you can deploy virtual machine scale sets in Flexible orchestration mode, you must first register your subscription for the preview feature. The registration may take several minutes to complete. You can use Azure Portal, Azure PowerShell, or Azure CLI to register.

### Azure Portal
Navigate to the details page for the subscription you would like to create a scale set in Flexible orchestration mode, and select Preview Features from the menu. Select the two orchestrator features to enable: _VMOrchestratorSingleFD_ and _VMOrchestratorMultiFD_, and press the Register button. Feature registration can take up to 15 minutes.

![Feature registration.](https://user-images.githubusercontent.com/157768/110361543-04d95880-7ff5-11eb-91a7-2e98f4112ae0.png)

Once the features have been registered for your subscription, complete the opt-in process by propagating the change into the Compute resource provider. Navigate to the Resource providers tab for your subscription, select Microsoft.compute, and click Re-register.

![Re-register](https://user-images.githubusercontent.com/157768/110362176-cd1ee080-7ff5-11eb-8cc8-36aa967e267a.png)


### Azure PowerShell
Use the [Register-AzProviderFeature](/powershell/module/az.resources/register-azproviderfeature) cmdlet to enable the preview for your subscription.

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName VMOrchestratorMultiFD -ProviderNamespace Microsoft.Compute `
Register-AzProviderFeature -FeatureName VMOrchestratorSingleFD -ProviderNamespace Microsoft.Compute
```

Feature registration can take up to 15 minutes. To check the registration status:

```azurepowershell-interactive
Get-AzProviderFeature -FeatureName VMOrchestratorMultiFD -ProviderNamespace Microsoft.Compute
```

Once the feature has been registered for your subscription, complete the opt-in process by propagating the change into the Compute resource provider.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
```

### Azure CLI 2.0
Use [az feature register](/cli/azure/feature#az_feature_register) to enable the preview for your subscription.

```azurecli-interactive
az feature register --namespace Microsoft.Compute --name VMOrchestratorMultiFD
az feature register --namespace microsoft.compute --name VMOrchestratorSingleFD
```

Feature registration can take up to 15 minutes. To check the registration status:

```azurecli-interactive
az feature show --namespace Microsoft.Compute --name VMOrchestratorMultiFD
```

Once the feature has been registered for your subscription, complete the opt-in process by propagating the change into the Compute resource provider.

```azurecli-interactive
az provider register --namespace Microsoft.Compute
```


## Get started with Flexible orchestration mode

Get started with Flexible orchestration mode for your scale sets through the Azure portal, Azure CLI, Azure PowerShell, Terraform, or REST API.
- [Azure Portal](flexible-virtual-machine-scale-sets-portal.md)
- [Azure CLI](flexible-virtual-machine-scale-sets-cli.md)
- [Azure PowerShell](flexible-virtual-machine-scale-sets-powershell.md)
- [Terraform](](flexible-virtual-machine-scale-sets-terraform.md))
- [REST API](flexible-virtual-machine-scale-sets-rest-api.md) 


## Autoscaling 

Create a Flexible scale set with a VM Profile and autoscaling. Flexible orchestration uses the same API as Uniform orchestration that is Generally Available. The following parameters are required: 
- Api-version 2021-03-01 (or greater) 
- Single placement group: false 
- orchestrationMode: Flexible 
- virtualMachineProfile.networkProfile.networkApiVersion: 2020-11-01 or later 


## Troubleshoot scale sets with Flexible orchestration
Find the right solution to your troubleshooting scenario.

```
InvalidParameter. The value 'False' of parameter 'singlePlacementGroup' is not allowed. Allowed values are: True
```

**Cause:** The subscription is not registered for the Flexible orchestration mode Public Preview.

**Solution:** Follow the instructions above to register for the Flexible orchestration mode Public Preview.

```
InvalidParameter. The specified fault domain count 2 must fall in the range 1 to 1.
```

**Cause:** The `platformFaultDomainCount` parameter is invalid for the region or zone selected.

**Solution:** You must select a valid `platformFaultDomainCount` value. For zonal deployments, the maximum `platformFaultDomainCount` value is 1. For regional deployments where no zone is specified, the maximum `platformFaultDomainCount` varies depending on the region. See [Manage the availability of VMs for scripts](../virtual-machines/availability.md) to determine the maximum fault domain count per region.

```
OperationNotAllowed. Deletion of Virtual Machine Scale Set is not allowed as it contains one or more VMs. Please delete or detach the VM(s) before deleting the Virtual Machine Scale Set.
```

**Cause:** Trying to delete a scale set in Flexible orchestration mode that is associated with one or more virtual machines.

**Solution:** Delete all of the virtual machines associated with the scale set in Flexible orchestration mode, then you can delete the scale set.

```
InvalidParameter. The value 'True' of parameter 'singlePlacementGroup' is not allowed. Allowed values are: False.
```
**Cause:** The subscription is registered for the Flexible orchestration mode preview; however, the `singlePlacementGroup` parameter is set to *True*.

**Solution:** The `singlePlacementGroup` must be set to *False*.


## Next steps
> [!div class="nextstepaction"]
> [Flexible orchestration mode for your scale sets with Azure Portal.](flexible-virtual-machine-scale-sets-portal.md)