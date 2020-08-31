---
title: Add and remove node types of a Managed Service Fabric cluster (preview)
description: In this tutorial, learn how to add and remove node types of a Managed Service Fabric cluster.
ms.topic: tutorial
ms.date: 07/31/2020
---

# Tutorial: Add and remove node types of a Managed Service Fabric cluster (preview)

In this tutorial series we will discuss:

> [!div class="checklist"]
> * [How to deploy a Managed Service Fabric cluster.](tutorial-managed-cluster-deploy.md) 
> * [How to scale out a Managed Service Fabric cluster](tutorial-managed-cluster-scale.md)
> * How to add and remove nodes in a Managed Service Fabric cluster
> * [How to add a certificate to a Managed Service Fabric cluster](tutorial-managed-cluster-certificate.md)
> * [How to upgrade your Managed Service Fabric cluster resources](tutorial-managed-cluster-upgrade.md)

This part of the series covers how to:

> [!div class="checklist"]
> * Add a node type to a Managed Service Fabric cluster
> * Delete a node type from a Managed Service Fabric cluster

## Prerequisites
> [!Note]
> This tutorial uses Azure PowerShell commands which have not yet been released. They will become released as part of the Azure PowerShell module on 9/16/2020.

Follow the steps below to use the module before the official release is available:
* [Download and load Modules](https://github.com/a-santamaria/ServiceFabricManagedClustersClients#download-and-load-modules)
* [Documentation and Examples](https://github.com/a-santamaria/ServiceFabricManagedClustersClients#documentation-and-examples). 


## Add a node type to a Managed Service Fabric cluster

You can add a node type to a managed Service Fabric cluster through an Azure Resource Manager template, PowerShell, or CLI. In this tutorial we will be adding a node type using the Azure PowerShell.

To create a new node type, we will need to define three properties:
* **Node Type Name**: This should be a unique name from any other node types that already exist in the cluster. 
* **Instance Count**: This will be the initial number of nodes in the new node type. 
* **VM Size**: This will be the VM SKU which the nodes are running on. If this property is not specified the default value with be a Standard_D2. 

```powershell
$resourceGroup = "myResourceGroup"
$clusterName = "myCluster"
$nodeTypeName = "nt2"
$vmSize = "Standard_D2_v2"

New-AzServiceFabricManagedNodeType -ResourceGroupName $resourceGroup -ClusterName $clusterName -Name $nodeTypeName -InstanceCount 3 -vmSize $vmSize
```

## Remove a node type from a Managed Service Fabric cluster

To remove a node type from a manged Service Fabric cluster, you must use PowerShell or CLI. In this tutorial we will be remove a node type using the Azure PowerShell. 

> [!Note]
> It is not possible to remove a primary node type if it is the only primary node type in the cluster.  

To remove a node type:

```powershell
$resourceGroup = "myResourceGroup"
$clusterName = "myCluster"
$nodeTypeName = "nt3"

Remove-AzServiceFabricManagedNodeType -ResourceGroupName $resourceGroup -ClusterName $clusterName  -Name $nodeTypeName
```

<!-- This command is only used if the customer does not have the PowerShell module installed.-->
To delete a node type obtaining the resource ID for the target node type, simply remove the resource.

```powershell
Remove-AzResource -ResourceId <your-resource-id> -ApiVersion 2020-01-01-preview
```

## Next steps

In this step we added and deleted node types. To learn more about how to use client certificates, see:

> [!div class="nextstepaction"]
> [Add a client certificate to a Manged Service Fabric cluster](./tutorial-managed-cluster-certificate.md)