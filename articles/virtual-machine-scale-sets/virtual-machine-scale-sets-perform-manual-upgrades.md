---
title: Performing Manual Upgrades on Virtual Machine Scale Sets
description: Learn about to perform a Manual Upgrade on Virtual Machine Scale Sets
author: mimckitt
ms.author: mimckitt
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.date: 01/19/2024
ms.reviewer: ju-shim
ms.custom: upgradepolicy
---
# Performing Manual Upgrades on Virtual Machine Scale Sets

> [!IMPORTANT]
> **Manual Upgrade Policy for Virtual Machine Scale Sets with Flexible Orchestration are currently in preview**. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of these features may change prior to general availability (GA). 
>
>Upgrade Policies for Virtual Machine Scale Sets with Uniform Orchestration mode are generally available (GA). 
 
If you have the Upgrade Policy set to manual, you need to trigger manual upgrades of each existing virtual machine to apply changes to the instances based on the updated scale set model. 

### [Portal](#tab/portal1)

Select the Virtual Machine Scale Set you want to perform instance upgrades for. In the menu under **Settings**, select **Instances** and select the instances you want to upgrade. Once selected, click the **Upgrade** option.

:::image type="content" source="../virtual-machine-scale-sets/media/maxsurge/manual-upgrade-1.png" alt-text="Screenshot showing how to perform manual upgrades using the Azure portal.":::


### [CLI](#tab/cli1)
Update Virtual Machine Scale Set instances using [az vmss update-instances](/cli/azure/vmss#az-vmss-update-instances). The `--instance-ids` parameter refers to the ID of the instance if using Uniform Orchestration and the Instance name if using Flexible Orchestration.  

```azurecli-interactive
az vmss update-instances --resource-group myResourceGroup --name myScaleSet --instance-ids {instanceIds}
```
### [PowerShell](#tab/powershell1)
Update Virtual Machine Scale Set instances using [Update-AzVmssInstance](/powershell/module/az.compute/update-azvmssinstance). The `-InstanceId` parameter refers to the ID of the instance if using Uniform Orchestration and the Instance name if using Flexible Orchestration. 
    
```azurepowershell-interactive
Update-AzVmssInstance -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId instanceId
```

### [REST API](#tab/rest1)
Update Virtual Machine Scale Set instances using [update instances](/rest/api/compute/virtualmachinescalesets/updateinstances). The `instanceIds` parameter refers to the ID of the instance if using Uniform Orchestration and the Instance name if using Flexible Orchestration. 

```rest
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/manualupgrade?api-version={apiVersion}

{
  "instanceIds": [
    "myScaleSet1",
    "myScaleSet2"
  ]
}

```
---


## Next steps
You can also perform common management tasks on Virtual Machine Scale Sets using the [Azure CLI](virtual-machine-scale-sets-manage-cli.md) or [Azure PowerShell](virtual-machine-scale-sets-manage-powershell.md).
