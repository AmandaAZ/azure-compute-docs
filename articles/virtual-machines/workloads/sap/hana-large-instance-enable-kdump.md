---
title: Script to enable Kdump in SAP HANA (Large Instances)| Microsoft Docs
description: Script to enable Kdump in SAP HANA (Large Instances) HLI Type I, HLI Type II
services: virtual-machines-linux
documentationcenter:
author: prtyag
manager: hrushib
editor:

ms.service: virtual-machines-linux

ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/30/2020
ms.author: prtyag
ms.custom: H1Hack27Feb2017

---

# Enable Kdump service
This document describes the details on how to enable Kdump service on  HANA Large Instances(**Type I and Type II**)

## Supported SKU
|  Hana Large Instance Type   |  OS Vendor   |  OS Package Version   |  SKU	       |
|-----------------------------|--------------|-----------------------|-------------|
|   Type 2                    |  SuSE        |   SLES 12 SP3         |  S384       |
|   Type 2                    |  SuSE        |   SLES 12 SP3         |  S576m      |
|   Type 1                    |  SuSE        |   SLES 12 SP3         |  S224m      |
|   Type 1                    |  SuSE        |   SLES 12 SP4         |  S224m      |
|   Type 1                    |  SuSE        |   SLES 12 SP3         |  S96        |
|   Type 1                    |  SuSE        |   SLES 12 SP4         |  S96        |

## Pre-requisites
- Kdump service uses /var directory to write logs, make sure the partition correspond to this directory has sufficient
space to accommodate logs.

## Setup details
- Script to enable Kdump can be found [here](https://github.com/Azure/sap-hana/blob/master/tools/enable-kdump.sh)
- Run this script on HANA Large Instance using the below command
```
bash enable-kdump.sh
```
- Reboot the system to apply changes.

## Test Kdump
- Trigger a kernel crash
```
echo 1 > /proc/sys/kernel/sysrq
echo c > /proc/sysrq-trigger
```
- After the system reboots successfully, check the `/var/crash directory` for kernel crash logs.
- If the `/var/crash` has directory with current date, then the Kdump is successfully enabled.

### Support issue
In case, if the script fails with some error or Kdump is not enabled, Please raise a ticket with Microsoft Op erations team by opening a support ticket.

Include in your ticket:

* Your HLI subscription ID.
* Your server name.
* OS name
* OS version