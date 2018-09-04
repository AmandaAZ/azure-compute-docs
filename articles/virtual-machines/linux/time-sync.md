---
title: Time sync for Linux VMs in Azure | Microsoft Docs
description: Time sync for Linux virtual machines.
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager

ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 09/04/2018
ms.author: cynthn
---

# Time sync for Linux VMs in Azure

Time sync is important for security and event correlation. Sometimes it is used for distributed transactions implementation. Time accuracy between multiple computer systems is achieved through synchronization. Synchronization can be affected by multiple things, including reboots and network traffic between the time source and the computer fetching the time. 

Azure is backed by infrastructure running Windows Server 2016. Windows Server 2016 has improved algorithms used to correct time and condition the local clock to synchronize with UTC.  The Windows Server 2016 Accurate Time feature greatly improved how the VMICTimeSync service that governs Vms with the host for accurate time. Improvements include more accurate initial time on VM start or VM restore and interrupt latency correction. 

>[!NOTE]
>For a quick overview of Windows Time service, take a look at this [high-level overview video](https://aka.ms/WS2016TimeVideo).
>
> For more details about time sync in Windows Server 2016, see [Accurate time for Windows Server 2016](https://docs.microsoft.com/en-us/windows-server/networking/windows-time-service/accurate-time). 

## Overview

Accuracy for a computer clock is gauged on how close the computer clock is to the Coordinated Universal Time (UTC) time standard. UTC is defined by a multinational sample of very precise atomic clocks that can only be off by one second in 300 years. But, reading UTC directly requires specialized hardware. Instead, time servers are synced to UTC and are accessed from other computers to provide scalability and robustness. Every computer has time synchronization service running that knows what time servers to use and periodically checks if computer clock needs to be corrected and adjusts time if needed. 

In Azure, virtual machines can either depend on their host to synchronize with time.windows.com and pass the accurate time (*host time*) on to the VM or the VM can directly get time from a time server, or a combination of both. On stand alone hardware, the Linux OS only reads the host hardware clock on boot. After that, the clock is maintained using the interrupt timer in the Linux kernel. In this configuration, the clock can drift over time. In newer Linux distributions on Azure, VMs can use the VMICTimeSync provider, included in the Linux integration services (LIS), to query for clock updates from the host more frequently.

Virtual machine interactions with the host can also affect the clock. During [memory preserving maintenance](maintenance-and-updates.md#memory-preserving-maintenance), VMs are paused for up to 30 seconds. For example, before maintenance begins the VM clock shows 10:00:00 AM and lasts 28 seconds. After the VM resumes, the clock on the VM would still show 10:00:00 AM, which would be 28 seconds off. To correct for this, the VMICTimeSync service monitors what is happening on the host and prompts for changes to happen on the VMs to compensate.

Without time synchronization working, the clock on the VM would accumulate errors. When there is only one VM, the effect might not be that significant unless the workload requires highly accurate timekeeping. But in most cases, we have multiple, interconnected VMs that use time to track transactions and the time needs to be consistent throughout the entire deployment. When time between VMs is different, you could see the following affects:

- Authentication problems. Security protocols like Kerberos or certificate-dependent technology rely on time being consistent across the systems. Systems stop to trust each other if their time differs significantly. Login and access attempts can fail because of unsynchronized time.
- Distributed event correlation problems. It's very hard or practically impossible to figure out what have happened in a system if logs (or other data) don't agree on time. The same event would be shown to occur on different times making event correlation a new problem.
- There are number of distributed transactions algorithms that rely on precise time sync and they would not work if time drifts freely.
- Billing intrinsically depends on the time sync. If clock is off the billing cycles would be calculated incorrectly.
- Financial operations require precise time sync to resolve when each sell or buy did actually happen and consequently on what price.


## Configuration options

There are generally three ways to configure time sync for your Linux VMs hosted in Azure:

- The deafult congfiguration for Azure Marketplace images uses both NTP time and VMICTimeSync host-time. 
- Host-only using VMICTimeSync.
- Use another, external time server with or without using VMICTimeSync host-time.


### Use the default

By default, most Azure Marketplace images for Linux are are configured to sync from two sources: 

- NTP as primary, which gets time from an NTP server. For example, Ubuntu 16.04 LTS Marketplace images use **ntp.ubuntu.com**.
- The VMICTimeSync service as secondary, used to communicate the host time to the VMs and make corrections after the VM is paused for maintenance. Azure hosts use time.microsoft.com to keep accurate time.

In newer Linux distributions, the VMICTimeSync service uses the precision time protocol (PTP), but earlier distributions may not support PTP and will fall-back to NTP for getting time from the host.


### Host-only 

Because NTP servers like time.windows.com and ntp.ubuntu.com are public, they require sending traffic over the internet. Varying packet delays can negatively affect quality of the time sync. Removing NTP by switching to host-only sync can sometimes improve your time sync results.

Switching to host-only time sync makes sense if you experience time sync issues using the default configuration. Try out the host-only sync to see if that would improve the time sync on your VM. 

### External time server

If you have specific time sync requirements, there is also an option of using external time servers. Such time servers can provide specific time, for example for test scenarios or for time uniformity with machines hosted in on-premises or if you need to handle leap seconds in a special way.

You can use an external time server alone, or combine it with the VMICTimeSync service to provide results similar to the default configuration. Combining an external time server with VMICTimeSync is the best option for dealing with issues that can be cause when VMs are paused for maintenance. 

## Configuration

There are some basic commands for checking your time synchronization configuration. Documentation for Linux distribution will have more details on the best way to configure time synchronization for that distribution.

### Integration services

Check to see if the integations service (hv_utils) is loaded.

```bash
lsmod | grep hv_utils
```
You should see something similar to this:

```
hv_utils               24418  0
hv_vmbus              397185  7 hv_balloon,hyperv_keyboard,hv_netvsc,hid_hyperv,hv_utils,hyperv_fb,hv_storvsc
```

See if the Hyper-V integration services daemon is running.

```bash
ps -ef | grep hv
```

You should see sometihng similar to this:

```
root        229      2  0 17:52 ?        00:00:00 [hv_vmbus_con]
root        391      2  0 17:52 ?        00:00:00 [hv_balloon]
```


### Check for PTP

With newer versions of Linux, a Precision Time Protocol (PTP) clock source is available as part of the VMICTimeSync provider. On older versions of Red Hat Enterprise Linux or CentOS 7.x the Linux Integration Services can be downloaded and used to install the updated driver. When using PTP, the Linux device will be of the form /dev/ptp*x*. 

See which PTP clock sources are available.

```bash
ls /sys/class/ptp
```

In this example, the value returned is *ptp0*, so we use that to check the clock name. To verify the device is coming from the Azure host, check the clock name.

```bash
cat /sys/class/ptp/ptp0/clock_name
```

This should return **hyperv**.

### chrony

On Red Hat Enterprise Linux and CentOS 7.x, [chrony](https://chrony.tuxfamily.org/) can be used to configure the OS to use a PTP source clock. The Network Time Protocol daemon (ntpd) doesn’t support PTP sources, so using **chronyd** is recommended. To enable PTP from the host clock, update **chrony.conf**.

```bash
refclock PHC /dev/ptp0 poll 3 dpoll -2 offset 0
```

For more information on Red Hat and NTP, see [Configure NTP](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/s1-configure_ntp). 

For more information on chrony, see [Using chrony](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-using_chrony).

If both chrony and TimeSync sources are enabled simultaneously, you can mark one as **prefer** which sets the other source as a backup. Because NTP services do not update the clock for large skews except after a long period, the VMICTimeSync will recover the clock from paused VM events far more quickly than NTP-based tools alone.


### systemd 

On Ubuntu and SUSE time sync is configured using [systemd](https://www.freedesktop.org/wiki/Software/systemd/). For more information on Ubuntu, see [Time Synchronization](https://help.ubuntu.com/lts/serverguide/NTP.html). For more information on SUSE, see Section 4.5.8 in [SUSE Linux Enterprise Server 12 SP3 Release Notes](https://www.suse.com/releasenotes/x86_64/SUSE-SLES/12-SP3/#InfraPackArch.ArchIndependent.SystemsManagement).



## Next steps

For more details about time sync in Windows Server 2016, see [Accurate time for Windows Server 2016](https://docs.microsoft.com/en-us/windows-server/networking/windows-time-service/accurate-time).


