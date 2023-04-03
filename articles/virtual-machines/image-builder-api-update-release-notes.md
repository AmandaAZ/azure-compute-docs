---
title: What's new in Azure VM Image Builder 
description: This article offers the latest release notes, known issues, bug fixes, deprecated functionality, and upcoming changes.
author: kof-f
ms.service: virtual-machines
ms.topic: conceptual
ms.workload: infrastructure
ms.date: 04/03/2023
ms.reviewer: erd
ms.subservice: image-builder
ms.custom: references_regions


---

# What's new in Azure VM Image Builder

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

This article contains all major API changes and feature updates for the Azure VM Image Builder (AIB) service.

## API releases


### Version 2022-07-01

**Improvements**
- Added support to pull the latest image version stored in Azure Compute Gallery to AIB templates
- Added support to pull the latest image version stored in Azure Compute Gallery as the source for the image template
- Added support for image versioning
- Added `DistributeVersioner` to support generating version numbers for image distributions. For more information, see [properties: DistributeVersioner](../../articles/virtual-machines/linux/image-builder-json.md#properties-distributeversioner)
- Added support for per region configuration when distributing to Azure Compute Gallery


**Changes**
- Added new 'File' validation type. For more information, see the [validate properties](/azure/virtual-machines/linux/image-builder-json?tabs=json%2Cazure-powershell#properties-validate&preserve-view=true)
- `replicationRegions` is deprecated for gallery distributions. For more information, use [gallery-replicated-regions](/cli/azure/image/builder/output?view=azure-cli-latest#az-image-builder-output-add-examples&preserve-view=true)
- VHDs can now be distributed to a custom blob or container in a custom storage account


### Version 2022-02-14

**Improvements**
- [Validation support](./linux/image-builder-json.md#properties-validate)
    - Shell (Linux): Script or inline
    - PowerShell (Windows): Script or inline, run elevated, run as system
    - Source-Validation-Only mode
- [Customized staging resource group support](./linux/image-builder-json.md#properties-stagingresourcegroup)

### Version 2021-10-01

**Breaking change**
 
API version 2021-10-01 introduces a change to the error schema that will be part of every future API release. If you have any Azure VM Image Builder automations, be aware of the [new error output](#error-output-for-version-2021-10-01-and-later) when you switch to API version 2021-10-01 or later. We recommend, after you've switched to the latest API version, that you don't revert to an earlier version, because you'll have to change your automation again to produce the earlier error schema. We don't anticipate that we'll change the error schema again in future releases.

##### **Error output for version 2020-02-14 and earlier**

```
{ 
  "code": "ValidationFailed",
  "message": "Validation failed: 'ImageTemplate.properties.source': Field 'imageId' has a bad value: '/subscriptions/subscriptionID/resourceGroups/resourceGroupName/providers/Microsoft.Compute/images/imageName'. Please review  http://aka.ms/azvmimagebuildertmplref  for details on fields requirements in the Image Builder Template." 
} 
```

##### **Error output for version 2021-10-01 and later**

```
{ 
  "error": {
    "code": "ValidationFailed", 
    "message": "Validation failed: 'ImageTemplate.properties.source': Field 'imageId' has a bad value: '/subscriptions/subscriptionID/resourceGroups/resourceGroupName/providers/Microsoft.Compute/images/imageName'. Please review  http://aka.ms/azvmimagebuildertmplref  for details on fields requirements in the Image Builder Template." 
  }
}
```

**Improvements**

- Added support for [Build VM MSIs](linux/image-builder-json.md#user-assigned-identity-for-the-image-builder-build-vm).
- Added support for Proxy VM size customization.

### Version 2020-02-14

**Improvements**

- Added support for creating images from the following sources:
    - Managed image
    - Azure Compute Gallery
    - Platform Image Repository (including Platform Image Purchase Plan)
- Added support for the following customizations:
    - Shell (Linux): Script or inline
    - PowerShell (Windows): Script or inline, run elevated, run as system
    - File (Linux and Windows)
    - Windows Restart (Windows)
    - Windows Update (Windows): Search criteria, filters, and update limit
- Added support for the following distribution types:
    - VHD (virtual hard disk)
    - Managed image
    - Azure Compute Gallery
- Other features:
    - Added support for customers to use their own virtual network
    - Added support for customers to customize the build VM (VM size, operating system disk size)
    - Added support for user-assigned Microsoft Windows Installer (MSI) (for customize/distribute steps)
    - Added support for [Gen2 images](image-builder-overview.md#hyper-v-generation)

### Preview APIs

 The following APIs are deprecated, but still supported:
- Version 2019-05-01-preview


## Next steps
Learn more about [VM Image Builder](image-builder-overview.md).
