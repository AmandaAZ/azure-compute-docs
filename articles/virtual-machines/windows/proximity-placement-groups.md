---
title: Proximity placement groups preview for Windows VMs | Microsoft Docs
description: Learn about creating and using proximity placement groups for Windows virtual machines in Azure. 
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc

ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 05/17/2019
ms.author: cynthn

---

# Creating and using proximity placement groups using PowerShell


[!INCLUDE [virtual-machines-common-ppg-overview](../../../includes/virtual-machines-common-ppg-overview.md)]

## Create a proximity placement group
Create a proximity placement group using the [New-AzProximityPlacementGroup](https://docs.microsoft.com/powershell/module/az.compute/new-azproximityplacementgroup) cmdlet. 

```azurepowershell-interactive
$resourceGroup = "myPPGResourceGroup"
$location = "East US"
$ppgName = "myPPG"
New-AzResourceGroup -Name $resourceGroup -Location $location
$ppg = New-AzProximityPlacementGroup `
   -Location $location `
   -Name $ppgName `
   -ResourceGroupName $resourceGroup `
   -ProximityPlacementGroupType Standard
```

## List proximity placement groups

You can list all of the proimity placement groups using the [Get-AzProximityPlacementGroup](/powershell/module/az.compute/get-azproximityplacementgroup) cmdlet.

```azurepowershell-interactive
Get-AzProximityPlacementGroup
```


## Create a VM

Create a VM in the proximity placement group using `-ProximityPlacementGroup $ppg.Id` to refer to the proximity placement group ID when you use [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) to create the VM.

```azurepowershell-interactive
$vmName = "myVM"

New-AzVm `
  -ResourceGroupName $resourceGroup `
  -Name $vmName `
  -Location $location `
  -OpenPorts 3389 `
  -ProximityPlacementGroup $ppg.Id
```



## Next steps

You can also use the [Azure CLI](../linux/proximity-placement-groups.md) to create proximity placement groups.