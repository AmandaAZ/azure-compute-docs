---
title: Share resources in an Azure Compute Gallery
description: Learn how to share resources explicitly or to all Azure users using role-based access control or community galleries.
author: sandeepraichura
ms.service: virtual-machines
ms.subservice: gallery
ms.topic: how-to
ms.workload: infrastructure
ms.date: 06/20/2022
ms.author: saraic
ms.reviewer: cynthn
ms.custom: template-how-to , devx-track-azurecli 
ms.devlang: azurecli

---

# Share a gallery with subscriptions or tenants

This article covers how to share an Azure Compute Gallery with specific subscriptions or tenants using direct sharing.


> [!IMPORTANT]
> Azure Compute Gallery – direct sharing is currently in PREVIEW and subject to the [Preview Terms for Azure Compute Gallery](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
> 
> During the preview, you need to create a new gallery, with the property `sharingProfile.permissions` set to `Groups`. When using the CLI to create a gallery, use the `--permissions groups` parameter. You can't use an existing gallery, the property can't currently be updated.
>
> You can't currently create a Flexible virtual machine scale set from an image shared to you by another tenant.


There are three main ways to share images in an Azure Compute Gallery, depending on who you want to share with:

| Who? | Option |
|----|----|
| [Specific people, groups, or service principals](./share-gallery.md) | Role-based access control (RBAC) lets you share resources to specific people, groups, or service principals on a granular level. |
| [Subscriptions or tenants](explained in this article) | Direct sharing lets you share to everyone in a subscription or tenant. |
| [Everyone](./share-gallery-community.md) | Community gallery lets you share your entire gallery publicly, to all Azure users. |


## Limitations

During the preview:
- You can only share to subscriptions that are also in the preview.
- You can only share to 30 subscriptions and 5 tenants.
- The gallery using direct sharing cannot contain encrypted image versions. Encrypted images cannot be created within a gallery that is directly shared.
- The user or service principal that will share must be a member of the `Owner` role definition. Only an `Owner` at the scope of the gallery or higher will be able to enable group-based sharing.
- You need to create a new gallery,  with the property `sharingProfile.permissions` set to `Groups`. When using the CLI to create a gallery, use the `--permissions groups` parameter. You can't use an existing gallery, the property can't currently be updated.

## Prerequisites

You need to create a [new gallery with direct sharing enabled](./create-gallery.md#create-a-direct-sharing-gallery). Direct sharing means that the `sharingProfile.permissions` property is set to `Groups`. When using the CLI to create a gallery, use the `--permissions groups` parameter. You can't use an existing gallery, the property can't currently be updated.
### [Portal](#tab/portaldirect)

1. Sign in to the Azure portal at https://portal.azure.com.
1. Type **Azure Compute Gallery** in the search box and select **Azure Compute Gallery** in the results.


### [CLI](#tab/clidirect)

To create a gallery that can be shared to a subscription or tenant using direct sharing, you need to create the gallery with the `--permissions` parameter set to `groups`.

```azurecli-interactive
az sig create \
   --gallery-name myGallery \
   --permissions groups \
   --resource-group myResourceGroup  
```
 

To start sharing the gallery with a subscription or tenant, use [az sig share add](/cli/azure/sig#az-sig-share-add) 

```azurecli-interactive
sub=<subscription-id>
tenant=<tenant-id>
gallery=<gallery-name>
rg=<resource-group-name>
az sig share add \
   --subscription-ids $sub \
   --tenant-ids $tenant \
   --gallery-name $gallery \
   --resource-group $rg
```
 

Remove access for a subscription or tenant using [az sig share remove](/cli/azure/sig#az-sig-share-remove).

```azurecli-interactive
sub=<subscription-id>
tenant=<tenant-id>
gallery=<gallery-name>
rg=<resource-group-name>

az sig share remove \
   --subscription-ids $sub \
   --tenant-ids $tenant \
   --gallery-name $gallery \
   --resource-group $rg
```
 


 
### [REST](#tab/restdirect)

Create a gallery for subscription or tenant-level sharing using the Azure REST API.

```rest
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}?api-version=2020-09-30

{
	"properties": {
		"sharingProfile": {
			"permissions": "Groups"
		}
	},
	"location": "{location}
}
 
```


Share a gallery to subscription or tenant.


```rest
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{galleryName}/share?api-version=2020-09-30

{
  "operationType": "Add",
  "groups": [
    {
      "type": "Subscriptions",
      "ids": [
        "{SubscriptionID}"
      ]
    },
    {
      "type": "AADTenants",
      "ids": [
        "{tenantID}"
      ]
    }
  ]
}

```
 

Remove access for a subscription or tenant.

```rest
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{galleryName}/share?api-version=2020-09-30

{
	"operationType": "Remove",
	"groups":[ 
		{
			"type": "Subscriptions",
			"ids": [
				"{subscriptionId1}",
				"{subscriptionId2}"
			],
},
{
			"type": "AADTenants",
			"ids": [
				"{tenantId1}",
				"{tenantId2}"
			]
		}
	]
}

```


Reset sharing to clear everything in the `sharingProfile`.

```rest
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{galleryName}/share?api-version=2020-09-30 

{ 
 "operationType" : "Reset", 
} 
```

---


## Next steps

Create an [image definition and an image version](image-version.md).

You can also create Azure Compute Gallery resources using templates. There are several quickstart templates available:

- [Create an Azure Compute Gallery](https://azure.microsoft.com/resources/templates/sig-create/)
- [Create an Image Definition in an Azure Compute Gallery](https://azure.microsoft.com/resources/templates/sig-image-definition-create/)
- [Create an Image Version in an Azure Compute Gallery](https://azure.microsoft.com/resources/templates/sig-image-version-create/)
