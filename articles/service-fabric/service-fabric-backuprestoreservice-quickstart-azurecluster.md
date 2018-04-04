---
title: Periodic Backup and Restore in Azure Service Fabric | Microsoft Docs
description: Use Service Fabric's Periodic Backup and Restore feature for protecting your applications from Data Loss.
services: service-fabric
documentationcenter: .net
author: hrushib
manager: timlt
editor: hrushib

ms.assetid: FAA58600-897E-4CEE-9D1C-93FACF98AD1C
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/04/2018
ms.author: hrushib

---
# Periodic Backup and Restore in Azure Service Fabric
> [!div class="op_single_selector"]
> * [Service Fabric Azure Cluster](service-fabric-backuprestoreservice-quickstart-azurecluster.md) 
> * [Service Fabric Standalone Cluster](service-fabric-backuprestoreservice-quickstart-standalonecluster.md)
> 

Service fabric is a distributed systems platform that makes it easy to develop and manage reliable distributed, micro-services based cloud applications. It allows running of both stateless and stateful micro services. Stateful micro-services maintain a mutable, authoritative state beyond the request and response or a complete transaction. If this service went down for some time, it would need to be initialized with the state at the time of it going down, for it to provide meaningful and expected service after it comes up.

Service fabric replicates the state across multiple nodes to ensure that the service is highly available. Even if one node in the cluster fails, the service continues to be available. In certain cases, however, it is still desirable for the service data to be backed up. As an example, if entire service fabric cluster is lost, or for cases outside of the control of service fabric, ex. Accidental corruption or deletion of data.

Service Fabric today provides an inbuilt API to do point in time backup and restore. Application developers must take care of backing up the state of their services by invoking this API. Additionally, if they want to trigger it from outside of the application at a specific time, like before upgrading the application, they need to expose Backup (and Restore) as an API themselves which they can invoke. This is just to take backup and trigger restore. Maintaining the backup points is an additional cost above this. All this means, additional code for application developer.

Backup of the application data on a periodic basis is a basic need for managing a distributed application and guarding against loss of data or prolonged loss of service availability. To help achieve the above, Service Fabric provides Periodic Backup and Restore feature which allows you to configure periodic backup of stateful applications without having to write any additional code. And also facilitates to Restore previously taken backup to the service partition. 

Service Fabric provides set of REST APIs to achieve the following functionality related to Periodic Backup and Restore feature:

- Scheduled Periodic Backup of reliable stateful services and reliable persisted actors.
- Supported backup locations - Azure Storage and File Share (on-premise)
- Enumerate Backups
- Trigger an ad-hoc backup of a partition
- Restore partition from a previous backup point
- Temporarily suspend backups of a partition
- Retention management of backups (upcoming)

## Enabling the Periodic Backup and Restore feature
First you need to enable the Backup Restore service in your cluster. Get the template for the cluster that you want to deploy. You can either use the [sample templates](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype)  or create a Resource Manager template. You can enable the Backup Restore service with the following steps:

1. Check that the `apiversion` is set to **`2018-04-01-preview ??`** for the `Microsoft.ServiceFabric/clusters` resource, and if not, update it as shown in the following snippet:

    ```json
    {
        "apiVersion": "2018-04-01-preview",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
    }
    ```

2. Now enable the Backup Restore service by adding the following `addonFeatures` section under `properties` section as shown in the following snippet: 

    ```json
        "properties": {
            ...
            "addonFeatures":  ["BackupRestoreService"],
            "fabricSettings": [ ... ]
            ...
        }

    ```

3. Once you have updated your cluster template with the preceding changes, apply them and let the deployment/upgrade complete. Once complete, the Backup Restore system service starts running in your cluster. The uri of this service is `fabric:/System/BackupRestoreService` and this can be loacted under system service section in the Service Fabric explorer. 

Alternatively, you can enable the Backup Restore service through the portal at the time of cluster creation. (upcoming)


## Enabling backups for reliable stateful service
Let's walkthrough steps to enable Periodic Backup for reliable statefule service. These steps assume
- That the local cluster is setup with Periodc Backup and Restore feature.
- One application with reliable service is deployed on the cluster. Lets assume that the application Uri is `fabric:/SampleApp` and the uri for reliable service belonging to this app is `fabric:/SampleApp/MyStatefulService` with single partition with partition ID `23aebc1e-e9ea-4e16-9d5c-e91a614fefa7`.  

### Policy creation

First step is to create policy describing backup schedule and target storage for backup data. Execute following powershell script for invoking required REST API to create new policy.
```powershell
$StorageInfo = @{
    Path = '\\localhost\BackupStore'
    StorageKind = 'FileShare'
}

$ScheduleInfo = @{
    IntervalType = 'Minutes'
    Interval = 1
    ScheduleKind = 'FrequencyBased'
}   

$BackupPolicy = @{
    Name = 'BackupPolicy1'
    MaxIncrementalBackups = 20
    Schedule = $ScheduleInfo
    Storage = $StorageInfo
}

$body = (ConvertTo-Json $BackupPolicy)
$url = "http://localhost:19080/BackupRestore/BackupPolicies/$/Create?api-version=6.2-preview&timeout=1000"

try
{
    $response  = Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json'
    
    If ($response.StatusCode -eq 201)
    {
        Write-Host "Policy creation request accepted by server."
    }
}
catch
{
    If ($_.Exception.Response.StatusCode.value__ -eq 409)
    {
        Write-Host "Policy already exists with given name, can not create another policy with same name."
    }
    Else
    {
        Write-Host "Unexpected exception during policy creation:" $_.Exception
    }
}

```

