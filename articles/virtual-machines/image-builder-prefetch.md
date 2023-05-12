--- 
title: VM Boot Optimization for Azure Compute Gallery Images with Azure VM Image Builder 
description: Optimize VM Boot and Provisioning time with Azure VM Image Builder 
ms.author: surbhijain 
ms.reviewer: kofiforson 
ms.date: 04/05/2023 
ms.topic: how-to 
ms.service: virtual-machines 
ms.subservice: image-builder
--- 

  

# VM optimization for gallery images with Azure VM Image Builder 

  **Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Virtual Machine Scale Sets 

In this article, you learn how to use Azure VM Image Builder to optimize your ACG (Azure Compute Gallery) or Managed Images or VHDs to improve the create time for your VMs. 

## Azure VM Optimization 
Azure VM optimization improves virtual machine creation time by updating the gallery image to optimize the image for a faster boot time. 

## Image types supported 

Optimization for the following images is supported: 

| Features  | Details   |
|---|---|
|OS Type| Linux/Windows |
| Partition | MBR/GPT |
| Hyper-V | Gen1/Gen2 |
| OS State | Generalized |

The following types of images aren't supported: 

* Images with size greater than 2 TB 
* ARM64 images 
* Specialized images

 

## Optimization in Azure VM Image Builder 

Optimization can be enabled while creating a VM image using the CLI. 

Customers can create an Azure VM Image Builder template using CLI. It contains details regarding source, type of customization and distribution. For more information on how to create an image builder template, see [Create an Azure Image Builder Bicep or ARM JSON template](/azure/virtual-machines/linux/image-builder-json). 

To optimize the image, you can enable additional fields in the template](insert AIB repo link) like shown in below snippet. 


```json 

 "optimize": { 

      "vmboot": { 

        "state": "Enabled" 

      } 

    } 

``` 

> [!NOTE]
> Use API Version `2022-07-01` or beyond to avail optimization benefits.

  

## FAQs 

  

### Can VM optimization be used without Azure VM Image Builder customization? 

  

Yes, customers can opt for only VM optimization without using Azure VM Image Builder customization feature. Customers can simply enable the optimization flag and keep customization field as empty.  

  

### Can an existing ACG image version be optimized? 

No, this optimization feature won't update an existing SIG image version. However, optimization can be enabled during new version creation for an existing image 

  

## How much time does it take for generating an optimized image? 

 

 The below latencies have been observed at various percentiles: 

| OS | Size | P50 | P95 | Average |
| --- | --- | --- | --- | --- |
| Linux | 30 GB VHD | 20 mins | 21 mins | 20 mins |
| Windows | 127 GB VHD | 34 mins | 35 mins | 33 mins |

  

This is the end to end duration observed. Note, image generation duration varies based on different factors such as, OS Type, VHD size, OS State, etc. 

  

### Is OS image copied out of customer subscription for optimization? 

Yes, the OS VHD is copied from customer subscription to Azure subscription for optimization in the same geographic location. Once optimization is finished or timed out, Azure internally deletes all copied OS VHDs. 

  

## Next steps 

  

Learn more about [Azure Compute Gallery](../virtual-machines/azure-compute-gallery.md).
