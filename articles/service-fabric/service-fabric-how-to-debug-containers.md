---
title: How to debug containers with Azure Service Fabric and Visual Studio 2017 | Microsoft Docs
description: Shows you how to debug containers in Azure Service Fabric and Visual Studio 2017
documentationcenter: .net
author: mikkelhegn
manager: msfussell
editor: ''

ms.service: service-fabric

ms.devlang: dotNet
ms.topic: article
ms.tgt_pltfrm: NA

ms.workload: NA
ms.date: 05/14/2018
ms.author: mikhegn

---
# How to debug containers in Azure Service Fabric on Windows

The below procedure shows you how to debug a .NET application in a container running in local Service Fabric cluster, using Visual Studio 2017.

## Debug a .NET application running in docker containers on Service Fabric

> [!NOTE]
> To complete the below, you need to have your development environemtn configured according to this article: [How to configure your developer environment to debug containers in Azure Service Fabric on Windows](service-fabric-how-to-configure-for-container-debugging.md).
>

1. Run Visual Studio as an administrator.

1. Open an existing .NET application or create a new one.

1. Right-click the project and select **Add -> Container Orchestrator Support -> Service Fabric**

1. Press **F5** to start debugging the application.

> [!NOTE]
> Visual Studio supports console and ASP.NET project types for .NET and .NET Core
>

## More Information
To learn how to configure your developer environment to debug .NET applications in containers, see the [How to configure your developer environment to debug containers in Azure Service Fabric on Windows](service-fabric-how-to-configure-for-container-debugging.md).