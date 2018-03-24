---
title: Azure Service Fabric cluster scaling | Microsoft Docs
description: Learn about scaling Service Fabric clusters in or out and up or down.
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''

ms.assetid: 5441e7e0-d842-4398-b060-8c9d34b07c48
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/23/2018
ms.author: ryanwi

---
# Scaling Service Fabric clusters
A Service Fabric cluster is a network-connected set of virtual or physical machines into which your microservices are deployed and managed. A machine or VM that's part of a cluster is called a node. Clusters can contain potentially thousands of nodes. After creating a Service Fabric cluster, you can scale the cluster by increasing or decreasing the number of cluster nodes or increasing or decreasing the resources of the nodes.  You can scale the cluster at any time, even when workloads are running on the cluster.  As the cluster scales, your applications automatically scale as well.

Why scale the cluster?
- Increase the number of cluster nodes if the cluster's resources are almost all consumed. Once the new nodes join the cluster the [Cluster Resource Manager](service-fabric-cluster-resource-manager-introduction.md) moves services to them, reducing load on the existing nodes.
- Decrease the number of nodes if the cluster's resources are not being used efficiently.  As nodes leave the cluster, services move off those nodes and load increases on the remaining nodes.  Reducing the number of nodes in a cluster running in Azure can save you money, since you pay for the number of VMs you use and not the workload on those VMs.
- Increase the resources of the nodes
- Decrease the resources of the nodes

## Scaling an Azure cluster in or out
Virtual machine scale sets are an Azure compute resource that you can use to deploy and manage a collection of virtual machines as a set. Every node type that is defined in an Azure cluster is set up as a separate virtual machine scale set. Each node type can then be scaled in or out independently, have different sets of ports open, and can have different capacity metrics. Read more about it in the [Service Fabric nodetypes](service-fabric-cluster-nodetypes.md) document. 

Guidelines:
- Maintain a minimum count of five nodes for any virtual machine scale set that has durability level of Gold or Silver enabled
- Do not delete random VM instances, always use virtual machine scale set scale down feature. The deletion of random VM instances has a potential of creating imbalances in the VM instance spread across UD and FD. This imbalance could adversely affect the systems ability to properly load balance amongst the service instances/Service replicas.
- If using Autoscale, then set the rules such that scale in (removing of VM instances) are done only one node at a time. Scaling down more than one instance at a time is not safe.


Since the Service Fabric node types in your cluster are made up of virtual machine scale sets at the backend, you can [set up auto-scale rules or manually scale](service-fabric-cluster-scale-up-down.md) each node type/virtual machine scale set.

### Programmatic scaling
In many scenarios, [Scaling a cluster manually or with autoscale rules](service-fabric-cluster-scale-up-down.md) are good solutions. For more advanced scenarios, though, they may not be the right fit. Potential drawbacks to these approaches include:

- Manually scaling requires you to log in and explicitly request scaling operations. If scaling operations are required frequently or at unpredictable times, this approach may not be a good solution.
- When auto-scale rules remove an instance from a virtual machine scale set, they do not automatically remove knowledge of that node from the associated Service Fabric cluster unless the node type has a durability level of Silver or Gold. Because auto-scale rules work at the scale set level (rather than at the Service Fabric level), auto-scale rules can remove Service Fabric nodes without shutting them down gracefully. This rude node removal will leave 'ghost' Service Fabric node state behind after scale-in operations. An individual (or a service) would need to periodically clean up removed node state in the Service Fabric cluster.
- A node type with a durability level of Gold or Silver automatically cleans up removed nodes, so no additional clean-up is needed.
- Although there are [many metrics](../monitoring-and-diagnostics/insights-autoscale-common-metrics.md) supported by auto-scale rules, it is still a limited set. If your scenario calls for scaling based on some metric not covered in that set, then auto-scale rules may not be a good option.

Azure APIs exist which allow applications to programmatically work with virtual machine scale sets and Service Fabric clusters. If existing auto-scale options don't work for your scenario, these APIs make it possible to implement custom scaling logic. 

One approach to implementing this 'home-made' auto-scaling functionality is to add a new stateless service to the Service Fabric application to manage scaling operations. Within the service's `RunAsync` method, a set of triggers can determine if scaling is required (including checking parameters such as maximum cluster size and scaling cooldowns).   

The API used for virtual machine scale set interactions (both to check the current number of virtual machine instances and to modify it) is the [fluent Azure Management Compute library](https://www.nuget.org/packages/Microsoft.Azure.Management.Compute.Fluent/). The fluent compute library provides an easy-to-use API for interacting with virtual machine scale sets.

To interact with the Service Fabric cluster itself, use [System.Fabric.FabricClient](/dotnet/api/system.fabric.fabricclient).

Of course, the scaling code doesn't need to run as a service in the cluster to be scaled. Both `IAzure` and `FabricClient` can connect to their associated Azure resources remotely, so the scaling service could easily be a console application or Windows service running from outside the Service Fabric application.

Based on these limitations, you may wish to [implement more customized automatic scaling models](service-fabric-programmatic-scaling.md).

## Scaling a standalone cluster in or out


## Scaling an Azure cluster up or down
Changing the VM SKU of a Virtual Machine Scale Set is inherently an unsafe operation and so should be avoided if possible. Here is the process you can follow to avoid common issues.
- **For non-primary node types:** It is recommended that you create a new node type, modify the service placement constraint to include the new node type and then reduce the old node type instance count to 0, one node at a time (this is to make sure that removal of the nodes do not impact the reliability of the cluster).
- **For the primary node type:** Our recommendation is that you do not change VM SKU of the primary node type. Changing of the primary node type SKU is not supported. If the reason for the new SKU is capacity, we recommend adding more instances. If that not possible, create a new cluster and [restore application state](service-fabric-reliable-services-backup-restore.md) (if applicable) from your old cluster. You do not need to restore any system service state, they are recreated when you deploy your applications to your new cluster. If you were just running stateless applications on your cluster, then all you do is deploy your applications to the new cluster, you have nothing to restore. If you decide to go the unsupported route and want to change the VM SKU, then make modifications to the Virtual Machine Scale Set Model definition to reflect the new SKU. If your cluster has only one node type, then make sure that all your stateful applications respond to all [Service replica lifecycle events](service-fabric-reliable-services-lifecycle.md) (like replica in build is stuck) in a timely fashion and that your service replica rebuild duration is less than five minutes (for Silver durability level).

> [!WARNING]
> Changing the VM SKU Size for virtual machine scale sets not running at least Silver durability is not recommended. Changing VM SKU Size is a data-destructive in-place infrastructure operation. Without at least some ability to delay or monitor this change, it is possible that the operation can cause data loss for stateful services or cause other unforeseen operational issues, even for stateless workloads. 
> 


7. If Scaling down a primary node type, you should never scale it down more than what the reliability tier allows

## Scaling a standalone cluster up or down


## Next steps
* Learn about [application scalability](service-fabric-concepts-scalability.md)
* [Scale an Azure cluster in or out](service-fabric-tutorial-scale-cluster.md)
* [Scale an Azure cluster programmatically](service-fabric-cluster-programmatic-scaling.md)
* [Scale a standaone cluster in or out](service-fabric-cluster-windows-server-add-remove-nodes.md)

