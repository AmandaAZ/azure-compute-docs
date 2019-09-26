---
title: Use PowerShell to deploy low-priority VMs in Azure | Microsoft Docs
description: Learn how to use Azure PowerShell to deploy low-priority VMs to save on costs.
services: virtual-machines-windows
author: cynthn
manager: gwallace

ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: article
ms.date: 09/23/2019
ms.author: cynthn
---

# Preview: Deploy low-priority VMs using Azure PowerShell


Using [low-priority VMs](low-priority-vms.md) allows you to take advantage of our unused capacity at a significant cost savings. At any point in time when Azure needs the capacity back, the Azure infrastructure will evict low-priority VMs. Therefore, low-priority VMs are great for workloads that can handle interruptions like batch processing jobs, dev/test environments, large compute workloads, and more.

Pricing for low-priority VMs is variable, based on region and SKU. For more information, see VM pricing for [Linux](https://azure.microsoft.com/en-us/pricing/details/virtual-machines/linux/) and [Windows](https://azure.microsoft.com/en-us/pricing/details/virtual-machines/windows/). For more information about setting the max price, see [Low-priority VMs - Pricing](low-priority-vms.md#pricing).

You have option to set a max price you are willing to pay, per hour, for the VM. The max price for a low-priority VM can be set in USD, using up to 5 decimal places. For example, the value `0.98765`would be a max price of $0.98765 USD per hour. If you set the max price to be `-1`, the VM won't be evicted based on price. The price for the VM will be the current price for low-priority or the price for an on-demand VM, which ever is less, as long as there is capacity and quota available.

> [!IMPORTANT]
> Low-priority VMs are currently in public preview.
> This preview version is provided without a service level agreement, and it's not recommended for production workloads. Certain features might not be supported or might have constrained capabilities. 
> For more information, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>
> For the early part of the public preview, you can set a max price, but it will be ignored. Low-priority VMs will have a fixed price, so there will not be any price-based evictions.

## Register the feature

For the public preview, you first need to register the feature.

```powershell
Register-AzProviderFeature -FeatureName LowPrioritySingleVM -ProviderNamespace Microsoft.Compute
```

It takes a few minutes for the registration to complete. Use [Get-AzProviderFeature](/powershell/module/az.resources/get-azproviderfeature) to check on the status of the feature registration.

```powershell
Get-AzProviderFeature -FeatureName LowPrioritySingleVM -ProviderNamespace Microsoft.Compute
```

When `RegistrationState` returns `Registered`, you can move on to the next step.

## Create the VM

Create a low-priorityVM using [New-AzVmConfig](/powershell/module/az.compute/new-azvmconfig) to create the configuration. Include `-Priority low` and set `-MaxPrice` to either:
- `-1` so the VM is not evicted based on price.
- a dollar amount, up to 5 digits. For example `-MaxPrice .98765` means that the VM will be deallocated once the price for a low-priorityVM goes about $.98765 per hour.

> [!IMPORTANT]
> For the early part of the public preview, you can set a max price, but it will be ignored. Low-priority VMs will have a fixed price, so there will not be any price-based evictions.


This example creates a low-priorityVM that will not be deallocated based on pricing (only when Azure needs the capacity back).

```azurepowershell-interactive
$resourceGroup = "myLowPriRG"
$location = "eastus"
$vmName = "myLowPriVM"
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."
New-AzResourceGroup -Name $resourceGroup -Location $location
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24
$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroup -Location $location `
  -Name MYvNET -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig
$pip = New-AzPublicIpAddress -ResourceGroupName $resourceGroup -Location $location `
  -Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleRDP  -Protocol Tcp `
  -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 3389 -Access Allow
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $resourceGroup -Location $location `
  -Name myNetworkSecurityGroup -SecurityRules $nsgRuleRDP
$nic = New-AzNetworkInterface -Name myNic -ResourceGroupName $resourceGroup -Location $location `
  -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

# Create a virtual machine configuration and set this to be a low-priority VM
$vmConfig = New-AzVMConfig -VMName $vmName -VMSize Standard_D1 -Priority "Low" -MaxPrice -1| `
Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred | `
Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2016-Datacenter -Version latest | `
Add-AzVMNetworkInterface -Id $nic.Id

New-AzVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig
```

After the VM is created, you can query to see the max price for all of the VMs in the resource group.

```powershell
Get-AzVM -ResourceGroupName $resourceGroup | `
   Select-Object Name,@{Name="maxPrice"; Expression={$_.BillingProfile.MaxPrice}}
```

## Next steps

You can also create a low-priority VM using the [Azure CLI](../linux/low-priority-cli.md) or a [template](../linux/low-priority-template.md).

If you encounter an error, see [Error codes](../error-codes-low-priority.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).