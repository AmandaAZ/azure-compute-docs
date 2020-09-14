---
title: Overview of Maintenance control for Azure virtual machine scale sets
description: Learn how to control when maintenance is applied to your Azure virtual machine scale sets using Maintenance Control.
author: cynthn
ms.service: virtual-machine-scale-sets
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 09/11/2020
ms.author: cynthn
#pmcontact: shants
---

# Maintenance control for Azure virtual machine scale sets 

Manage [automatic OS image upgrades](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md) for your virtual machine scale sets using maintenance control.

Maintenance control lets you decide when to apply updates to OS disks in your virtual machine scale sets through an easier and more predictable experience. 

Maintenance configurations work across subscriptions and resource groups.

The entire workflow comes down to these steps: 
- Create a maintenance configuration.
- Associate a virtual machine scale set to a maintenance configuration.
- Enable automatic OS upgrades.


## Limitations

- VMs must be in a scale set.
- User must have **Resource Contributor** access.


## Management options

You can create and manage maintenance configurations using any of the following options:

- [Azure PowerShell](virtual-machine-scale-sets-maintenance-control-powershell.md)


## Next steps

Learn how to control when maintenance is applied to your Azure virtual machine scale sets using Maintenance control and PowerShell.

> [!div class="nextstepaction"]
> [VMSS Maintenance control using PowerShell](virtual-machine-scale-sets-maintenance-control-powershell.md)
