﻿---
title: Service Fabric CLI Script Sample - Deploy application to a cluster
description: Service Fabric CLI Script Sample - Deploy an application to a Service Fabric cluster.
services: service-fabric
documentationcenter: 
author: Thraka
manager: timlt
editor: 
tags: azure-service-management

ms.assetid: 
ms.service: service-fabric
ms.workload: multiple
ms.devlang: na
ms.topic: article
ms.date: 07/21/2017
ms.author: adegeo
ms.custom: mvc
---

# Deploy an application to a Service Fabric cluster

This sample script copies an application package to a cluster image store, registers the application type in the cluster, and creates an application instance from the application type.  If any default services were defined in the application manifest of the target application type, then those services are created at this time.

If needed, install the [Service Fabric CLI](../service-fabric-cli.md).

## Sample script

[!code-sh[main](../../../cli_scripts/service-fabric/deploy-application/deploy-application.sh "Deploy an application to a cluster")]

## Clean up deployment 

After the script sample has been run, the script in [Remove an application](cli-remove-application.md) can be used to remove the application instance, unregister the application type, and delete the application package from the image store.

## Script explanation

This script uses the following commands. Each command in the table links to command specific documentation.


| Command | Notes |
|---|---|
| [sfctl cluster select](/cli/azure/sf/cluster#select) | Selects the cluster to work with. |
| [sfctl application upload](/cli/azure/sf/application#upload) | Upload the app files and manifests. |
| [sfctl application provision](/cli/azure/sf/application#provision) | Register the application on the cluster.|
| [sfctl application create](/cli/azure/sf/application#create) | Create an instance of the application and deploy any defined services to the nodes. |

## Next steps

For more information, see the [Service Fabric CLI documentation](../service-fabric-cli.md).

Additional Service Fabric CLI samples for Azure Service Fabric can be found in the [Service Fabric CLI samples](../samples-cli.md).
