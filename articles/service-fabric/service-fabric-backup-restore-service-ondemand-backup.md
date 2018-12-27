---
title: On-demand backup in Azure Service Fabric | Microsoft Docs
description: Use Service Fabric's backup and restore feature to backup your application data on a need basis.
services: service-fabric
documentationcenter: .net
author: aagup
manager: timlt
editor: aagup

ms.assetid: 02DA262A-EEF6-4F90-842E-FFC4A09003E5
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/30/2018
ms.author: aagup

---
# On-demand backup in Azure Service Fabric

You can back up data of Reliable Stateful services and Reliable Actors to address disaster or data loss scenarios.

Service Fabric has features for the [periodic backup of data](service-fabric-backuprestoreservice-quickstart-azurecluster.md) and the backup of data on a need basis. On-demand backup is useful because it guards against _data loss_/_data corruption_ due to planned changes in the underlying service or its environment.

The on-demand backup features are helpful for capturing the state of the services before you manually trigger a service or service environment operation. For example, if you make a change in service binaries like upgrading or downgrading the service, on-demand backup can help guard the data against corruption by bugs in an application's code.

## Triggering on-demand backup

On-demand backup requires storage details for uploading backup files. You can specify the location for on-demand backup in the periodic backup policy or in an on-demand backup request.

### On-demand backup to storage specified by periodic backup policy

You can configure periodic backup policy to use a partition of a Reliable Stateful service or Reliable Actor for extra on-need backup to storage.

The following case is the continuation of the scenario in [Enabling periodic backup for Reliable Stateful service and Reliable Actors](service-fabric-backuprestoreservice-quickstart-azurecluster.md#enabling-periodic-backup-for-reliable-stateful-service-and-reliable-actors). In this case, you enable a backup policy to use a partition and a backup occurs at a desired frequency in Azure Storage.

You set the on-demand backup for partition ID `974bd92a-b395-4631-8a7f-53bd4ae9cf22` to be triggered by [BackupPartition](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-backuppartition) API.

```powershell
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/Backup?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
```

You set the [on-demand backup progress](service-fabric-backup-restore-service-ondemand-backup.md#tracking-on-demand-backup-progress) to be tracked by using the [GetBackupProgress](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-getpartitionbackupprogress) API.

### On-demand backup to specified storage

You can request on-demand backup for a partition of a Reliable Stateful service or Reliable Actor. You provide the storage information as a part of the on-demand backup request.

You set the on-demand backup for partition ID `974bd92a-b395-4631-8a7f-53bd4ae9cf22` to be triggered by using the [BackupPartition](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-backuppartition) API and including the following Azure Storage information:

```powershell
$StorageInfo = @{
    ConnectionString = 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net'
    ContainerName = 'backup-container'
    StorageKind = 'AzureBlobStore'
}

$OnDemandBackupRequest = @{
    BackupStorage = $StorageInfo
}

$body = (ConvertTo-Json $OnDemandBackupRequest)
$url = "https://mysfcluster.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/Backup?api-version=6.4"

Invoke-WebRequest -Uri $url -Method Post -Body $body -ContentType 'application/json' -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3'
```

You can set the [on-demand backup progress](service-fabric-backup-restore-service-ondemand-backup.md#tracking-on-demand-backup-progress) to be tracked by using the [GetBackupProgress](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-getpartitionbackupprogress) API.

## Tracking on-demand backup progress

A partition of a Reliable Stateful service or Reliable Actor accepts only one on-demand backup request at a time. Another request can be accepted only after the current on-demand backup request has completed.

Different partitions can trigger on-demand backup requests at a same time.

```powershell
$url = "https://mysfcluster-backup.southcentralus.cloudapp.azure.com:19080/Partitions/974bd92a-b395-4631-8a7f-53bd4ae9cf22/$/GetBackupProgress?api-version=6.4"

$response = Invoke-WebRequest -Uri $url -Method Get -CertificateThumbprint '1b7ebe2174649c45474a4819dafae956712c31d3' 
$backupResponse = (ConvertFrom-Json $response.Content) 
$backupResponse
```

On-demand backup requests can be in the following states:

* **Accepted** - The backup has been initiated on the partition and is in progress.
  ```
  BackupState             : Accepted
  TimeStampUtc            : 0001-01-01T00:00:00Z
  BackupId                : 00000000-0000-0000-0000-000000000000
  BackupLocation          :
  EpochOfLastBackupRecord :
  LsnOfLastBackupRecord   : 0
  FailureError            :
  ```
* **Success/ Failure/ Timeout** - A requested on-demand backup can be completed in any of the following states:
  * **Success** - The backup state as _Success_ indicates that the partition state is backed up successfully. The response will provide __BackupEpoch__ and __BackupLSN__ for the partition along with the time in UTC.
    ```
    BackupState             : Success
    TimeStampUtc            : 2018-11-21T20:00:01Z
    BackupId                : 5d64b697-6acd-45a4-adbd-3d75e0078081
    BackupLocation          : SampleApp\MyStatefulService\974bd92a-b395-4631-8a7f-53bd4ae9cf22\2018-11-21 20.00.01.zip
    EpochOfLastBackupRecord : @{DataLossNumber=131873018908156893; ConfigurationNumber=8589934592}
    LsnOfLastBackupRecord   : 36
    FailureError            :
    ```
  * **Failure** - The backup state as _Failure_ indicates that a failure occurred during backup of the partition's state. The cause of the failure will be stated in response.
    ```
    BackupState             : Failure
    TimeStampUtc            : 0001-01-01T00:00:00Z
    BackupId                : 00000000-0000-0000-0000-000000000000
    BackupLocation          :
    EpochOfLastBackupRecord :
    LsnOfLastBackupRecord   : 0
    FailureError            : @{Code=FABRIC_E_BACKUPCOPIER_UNEXPECTED_ERROR; Message=An error occurred during this operation.  Please check the trace logs for more details.}
    ```
  * **Timeout** - The backup state as _Timeout_ indicates that the partition state backup couldn't be created in a given time frame. The default timeout value is ten minutes. You should initiate a new on-demand backup request with greater [BackupTimeout](https://docs.microsoft.com/rest/api/servicefabric/sfclient-api-backuppartition#backuptimeout) in this scenario.
    ```
    BackupState             : Timeout
    TimeStampUtc            : 0001-01-01T00:00:00Z
    BackupId                : 00000000-0000-0000-0000-000000000000
    BackupLocation          :
    EpochOfLastBackupRecord :
    LsnOfLastBackupRecord   : 0
    FailureError            : @{Code=FABRIC_E_TIMEOUT; Message=The request of backup has timed out.}
    ```

## Next steps
- [Understand periodic backup configuration](./service-fabric-backuprestoreservice-configure-periodic-backup.md)
- [BackupRestore REST API reference](https://docs.microsoft.com/rest/api/servicefabric/sfclient-index-backuprestore)

