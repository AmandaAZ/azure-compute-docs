---
title: Azure Service Fabric Infrastucture as Code Best Practices | Microsoft Docs
description: Best practices for managing Service Fabric as infrastructure as code.
services: service-fabric
documentationcenter: .net
author: peterpogorski
manager: timlt
editor: ''

ms.assetid: 19ca51e8-69b9-4952-b4b5-4bf04cded217
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: 
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 01/11/2019
ms.author: pepogors

---
# Infrastructure as Code 
In a production scenario, Azure Service Fabric clusters should be created using Resource Manager templates. Resource Manager templates allow for a greater level of control of resource properties and ensures that you have a consistent resource model. Resource 
```bash
ResourceGroupName="sfclustergroup"

az group deployment create --resource-group {}
```

-- Sample repo link and API documentation. Modify these samples for these needs.
-- Portal should not be used to provision a production ready cluster.
-- deployment using AZ CLI and Powershell
-- The create SF powershell cmdlet should only be used for dev test

## Service Fabric ARM Resources 
-- JSON Snippet of the types of SF 
-- Cluster, Application, Application Version, Application Type, Service Version, Service Type
-- Link to packaging, explicity put Zip File create from directory. Need to do this to use this resource type. If you are on Linux use a different ZIP file utility. Put the Python snippet.


## Next steps

* Create a cluster on VMs or computers running Windows Server: [Service Fabric cluster creation for Windows Server](service-fabric-cluster-creation-for-windows-server.md)
* Create a cluster on VMs or computers running Linux: [Create a Linux cluster](service-fabric-cluster-creation-via-portal.md)
* Learn about [Service Fabric support options](service-fabric-support.md)