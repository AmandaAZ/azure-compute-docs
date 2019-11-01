---
title: Troubleshoot virtual machine deployment due to detached disks | Microsoft Docs
description: Troubleshoot virtual machine deployment due to detached disks
services: virtual-machines-windows
documentationCenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 10/31/2019
ms.author: vaaga
---

# Troubleshoot virtual machine deployment due to detached disks

## Error “Cannot attach data disk < GUID > to virtual machine because the disk is currently being detached”

### In the current scenario:

The user creates a virtual machine with a data disk named "Disk1". The user wants to detach "Disk1" and calls **PutVM** with "Disk1" not in the payload. This call fails.

If the client calls **GetVM()**, they receive a virtual machine Model having "Disk1" returned by CRP (where the virtual machine's pre-provisioning state fails) since its detach operation failed, and the client must be informed.

If the client wishes to attach another disk (e.g. "Disk2"), they make the call **PutVM()** for "Disk2" using the virtual machine model returned by **GetVM()**, unaware of existence of "Disk1" in the virtual machine model.

1. Cx updates the virtual machine "VM1" by detaching data disk "Disk1".
2. This **PutVMOperation** call fails because of an unknown issue.
3. Cx calls **GetVM** through PowerShell, which gives the payload for "VM1" with the data disk set to "Disk1". 
4. The client performs a virtual machine update, such as adding tags in the above payload.

   > ![NOTE:]
   > Cx need not remove "Disk1" from the payload.

5. This update operation fails with **AttachDisksWhileBeingDetached** for "Disk1".

In cases such as the one above, the **PutVM()** request payload will include Disk1, which Cx tried to detach before and failed. When the newer update payload has this disk, CRP assumes that the client is trying to reattach the failed detached "Disk1". Consequently, CRP throws the error **AttachDiskWhileBeingDetached**”** and fails the valid **PutVM()** call that was trying to attach "Disk2". 

### Current Scenario with the **toBeDetached** data disk flag. (This feature flag is already GA)

To improve the experience, Microsoft introduced the **toBeDetached** flag for data disks for API Version 2019-03-01, as per public documentation. This flag will help avoid/reduce **AttachDiskWhileBeing** detached CRIs from occurring.

In the new scenario:

1. Cx updates the virtual machine "VM1" by detaching data disk "Disk1".
2. This **PutVMOperation** call fails because of an unknown issue.
3. Cx calls **GetVM** through PowerShell, which gives the payload for "VM1" with the data disk set to "Disk1". In this case, the data disk "Disk1" will have the **toBeDetached** property set to "true".
4. The client performs a virtual machine update, adding tags in the above payload.

> ![NOTE:]
> Cx need not remove "Disk1" from the payload.

5. The update operation will now succeed.

## Attach Disks while being Detached Public Documentation Draft

This article describes how to resolve the [AttachDisksWhileBeingDetached](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/error-messages) error when trying to detach a disk from a virtual machine.

### Symptom

When you're trying to update a virtual machine whose previous data disk detach failed, you might come across this error code.

```
Code=\"AttachDiskWhileBeingDetached\" 
Message=\"Cannot attach data disk 'f94901ef-75ee-4477-9ad6-1c74da50e7ef' to virtual machine 'aks-systempool-23089071-vm_4' because the disk is currently being detached or the last detach  operation failed. Please wait until the disk is completely detached, and then try again or delete/detach the disk explicitly again\” 
```

### Cause

This error happens when you try reattaching a data disk whose last detach operation failed. The best way to get out of this state is to detach the failing disk.

## Solution 1: Powershell

### Step 1: Get the virtual machine and disk details

```azurepowershell-interactive
PS D:> $vm = Get-AzureRmVM -ResourceGroupName "varsha_newRG" -Name "varshaVM22October" 

PS D:> $vm.StorageProfile.DataDisks 
lun          : 0
name         : f94901ef-75ee-4477-9ad6-1c74da50e7ef
createOption : Attach
caching      : ReadOnly
managedDisk  : @{storageAccountType=Premium_LRS; id=/subscriptions/11ceafd0-fa99-4f18-a6b7-ced6ad02e323/resourceGroups/ mc_acse-jhub_acsejhub_westeurope/providers/Microsoft.Compute/disks/f94901ef-75ee-4477-9ad6-1c74da50e7ef}
diskSizeGB   : 8
toBeDetached : False 
```

### Step 2: Set the flag for failing disks to "true".

Get the array index of the failing disk and set the **toBeDetached** flag for the failing disk (for which **AttachDiskWhileBeingDetached** error occurred) to "true". This setting implies detaching the disk from the virtual machine. The failing disk name can be found in the **errorMessage**.

> !Note:
> The API version specified for Get and Put calls needs to be 2019-03-01 or greater.

```azurepowershell-interactive
PS D:> $vm.StorageProfile.DataDisks[0].ToBeDetached = $true 
```

Alternately, you can also detach this disk using the command below, which will be helpful for users using API versions before 2019-03-01

```azurepowershell-interactive
PS D:> Remove-AzureRmVMDataDisk -VM $vm -Name "f94901ef-75ee-4477-9ad6-1c74da50e7ef" 
```

### Step 3: Update the virtual machine

```azurepowershell-interactive
PS D:> Update-AzureRmVM -ResourceGroupName "mc_acse-jhub_acsejhub_westeurope" -VM $vm 
```

## Solution 2: REST

### Step 1: Get the virtual machine payload.

```azurepowershell-interactive
GET https://management.azure.com/subscriptions/11ceafd0-fa99-4f18-a6b7-ced6ad02e323/providers/Microsoft.Compute/virtualMachines/aks-systempool-23089071-vm_4?api-version=2019-03-01
```

### Step 2: Set the flag for failing disks to "true".

Set the **toBeDetached** for failing disk to true in the payload returned in Step 1. Please Note: The API version specified for Get and Put calls needs to be 2019-03-01 or greater.

**Sample Request Body**

```azurepowershell-interactive
{
  "properties": {
    "hardwareProfile": {
      "vmSize": "Standard_D2_v2"
    },
    "storageProfile": {
      "imageReference": {
        "sku": "2016-Datacenter",
        "publisher": "MicrosoftWindowsServer",
        "version": "latest",
        "offer": "WindowsServer"
      },
      "osDisk": {
        "caching": "ReadWrite",
        "managedDisk": {
          "storageAccountType": "Standard_LRS"
        },
        "name": "myVMosdisk",
        "createOption": "FromImage"
      },
      "dataDisks": [
        {
          "diskSizeGB": 1023,
          "createOption": "Empty",
	    "name": "f94901ef-75ee-4477-9ad6-1c74da50e7ef",
          "lun": 0,
          "toBeDetached": true
        },
        {
          "diskSizeGB": 1023,
          "createOption": "Empty",
          "lun": 1,
          "toBeDetached": false
        }
      ]
    },
    "osProfile": {
      "adminUsername": "{your-username}",
      "computerName": "myVM",
      "adminPassword": "{your-password}"
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/{existing-nic-name}",
          "properties": {
            "primary": true
          }
        }
      ]
    }
  }
}
```

Alternately you can also remove the failing data disk from the above payload, which is helpful for users using API versions before 2019-03-01

### Step 3: Update the virtual machine

Use the request body payload set in Step 2 and update the virtual machine as follows:

```azurepowershell-interactive
PATCH https://management.azure.com/subscriptions/11ceafd0-fa99-4f18-a6b7-ced6ad02e323/providers/Microsoft.Compute/virtualMachines/aks-systempool-23089071-vm_4?api-version=2019-03-01
```

**Sample Response:**

```azurepowershell-interactive
{
  "id": "/subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "type": "Microsoft.Compute/virtualMachines",
  "properties": {
    "osProfile": {
      "adminUsername": "{your-username}",
      "secrets": [],
      "computerName": "myVM",
      "windowsConfiguration": {
        "provisionVMAgent": true,
        "enableAutomaticUpdates": true
      }
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/nsgExistingNic",
          "properties": {
            "primary": true
          }
        }
      ]
    },
    "storageProfile": {
      "imageReference": {
        "sku": "2016-Datacenter",
        "publisher": "MicrosoftWindowsServer",
        "version": "latest",
        "offer": "WindowsServer"
      },
      "osDisk": {
        "osType": "Windows",
        "caching": "ReadWrite",
        "createOption": "FromImage",
        "name": "myVMosdisk",
        "managedDisk": {
          "storageAccountType": "Standard_LRS"
        }
      },
      "dataDisks": [
        {
          "caching": "None",
          "managedDisk": {
            "storageAccountType": "Standard_LRS"
          },
          "createOption": "Empty",
          "lun": 0,
          "diskSizeGB": 1023,
	    "name": "f94901ef-75ee-4477-9ad6-1c74da50e7ef",
          "toBeDetached": true
        },
        {
          "caching": "None",
          "managedDisk": {
            "storageAccountType": "Standard_LRS"
          },
          "createOption": "Empty",
          "lun": 1,
          "diskSizeGB": 1023,
          "toBeDetached": false
        }
      ]
    },
    "vmId": "3906fef9-a1e5-4b83-a8a8-540858b41df0",
    "hardwareProfile": {
      "vmSize": "Standard_D2_v2"
    },
    "provisioningState": "Updating"
  },
  "name": "myVM",
  "location": "westus"
}
```

## Next Steps

If you are having issues connecting to your VM, see [Troubleshoot RDP connections to an Azure VM](troubleshoot-rdp-connection.md).

For issues with accessing applications running on your VM, see [Troubleshoot application connectivity issues on a Windows VM](troubleshoot-app-connection.md).