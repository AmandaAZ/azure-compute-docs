---
title: Standby Pools for Virtual Machine Scale Sets
description: Learn how to utilize Standby Pools to reduce scale-out latency with Virtual Machine Scale Sets
author: mimckitt
ms.author: mimckitt
ms.service: virtual-machine-scale-sets
ms.topic: how-to
ms.date: 01/16/2024
ms.reviewer: ju-shim
---

# Standby Pools for Virtual Machine Scale Sets

> [!IMPORTANT]
> Standby Pools for Virtual Machine Scale Sets are currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA). 

Standby Pools for Virtual Machine Scale Sets allow you to increase scaling performance by creating a pool of pre-provisioned virtual machines from which the scale set can draw from when scaling out. 

Standby Pools reduce the time to scale-out by performing various initialization steps such as installing applications/ software or loading large amounts of data. These initialization steps are performed on the VMs in the Standby Pool prior to being put into the scale set and before the instances begin taking traffic.

## Standby Pool Size
The number of VMs in a Standby Pool is determined by the number of VMs in your scale set and the total available capacity you want ready at any point in time. 

| Setting | Description | 
|---|---|
| `MaxReadyCapacity` | The maximum number of VMs you want to have ready.|
| `instanceCount` | The current number of VMs already deployed in your scale set.|
|Standby Pool Size | `MaxReadyCapacity`– `InstanceCount`

## Scaling

When your scale set requires more instances, rather than creating new instances and placing them directly into the scale set, the scale set can instead pull VMs from the Standby Pool. This significantly reduces the time it takes to scale-out and have the instances ready to take traffic. 

When your scale set scales back down, the instances are deleted from your scale set and your Standby Pool will refill to meet the `MaxReadyCapacity` set.  

If at any point in time your scale set needs to scale beyond the number of instances you have in your Standby Pool, the scale set defaults to standard scale-out methods and create new instances that are added directly into the Scale Set

## Virtual Machine States

The VMs in the Standby Pool can be created in a Running State or a Stopped (deallocated) state. The states of the VMs in the Standby Pool are configured using the `virtualMachineState` parameter.

```
"virtualMachineState":"Running"

"virtualMachineState":"Deallocated"
```

**Stopped (Deallocated) VM State:** Deallocated VMs are shut down and keep any associated data disks, 
NICs, and any static IPs remain unchanged. 

**Running VM State:** Using VMs in a Running state is recommended when latency and reliability 
requirements are strict.

## Pricing

>[!IMPORTANT]
>The `VirtualMachineState` you choose will impact the cost of your Standby Pool. You can update the desired state at any point in time. 

There's no direct cost associated with using Standby Pools. Users are charged based on the resources deployed into the Standby Pool. For more information on Virtual Machine billing, see [VM power states and billing documentation](../virtual-machines/states-billing.md)

| State | Description |
|---|---|
|**Stopped (deallocated) VM State:** | Using a Standby Pool with VMs in the Stopped (deallocated) state is a great way to reduce the cost while keeping your scale-out times fast. VMs in the Stopped (deallocated) state don't incur any compute costs, only the associated resources incur costs. |
| **Running VM State:** | Running VMs incur a higher cost due to compute resources being consumed. |

## Considerations
- The total capacity of the Standby Pool and the Virtual Machine Scale Set together can't exceed 1000 instances. 
- Standby Pools don't currently support Spot Virtual Machines or Spot Priority Mix.
- If the instances in your Standby Pool are exhausted, the scale set defaults to standard scale-out methods and create new VMs directly in your scale set to meet the new desired instance count.  
- Creation of pooled resources is subject to the resource availability in each region.
- If using autoscale to trigger scaling, autoscale takes into account the metrics associated with your VMs in your scale set and the VMs in the pool. This could result in unexpected scale-out events.

## Next steps

Learn how to [create a Standby Pool](standby-pools-create.md)