### Enable protection
After defining policy for backup needs of the application, the policy needs to be associated with Application(s). Execute following powershell script for invoking required REST API to associate backup policy with name `BackupPolicy1` created in above step with Application `SampleApp`.
```powershell
$BackupPolicyReference = @{
    BackupPolicyName = 'BackupPolicy1'
}

$body = (ConvertTo-Json $BackupPolicyReference)
$url = "http://localhost:19080/Applications/SampleApp/$/EnableBackup?api-version=6.2-preview&timeout=1000"

try
{
    $response  = Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json'
    
    If ($response.StatusCode -eq 202)
    {
        Write-Host "Enable backup request accepted by server."
    }
}
catch
{
    Write-Host "Unexpected exception while enabling backup:" $_.Exception
}

``` 
### Know if periodic backups are working
After enabling backup for the application, all partitions belonging to stateful reliable services and reliable Actors under the application will start getting backed-up periodically as per the associated policy. 

![Partition BackedUp Health Event][0]
### List Backups
```powershell
$BackupPolicyReference = @{
	 BackupPolicyName = 'BackupPolicy1'
}

$body = (ConvertTo-Json $BackupPolicyReference)
$url = "http://localhost:19080/Applications/SampleApp/$/GetBackups?api-version=6.2-preview&timeout=10000"

try
{
    $response  = Invoke-WebRequest -Uri $url -Method Get
    
    If ($response.StatusCode -eq 200)
    {
        $BackupPoints = (ConvertFrom-Json $response.Content)
        $BackupPoints.Items | Format-List
    }
}
catch
{
    Write-Host "Unexpected exception while getting backup enumerations:" $_.Exception
}

```
```
BackupId                : d7e4038e-2c46-47c6-9549-10698766e714
BackupChainId           : d7e4038e-2c46-47c6-9549-10698766e714
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=23aebc1e-e9ea-4e16-9d5c-e91a614fefa7}
BackupLocation          : SampleApp\MyStatefulService\23aebc1e-e9ea-4e16-9d5c-e91a614fefa7\2018-04-01 19.39.40.zip
BackupType              : Full
EpochOfLastBackupRecord : @{DataLossNumber=131670844862460432; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 2058
CreationTimeUtc         : 2018-04-01T19:39:40Z
FailureError            : 

BackupId                : 8c21398a-2141-4133-b4d7-e1a35f0d7aac
BackupChainId           : d7e4038e-2c46-47c6-9549-10698766e714
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=23aebc1e-e9ea-4e16-9d5c-e91a614fefa7}
BackupLocation          : SampleApp\MyStatefulService\23aebc1e-e9ea-4e16-9d5c-e91a614fefa7\2018-04-01 19.40.38.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131670844862460432; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 2237
CreationTimeUtc         : 2018-04-01T19:40:38Z
FailureError            : 

BackupId                : fc75bd4c-798c-4c9a-beee-e725321f73b2
BackupChainId           : d7e4038e-2c46-47c6-9549-10698766e714
ApplicationName         : fabric:/SampleApp
ServiceName             : fabric:/SampleApp/MyStatefulService
PartitionInformation    : @{LowKey=-9223372036854775808; HighKey=9223372036854775807; ServicePartitionKind=Int64Range; Id=23aebc1e-e9ea-4e16-9d5c-e91a614fefa7}
BackupLocation          : SampleApp\MyStatefulService\23aebc1e-e9ea-4e16-9d5c-e91a614fefa7\2018-04-01 19.41.44.zip
BackupType              : Incremental
EpochOfLastBackupRecord : @{DataLossNumber=131670844862460432; ConfigurationNumber=8589934592}
LsnOfLastBackupRecord   : 2437
CreationTimeUtc         : 2018-04-01T19:41:44Z
FailureError            : 
```

## Preview Limitation/ Caveats
- No support for Powershell
- Manual backup clean-up
## Next steps

Learn more about Periodic Backup and Restore concepts
[Configuring Periodic Backups & Automatic Restore for your Applications](service-fabric-backuprestoreservice-configure-periodic-backup.md)
* [Ad-Hoc Backup and Restore of a Partition](service-fabric-backuprestoreservice-adhoc-backuprestore.md)
* [Alternate site recovery](service-fabric-backuprestoreservice-alternate-site-recovery.md)
* [FAQs](service-fabric-backuprestoreservice-faqs.md)

[0]: ./media/service-fabric-backuprestoreservice/PartitionBackedUpHealthEvent.png

