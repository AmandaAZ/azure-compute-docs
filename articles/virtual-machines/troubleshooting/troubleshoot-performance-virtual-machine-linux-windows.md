---
title: Generic performance troubleshooting for Azure Virtual Machine running Linux or Windows| Microsoft Docs
description: This article describes virtual machine (VM) generic performance troubleshooting through monitoring and observing bottlenecks and provides possible remediation for issues that may occur.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: top-support-issue, azure-resource-manager

ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows

ms.topic: troubleshooting
ms.date: 09/18/2019
ms.author: v-miegge
---

# Generic performance troubleshooting for Azure Virtual Machine running Linux or Windows

This article describes virtual machine (VM) generic performance troubleshooting through monitoring and observing bottlenecks and provides possible remediation for issues that may occur.

## Enabling Monitoring

### Azure IAAS VM Monitoring

To monitor the Guest VM, use the Azure VM Monitoring, which will alert you to certain high-level resource conditions. To check whether you have the VM diagnostics enabled, click [here](https://azure.microsoft.com/documentation/articles/insights-how-to-use-diagnostics/#change-settings-for-an-existing-resource). If you see the following, then you most likely don't have the diagnostics enabled:

![Monitoring is not enabled](media/troubleshoot-performance-virtual-machine-linux-windows/1-virtual-machines-monitoring-not-enabled.png)
 
### Enable VM Diagnostics through Portal

To enable VM diagnostics, go to the VM, click **Settings**, and then click **Diagnostics**.

![Click Settings, then Diagnostics](media/troubleshoot-performance-virtual-machine-linux-windows/2-virtual-machines-diagnostics.png)
 
### Enable Storage Account Diagnostics through Portal

First, identify which storage account (or accounts) your VM is using by selecting the VM. Click **Settings**, and then click **Disks**:

![Click Settings, then Disks](media/troubleshoot-performance-virtual-machine-linux-windows/3-storage-disks-disks-selection.png)

In the portal, go to the storage account (or accounts) for the VM and work through the following steps:

![Select Blob Metrics](media/troubleshoot-performance-virtual-machine-linux-windows/4-select-blob-metrics.png)
 
1. Select **All Settings**.
2. Turn Diagnostics On.
3. Select **Blob* Metrics** and set retention to **30** days.
4. Save the changes.

## Observing Bottlenecks

### Accessing the Monitoring

Select the Azure VM you want to investigate and select **Monitoring**.

![Select Monitoring](media/troubleshoot-performance-virtual-machine-linux-windows/5-observe-monitoring.png)
 
### Timelines of Observation

To identify if you have any resource bottlenecks, review your data. If your find that your machine has been running fine, but it has been reported that the performance has recently degraded, review a time range of data that encompasses performance metric data before the reported changed, during and after the issue.

### Check for CPU Bottleneck

![Check for CPU Bottleneck](media/troubleshoot-performance-virtual-machine-linux-windows/6-cpu-bottleneck-time-range.png)

1. Edit the Graph.
2. Set the time Range.
3. You then need to add in the counter: CPU Percentage Guest OS
4. Save.

### CPU Observe Trends

When looking at performance issues, be aware of the trends and understand if they affect you. In the next sections, we will use the Monitoring graphs from the portal to show trends. They can also be useful for cross referencing difference resource behaviors in the same time period. This article discusses how to customize the [graphs](https://azure.microsoft.com/documentation/articles/insights-how-to-customize-monitoring/).

Spiking – Spiking could be related to a scheduled task/known event. If you know what this task is, determine whether it runs at the required performance level. If performance is acceptable, you may not need to increase resources.

Spike up and Constant – This issue could indicate a new workload. If it is not a recognized workload, enable monitoring in the VM to find out what process (or processes) cause this behavior. Once the process is recognized, determine whether the increased consumption is being caused by inefficient code, or normal consumption. If normal consumption, decide wether the process operates at the required performance level.

Constant – Has your VM always run at this level, or only since the diagnostics have been enabled? If so, identify the process (or processes) causing this issue, and consider adding more of that resource.

Steadily Increasing – If you see a constant increase in consumption, this could be either inefficient code or a process taking on more user workload.

### High CPU Utilization Remediation

If your application or process is not running at the correct performance level, and you are seeing 95% + CPU utilization constant, you can perform either of the following tasks:

* For immediate relief - Increase the size of the VM to a size with more cores
* Understand the issue – locate application/process and troubleshoot accordingly.

If you have increased the VM, and the CPU is still running 95%, determine whether this is offering better performance or higher application throughput to an acceptable level. If not, troubleshoot that individual application\process.

## Check for Memory Bottleneck

To view the metrics:

1. Add a Section.
2. Add a Tile.
3. Open the Gallery.
4. Select the Memory Usage and drag. When the tile is docked, right-click and select **6x4**.

### Memory Observe Trends

The Memory Usage shows you how much memory is being consumed with the VM. Understand the trend and whether it maps to the time where you see issues. You should always have more than 100 MB of available memory.

Spike and Constant/Constant Steady Consumption - High memory utilization may not be the cause of bad performance, as some applications such as relational database engines allocate a large amount of memory, and this utilization may not be significant. However, if there are multiple memory-hungry applications, you may see poor performance from memory contention causing trimming and paging/swapping to disk. This poor performance is often a noticeable cause of application performance impact.

Steadily Increasing Consumption – A possible application ‘warming up’, this consumption is common among database engines starting up. However, it could also be a sign of a memory leak in an application. Identify the application and understand whether the behavior is expected.

Page or Swap File Usage – Check whether you are using the Windows Paging file (located on D:\) or Linux Swap file (located on `/dev/sdb`) are being heavily used. If you have nothing on these volumes except these files, check for high Read/Writes on those disks. This issue is indicative of low memory conditions.

### High Memory Utilization Remediation

To resolve high memory utilization, perform any of the following tasks:

* For immediate relief or Page or Swap File Usage - Increase the VM size to one with more memory, then monitor.
* Understand issue – locate applications/process and troubleshoot for identifying high consuming memory applications.
* If you know the application, see if the memory allocation can be capped.

If after upgrading to a larger VM, you discover that you still have a constant steady increase until 100%, identify the application/process and troubleshoot.

## Check for Disk Bottleneck

To check the storage subsystem for the VM, check the diagnostics at the Azure VM level by using the counters in VM Diagnostics and also the Storage Account Diagnostics.

Note that we do not have counters for Zone Redundant and Premium Storage Accounts. For issues related to these counters, raise a support case.

### Viewing Storage Account Diagnostics in Monitoring

To work on the below items, go into the storage account for the VM in the portal:

![Viewing Storage Account Diagnostics in Monitoring](media/troubleshoot-performance-virtual-machine-linux-windows/7-virtual-machine-storage-account.png)

1. Edit the Monitoring Graph.
2. Set the time range.
3. Add the counters described in the steps below.
4. Save the changes.

### Disk Observe Trends (Standard Storage Only)

To identify issues with storage, look at the performance metrics from the Storage Account Diagnostics and the VM Diagnostics.

For each check below, look for key trends when the issues occur within the time range of the issue.

#### Check Azure Storage Availability – Add the Storage Account Metric: Availability

If you see a drop in availability, there could be an issue with the platform, check the [Azure Status](https://azure.microsoft.com/status/). If no issue is shown there, raise a new support request.

#### Check for Azure Storage Timeout - Add the Storage Account Metrics:

* ClientTimeOutError
* ServerTimeOutError
* AverageE2ELatency
* AverageServerLatency
* TotalRequests

If you see values in the *TimeOutError metrics, an IO operation took too long and timed out. Working through the next steps will help identify potential causes.

If you see AverageServerLatency increase at the same time at the TimeOutErrors, it could be a platform issue. Raise a new support request in this situation.

If you see AverageE2ELatency, this represents client latency then verify how the IOPS are being performed by the application. Look for an increase or constantly high TotalRequests metric. This metric represents IOPS. If you are starting to hit the limits of the storage account or single VHD, the latency could be related to throttling.

#### Check for Azure Storage Throttling - Add the Storage Account Metrics: ThrottlingError

Values for throttling indicates that you are being throttled at storage account level, meaning that your hitting the IOPS limit of the account. You can determine whether you are hitting the IOPs threshold by checking the metric **TotalRequests**.

Note that each VHD has a limit of 500 IOPS or 60 MBits, but is bound by the cumulative limit of 20000 IOPS per storage account.

With this metric, you cannot tell which blob is causing the throttling and which are affected by it. However, you are either hitting the IOPS or Ingress/Egress limits of the storage account.

To identify if you are hitting the IOPS limit, go into the Storage Account diagnostics and check the TotalRequests, looking to see if you are approaching 20 thousand TotalRequests. Identify whether this is a change in the pattern, whether this the first time it is being seen, or whether this happens at a certain time.

#### References

* [Scalability targets for virtual machine disks](https://azure.microsoft.com/documentation/articles/storage-scalability-targets/#scalability-targets-for-virtual-machine-disks)

The bandwidth of the storage account is measured by the Storage Account Metrics: TotalIngress and TotalEgress. You have different thresholds for bandwidth depending on type of redundancy and regions.

* [Scalability targets for blobs, queues, tables, and files](https://azure.microsoft.com/documentation/articles/storage-scalability-targets/#scalability-targets-for-blobs-queues-tables-and-files)

Check the TotalIngress and TotalEgress against the Ingress and Egress limits for the storage account redundancy type and region.

Check Throughput Limits of the VHDs attached to the VM. Add the VM Metrics Disk Read and Write.

Each VHD can support up to 60 MB/s (IOPS are not exposed per VHD). Look at the data to see if you are the hitting the limits of combined throughput MB of the VHD(s) at VM level using Disk Read and Write, then optimize your VM storage configuration to scale past single VHD limits.

### High Disk Utilization/Latency Remediation

Reduce Client Latency and Optimize VM IO to scale past VHD Limits

* [Optimizing IO for Windows in Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-sql-server-performance-best-practices/)

* [Optimizing IO for Linux in Azure](https://blogs.msdn.microsoft.com/igorpag/2014/10/23/azure-storage-secrets-and-linux-io-optimizations/)

#### Reduce Throttling

If hitting upper limits of storage accounts, re-balance the VHDs between storage accounts. Refer to [Azure Storage Scalability and Performance Targets](https://azure.microsoft.com/documentation/articles/storage-scalability-targets/).

### Increase Throughput and Reduce Latency

If you have a latency sensitive application and require high throughput, migrate your VHDs to Azure Premium storage using the DS and GS series VM.

These articles discuss the specific scenarios:

* [Migrating to Azure Premium Storage](https://azure.microsoft.com/documentation/articles/storage-migration-to-premium-storage/)

* [Use Azure Premium Storage with SQL Server](https://azure.microsoft.com/documentation/articles/virtual-machines-sql-server-use-premium-storage/)
