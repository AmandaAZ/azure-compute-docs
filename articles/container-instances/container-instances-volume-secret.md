---
title: Mount secret volume in Azure Container Instances
description: Learn how to mount a secret volume to store sensitive information for access by your container instances
services: container-instances
author: mmacy
manager: timlt

ms.service: container-instances
ms.topic: article
ms.date: 02/07/2018
ms.author: marsma
---

# Mount a secret volume in Azure Container Instances

Learn how to mount a secret volume in your container instances for the storage and retrieval of sensitive information by the containers in your container groups.

> [!NOTE]
> Mounting a secret volume is currently restricted to Linux containers. While we are working to bring all features to Windows containers, you can find current platform differences in [Quotas and region availability for Azure Container Instances](container-instances-quotas.md).

## Mount a secret volume

To mount a secret volume in a container instance, you must deploy using an [Azure Resource Manager template](/azure/templates/microsoft.containerinstance/containergroups).

First, populate the `volumes` array in the `properties` section of the template. Next, for each container in the container group in which you'd like to mount the *secret* volume, populate the `volumeMounts` array in the `properties` section of the container definition.

For example, following Resource Manager template creates container group consisting of a single container. The container mounts a *secret* volume consisting of two Base64-encoded secrets.

[!code-json[volume-secret](~/azure-docs-json-samples/container-instances/aci-deploy-volume-secret.json)]

To see an example of container instance deployment with an Azure Resource Manager template, see [Deploy multi-container groups in Azure Container Instances](container-instances-multi-container-group.md).

## Next steps

Learn how to mount other volume types in Azure Container Instances:

* [Mount an Azure file share in Azure Container Instances](container-instances-volume-azure-files.md)
* [Mount an emptyDir volume Azure Container Instances](container-instances-volume-emptydir.md)
* [Mount a gitRepo volume Azure Container Instances](container-instances-volume-gitrepo.md)
