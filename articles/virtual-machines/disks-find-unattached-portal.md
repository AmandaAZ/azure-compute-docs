---
title: Identify unattached Azure disks - Azure Portal
description: How to find unattached Azure managed and unmanaged (VHDs/page blobs) disks by using the Azure portal.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 05/28/2020
ms.author: rogarana
ms.subservice: disks
---

# Find and delete unattached Azure managed and unmanaged disks

When you delete a virtual machine (VM) in Azure, by default, any disks that are attached to the VM aren't deleted. This helps to prevent data loss due to the unintentional deletion of VMs. After a VM is deleted, you will continue to pay for unattached disks. This article shows you how to find and delete any unattached disks and reduce unnecessary costs.

## Managed disks: Find and delete unattached disks

Finding unattached disks in the Azure portal is easy.

1. Sign in to the Azure portal
1. Search for and select **Disks**.

    You are presented with a list of all your disks. Any disk which has - in the Owner column is an unattached disk.

    <image here>

1. Select the unattached disk you'd like to delete, and select delete.
1. Enter deletion information.


## Unmanaged disks: Find and delete unattached disks

Unmanaged disks are VHD files that are stored as [page blobs](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-page-blobs) in [Azure storage accounts](../../storage/common/storage-create-storage-account.md).


## Next steps

For more information, see [Delete storage account](../../storage/common/storage-create-storage-account.md) and [Identify Orphaned Disks Using PowerShell](https://blogs.technet.microsoft.com/ukplatforms/2018/02/21/azure-cost-optimisation-series-identify-orphaned-disks-using-powershell/)