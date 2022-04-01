---
title: Use Azure AD to securely import/export a managed disk
description: Learn how to use Azure AD to securely import/export a disk.
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 04/01/2022
ms.author: rogarana
ms.subservice: disks
---

# Use Azure AD to securely import/export a managed disk (preview)

You can use Azure Active Directory (Azure AD) to secure the export and import of data to Azure managed disks. This feature is currently in preview. When using Azure AD, you can ensure that the system validates the identity of the requesting user in Azure AD, and that the user has the required permissions to export and import that disk. At a higher level, a system administrator could set a policy at the Azure account or subscription level to ensure that all disks and snapshots must use Azure AD for import or export.

## Pre-requisites

1. Email AzureDisks@microsoft .com to have the feature enabled on your subscription.
1. Install the latest [Azure PowerShell module](/powershell/azure/install-az-ps).
1. Install the [pre-release version](https://aka.ms/DisksAzureADAuthSDK) of the Az.Storage PowerShell module.

## Restrictions

- You can't upload a VHD to an empty snapshot, only empty disks.
- Only currently available in xyz regions.

## Get started

First, make sure the disk you're importing to or exporting from has its dataAccessAuthMode set to AzureActiveDirectory.

```azurepowershell
# Declare variables
$subscriptionID = "yourSubID"
$resourceGroupName = "yourRGName"
$diskName = "yourDiskName"

#set context to the appropriate subscription
set-AzContext -subscription $subscriptionID

# Switch an existing disk to AzureActiveDirectory to enable Azure AD access
New-AzDiskUpdateConfig -dataAccessAuthMode "AzureActiveDirectory" | Update-AzDisk -ResourceGroupName $resourceGroupName -DiskName $diskName;
```

Next, create a custom RBAC role with the permissions to allow disk import/export.

```azurepowershell
$role = [Microsoft.Azure.Commands.Resources.Models.Authorization.PSRoleDefinition]::new()
$role.Id = $null
$role.Name = "Disks Data Operator"
$role.Description = "A user in this role can read the data of the underlying VHD, export the VHD or upload a VHD to a disk, which is not attached to a running VM."
$role.IsCustom = $true
$perms = 'Microsoft.Compute/disks/download/action', 'Microsoft.Compute/disks/upload/action', 'Microsoft.Compute/snapshots/download/action', 'Microsoft.Compute/snapshots/upload/action'
$role.DataActions = $perms
$role.AssignableScopes = '/subscriptions/'+$subscriptionId

New-AzRoleDefinition -Role $role 
```

After that, assign RBAC permissions, to allow the role to access your disk.

```azurepowershell
$myDisk=Get-AzDisk -DiskName $diskName -ResourceGroupName $resourceGroup

New-AzRoleAssignment -SignInName <email address of the user> `
-RoleDefinitionName "Disks Data Operator" `
-Scope $myDisk.Id
```

To download the underlying VHD of your disk, generate the disk's SAS URI and then authenticate yourself using Azure AD.

```azurepowershell
$diskSas = Grant-AzDiskAccess -ResourceGroupName $resourceGroup -DiskName
$diskName -DurationInSecond 86400 -Access 'Read'

Connect-AzAccount

$localFolder = "desiredFilePath"

$blob = Get-AzStorageBlobContent -Uri $diskSas.AccessSAS -Destination $localFolder -Force
```

## Next steps

[Upload a VHD to Azure or copy a managed disk to another region - Azure PowerShell](windows/disks-upload-vhd-to-managed-disk-powershell.md)