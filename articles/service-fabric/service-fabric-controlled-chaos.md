---
title: Induce Chaos in Service Fabric clusters | Microsoft Docs
description: Using Fault Injection and Cluster Analysis Service APIs to manage Chaos in the cluster.
services: service-fabric
documentationcenter: .net
author: motanv
manager: anmola
editor: motanv

ms.assetid: 2bd13443-3478-4382-9a5a-1f6c6b32bfc9
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/06/2017
ms.author: motanv

---
# Induce controlled Chaos in Service Fabric clusters
Large-scale distributed systems like cloud infrastructures are inherently unreliable. Azure Service Fabric enables developers to write reliable distributed services on top of an unreliable infrastructure. To write robust distributed services on top of an unreliable infrastructure, developers need to be able to test the stability of their services while the underlying unreliable infrastructure is going through complicated state transitions due to faults.

The [Fault Injection and Cluster Analysis Service](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-testability-overview) (also known as the Fault Analysis Service) gives developers the ability to induce faults to test their services. These targeted simulated faults, like [restarting a partition](https://docs.microsoft.com/en-us/powershell/module/servicefabric/start-servicefabricpartitionrestart?view=azureservicefabricps), can help exercise the most common state transitions. However targeted simulated faults are biased by definition and thus may miss bugs that show up only in hard-to-predict, long and complicated sequence of state transitions. For an unbiased testing, you can use Chaos.

Chaos simulates periodic, interleaved faults (both graceful and ungraceful) throughout the cluster over extended periods of time. Once you have configured Chaos with the rate and the kind of faults, you can start Chaos through C#, Powershell, or REST API to start generating faults in the cluster and in your services. You can configure Chaos to run for a specified time period (for example, for one hour), after which Chaos stops automatically, or you can call StopChaos API (C#, Powershell, or REST) to stop it at any time.

> [!NOTE]
> In its current form, Chaos induces only safe faults, which implies that in the absence of external faults a quorum loss, or data loss never occurs.
>

While Chaos is running, it produces different events that capture the state of the run at the moment. For example, an ExecutingFaultsEvent contains all the faults that Chaos has decided to execute in that iteration. A ValidationFailedEvent contains the details of a validation failure (health or stability issues) that was found during the validation of the cluster. You can invoke the GetChaosReport API (C#, Powershell, or REST) to get the report of Chaos runs. These events get persisted in a [reliable dictionary](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-reliable-services-reliable-collections), which has a truncation policy dictated by two configurations: **MaxStoredChaosEventCount** (default value is 25000) and **StoredActionCleanupIntervalInSeconds** (default value is 3600). Every *StoredActionCleanupIntervalInSeconds* Chaos checks and all but the most recent *MaxStoredChaosEventCount* events, are purged from the reliable dictionary.

## Faults induced in Chaos
Chaos generates faults across the entire Service Fabric cluster and compresses faults that are seen in months or years into a few hours. The combination of interleaved faults with the high fault rate finds corner cases that may otherwise be missed. This exercise of Chaos leads to a significant improvement in the code quality of the service.

Chaos induces faults from the following categories:

* Restart a node
* Restart a deployed code package
* Remove a replica
* Restart a replica
* Move a primary replica (configurable)
* Move a secondary replica (configurable)

Chaos runs in multiple iterations. Each iteration consists of faults and cluster validation for the specified period. You can configure the time spent for the cluster to stabilize and for validation to succeed. If a failure is found in cluster validation, Chaos generates and persists a ValidationFailedEvent with the UTC timestamp and the failure details. For example, consider an instance of Chaos that is set to run for an hour with a maximum of three concurrent faults. Chaos induces three faults, and then validates the cluster health. It iterates through the previous step until it is explicitly stopped through the StopChaosAsync API or one-hour passes. If the cluster becomes unhealthy in any iteration (that is, it does not stabilize or it does not become healthy within the passed-in MaxClusterStabilizationTimeout), Chaos generates a ValidationFailedEvent. This event indicates that something has gone wrong and might need further investigation.

To get which faults Chaos induced, you can use GetChaosReport API (powershell, C#, or REST). The API gets the next segment of the Chaos report based on the passed-in continuation token or the passed-in time-range. You can either specify the ContinuationToken to get the next segment of the Chaos report or you can specify the time-range through StartTimeUtc and EndTimeUtc, but you cannot specify both the ContinuationToken and the time-range in the same call. When there are more than 100 Chaos events, the Chaos report is returned in segments where a segment contains no more than 100 Chaos events.

## Important configuration options
* **TimeToRun**: Total time that Chaos runs before it finishes with success. You can stop Chaos before it has run for the TimeToRun period through the StopChaos API.

* **MaxClusterStabilizationTimeout**: The maximum amount of time to wait for the cluster to become healthy before producing a ValidationFailedEvent. This wait is to reduce the load on the cluster while it is recovering. The checks performed are:
  * If the cluster health is OK
  * If the service health is OK
  * If the target replica set size is achieved for the service partition
  * That no InBuild replicas exist
* **MaxConcurrentFaults**: The maximum number of concurrent faults that are induced in each iteration. The higher the number, the more aggressive Chaos is and the failovers and the state transition combinations that the cluster goes through are also more complex. 

> [!NOTE]
> Regardless how high a value *MaxConcurrentFaults* has, Chaos guarantees - in the absence of external faults - there is no quorum loss or data loss.
>

* **EnableMoveReplicaFaults**: Enables or disables the faults that cause the primary or secondary replicas to move. These faults are disabled by default.
* **WaitTimeBetweenIterations**: The amount of time to wait between iterations. That is, the amount of time Chaos will pause after having executed a round of faults and having finished the corresponding validation of the health of the cluster. The higher the value, the lower is the average fault injection rate.
* **WaitTimeBetweenFaults**: The amount of time to wait between two consecutive faults in a single iteration. The higher the value, the lower the concurrency of (or the overlap between) faults.
* **ClusterHealthPolicy**: Cluster health policy is used to validate the health of the cluster in between Chaos iterations. If the cluster health is in error or if an unexpected exception happens during fault execution, Chaos will wait for 30 minutes before the next health-check - to provide the cluster with some time to recuperate.
* **Context**: A collection of (string, string) type key-value pairs. The map can be used to record information about the Chaos run. There cannot be more than 100 such pairs and each string (key or value) can be at most 4095 characters long. This map is set by the starter of the Chaos run to optionally store the context about the specific run.
* **ChaosTargetFilter**: This filter can be used to target Chaos faults only to certain node types or only to certain application instances. If ChaosTargetFilter is not used, Chaos faults all cluster entities. If ChaosTargetFilter is used, Chaos faults only the entities that meet the ChaosTargetFilter specification. NodeTypeInclusionList and ApplicationInclusionList allow union semantics only. In other words, it is not possible to specify an intersection of NodeTypeInclusionList and ApplicationInclusionList. For example, it is not possible to specify "fault this application only when it is on that node type." Once an entity is included in either NodeTypeInclusionList or ApplicationInclusionList, that entity cannot be excluded using ChaosTargetFilter. Even if applicationX does not appear in ApplicationInclusionList, in some Chaos iteration applicationX can be faulted because it happens to be on a node of nodeTypeY that is included in NodeTypeInclusionList. If both NodeTypeInclusionList and ApplicationInclusionList are null or empty, an ArgumentException is thrown.
    * **NodeTypeInclusionList**: 
    A list of node types to include in Chaos faults. All types of faults (restart node, restart codepackage, remove replica, restart replica, move primary, and move secondary) are enabled for the nodes of these node types. If a nodetype (say NodeTypeX) does not appear in the NodeTypeInclusionList, then node level faults (like NodeRestart) will never be enabled for the nodes of NodeTypeX, but code package and replica faults can still be enabled for NodeTypeX if an application in the ApplicationInclusionList happens to reside on a node of NodeTypeX. At most 100 node type names can be included in this list, to increase this number, a config upgrade is required for MaxNumberOfNodeTypesInChaosEntityFilter configuration.
    * **ApplicationInclusionList**:
    A list of application URIs to include in Chaos faults. All replicas belonging to services of these applications are amenable to replica faults (restart replica, remove replica, move primary, and move secondary) by Chaos. Chaos may restart a code package only if the code package hosts replicas of these applications only. If an application does not appear in this list, it can still be faulted in some Chaos iteration if the application ends up on a node of a node type that is incuded in NodeTypeInclusionList. However if applicationX is tied to nodeTypeY through placement constraints and applicationX is absent from ApplicationInclusionList and nodeTypeY is absent from NodeTypeInclusionList, then applicationX will never be faulted. At most 1000 application names can be included in this list, to increase this number, a config upgrade is required for MaxNumberOfApplicationsInChaosEntityFilter configuration.

## How to run Chaos

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Fabric;

using System.Diagnostics;
using System.Fabric.Chaos.DataStructures;

class Program
{
    private class ChaosEventComparer : IEqualityComparer<ChaosEvent>
    {
        public bool Equals(ChaosEvent x, ChaosEvent y)
        {
            return x.TimeStampUtc.Equals(y.TimeStampUtc);
        }

        public int GetHashCode(ChaosEvent obj)
        {
            return obj.TimeStampUtc.GetHashCode();
        }
    }

    static void Main(string[] args)
    {
        var clusterConnectionString = "localhost:19000";
        using (var client = new FabricClient(clusterConnectionString))
        {
            var startTimeUtc = DateTime.UtcNow;
            var stabilizationTimeout = TimeSpan.FromSeconds(30.0);
            var timeToRun = TimeSpan.FromMinutes(60.0);
            var maxConcurrentFaults = 3;

            var parameters = new ChaosParameters(
                stabilizationTimeout,
                maxConcurrentFaults,
                true, /* EnableMoveReplicaFault */
                timeToRun);

            try
            {
                client.TestManager.StartChaosAsync(parameters).GetAwaiter().GetResult();
            }
            catch (FabricChaosAlreadyRunningException)
            {
                Console.WriteLine("An instance of Chaos is already running in the cluster.");
            }

            var filter = new ChaosReportFilter(startTimeUtc, DateTime.MaxValue);

            var eventSet = new HashSet<ChaosEvent>(new ChaosEventComparer());

            while (true)
            {
                var report = client.TestManager.GetChaosReportAsync(filter).GetAwaiter().GetResult();

                foreach (var chaosEvent in report.History)
                {
                    if (eventSet.Add(chaosEvent))
                    {
                        Console.WriteLine(chaosEvent);
                    }
                }

                // When Chaos stops, a StoppedEvent is created.
                // If a StoppedEvent is found, exit the loop.
                var lastEvent = report.History.LastOrDefault();

                if (lastEvent is StoppedEvent)
                {
                    break;
                }

                Task.Delay(TimeSpan.FromSeconds(1.0)).GetAwaiter().GetResult();
            }
        }
    }
}
```

```powershell
$connection = "localhost:19000"
$timeToRun = 60
$maxStabilizationTimeSecs = 180
$concurrentFaults = 3
$waitTimeBetweenIterationsSec = 60

Connect-ServiceFabricCluster $connection

$events = @{}
$now = [System.DateTime]::UtcNow

Start-ServiceFabricChaos -TimeToRunMinute $timeToRun -MaxConcurrentFaults $concurrentFaults -MaxClusterStabilizationTimeoutSec $maxStabilizationTimeSecs -EnableMoveReplicaFaults -WaitTimeBetweenIterationsSec $waitTimeBetweenIterationsSec

while($true)
{
    $stopped = $false
    $report = Get-ServiceFabricChaosReport -StartTimeUtc $now -EndTimeUtc ([System.DateTime]::MaxValue)

    foreach ($e in $report.History) {

        if(-Not ($events.Contains($e.TimeStampUtc.Ticks)))
        {
            $events.Add($e.TimeStampUtc.Ticks, $e)
            if($e -is [System.Fabric.Chaos.DataStructures.ValidationFailedEvent])
            {
                Write-Host -BackgroundColor White -ForegroundColor Red $e
            }
            else
            {
                if($e -is [System.Fabric.Chaos.DataStructures.StoppedEvent])
                {
                    $stopped = $true
                }

                Write-Host $e
            }
        }
    }

    if($stopped -eq $true)
    {
        break
    }

    Start-Sleep -Seconds 1
}

Stop-ServiceFabricChaos
```
