---
title: Manage costs effectively for SQL Server on Azure virtual machines | Microsoft Docs
description: Provides best practices for choosing the right SQL Server virtual machine pricing model.
services: virtual-machines-windows
documentationcenter: na
author: rothja
manager: jhubbard
editor: ''
tags: azure-service-management

ms.assetid: 
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 10/17/2017
ms.author: jroth
---
# Pricing guidance for SQL Server Azure VMs

This article provides pricing guidance for SQL Server virtual machines in Azure. There are several options that affect cost, and it is important to pick the right image that balances costs with business requirements.

## Free-licensed SQL Server editions

If you want to develop, test, or build a proof of concept, then use the free-licensed **SQL Server Developer edition**. This edition has everything in SQL Server Enterprise edition, thus you can use it to build whatever application you want. It’s only not allowed to run in production. A SQL Server Developer VM only charges for the cost of the VM,not for SQL Server licensing.

If you want to run a lightweight workload in production (<4 cores, <1 GB memory, <10 GB/database), then use the free-licensed **SQL Server Express edition**. A SQL Express VM only charges for the cost of the VM, not SQL licensing.

For these development/test or lightweight production workloads, you can also save money by choosing a smaller VM size that matches these workloads. The DS1v2 might be a good choice for these workloads.

To create a SQL Server 2016 Azure VM with one of these images, see the following links:

- [SQL Server 2016 Developer Azure VM](https://ms.portal.azure.com/#create/Microsoft.FreeLicenseSQLServer2016SP1DeveloperWindowsServer2016-ARM)
- [SQL Server 2016 Express Azure VM](https://ms.portal.azure.com/#create/Microsoft.FreeLicenseSQLServer2016SP1ExpressWindowsServer2016-ARM)

## Paid SQL Server editions

If you have a non-lightweight production workload, use one of the following SQL Server editions:

| SQL Server Edition | Workload |
|-----|-----|
| Web | Small web sites |
| Standard | Small to medium workloads |
| Enterprise | Large or mission-critical workloads|

You have two options to pay for SQL Server licensing for these editions: *pay per usage* or *bring your own license (BYOL)*.

### Pay per usage

**Paying the SQL Server license per usage** means that the per-minute cost of running the Azure VM includes the cost of the SQL Server license. You can see the pricing for the different SQL Server editions (Web, Standard, Enterprise) in the [Azure VM pricing page](https://azure.microsoft.com/pricing/details/virtual-machines/sql-server-standard). The cost is the same for all versions of SQL Server (2008 R2 to 2016). As with SQL Server licensing in general, the per-minute licensing cost depends on the number of VM cores.

Paying the SQL Server licensing per usage is recommended for:

- Temporary or periodic workloads. For example, an app that needs to support an event for a couple of months every year, or business analysis on Mondays.
- Workloads with unknown lifetime or scale. For example, an app that may not be required in a few months, or which may require more, or less compute power, depending on demand.

To create a SQL Server 2016 Azure VM with one of these pay-per-usage images, see the following links:

- [SQL Server 2016 Web Azure VM](https://ms.portal.azure.com/#create/Microsoft.SQLServer2016SP1WebWindowsServer2016)
- [SQL Server 2016 Standard Azure VM](https://ms.portal.azure.com/#create/Microsoft.SQLServer2016SP1StandardWindowsServer2016)
- [SQL Server 2016 Enterprise Azure VM](https://ms.portal.azure.com/#create/Microsoft.SQLServer2016SP1EnterpriseWindowsServer2016)

> [!IMPORTANT]
> When you create a SQL Server virtual machine in the Azure portal, the estimated monthly cost displayed on the **Choose a size** blade does not include SQL Server licensing costs. This is the cost of the VM alone.
>
> ![Choose VM size blade](./media/virtual-machines-windows-sql-server-pricing-guidance/sql-vm-choose-size-pricing-estimate.png)
>
>For the free-licensed Express and Developer editions of SQL Server, this is the total estimated cost. But for Web, Standard, and Enterprise, find the additional SQL licensing costs on the Azure pricing pages for [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) and [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/). On the pricing page, select your target edition of SQL Server.

### Bring your own license (BYOL)

**Bringing your own SQL Server license through License Mobility**, also referred to as **BYOL**, means using an existing SQL Server Volume License with Software Assurance in an Azure VM. A SQL Server VM using BYOL only charges for the cost of running the VM, not for SQL Server licensing, given that you have already acquired licenses and Software Assurance through a Volume Licensing program.

> [!NOTE]
> The BYOL images are currently only available for Windows virtual machines. Pay per usage VMs are available for both Windows and Linux.

Bringing your own SQL licensing through License Mobility is recommended for:

- Continuous workloads. For example, an app that needs to support business operations 24x7.
- Workloads with known lifetime and scale. For example, an app that is required for the whole year and which demand has been forecasted.

To use BYOL with a SQL Server VM, you must have a license for SQL Server Standard or Enterprise and [Software Assurance](https://www.microsoft.com/en-us/licensing/licensing-programs/software-assurance-default.aspx#tab=1), which is a required option through some [Volume Licensing](https://www.microsoft.com/en-us/download/details.aspx?id=10585) programs and an optional purchase with others.  The pricing level provided through Volume Licensing programs varies, based on the type of agreement and the quantity and or commitment to SQL Server. But as a rule of thumb, bringing your own license for continuous production workloads has the following benefits:

| BYOL benefit | Description |
|-----|-----|
| **Cost savings** | Bringing your own SQL Server license is more cost effective than paying it per usage if a workload runs continuously SQL Server Standard or Enterprise for *more than 10 months*. |
| **Long-term savings** | On average, it is *30% cheaper per year* to buy or renew a SQL Server license for the first 3 years. Furthermore, after 3 years, you don’t need to renew the license anymore, just pay for Software Assurance. At that point, it is *200% cheaper*. |
| **Free passive secondary replica** | Another benefit of bringing your own license is the [free licensing for one passive secondary replica](https://azure.microsoft.com/pricing/licensing-faq/) per SQL Server for high availability purposes. This cuts in half the licensing cost of a highly available SQL Server deployment (for example, using Always On Availability Groups). The rights to run the passive secondary are provided through the Fail-Over Servers Software Assurance benefit. |

To create a SQL Server 2016 Azure VM with one of these bring-your-own-license images, see the VMs prefixed with "{BYOL}":

- [SQL Server 2016 Enterprise Azure VM](https://ms.portal.azure.com/#create/Microsoft.BYOLSQLServer2016SP1EnterpriseWindowsServer2016)
- [SQL Server 2016 Standard Azure VM](https://ms.portal.azure.com/#create/Microsoft.BYOLSQLServer2016SP1StandardWindowsServer2016)

> [!NOTE]
> Let us know within 10 days how many SQL Server licenses you’ll use in Azure. The links to the previous images have instructions on how to do this.

## Avoid unnecessary costs

To avoid unnecessary costs, choose an optimal virtual machine size and consider intermittent shutdowns for non-continuous workloads.

### <a id="machinesize"></a> Correctly size your VM

The licensing cost of SQL Server is directly related to the number of cores. Choose a VM size that matches your expected needs for CPU, memory, storage, and I/O bandwidth. For a complete list of machine size options, see [Windows VM sizes](https://docs.microsoft.com/azure/virtual-machines/windows/sizes) and [Linux VM sizes](https://docs.microsoft.com/azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

There are new machine sizes that work well with certain types of SQL Server workloads. These machines sizes maintain high levels of memory, storage, and I/O bandwidth, but they have a lower virtualized core count. For example, consider the following example:

| VM Size | vCPUs | Memory | Max Disks | Max I/O Throughput | SQL licensing costs | Total costs (Compute + licensing) |
|---|---|---|---|---|---|---|
| **Standard_DS14v2** | 16 | 112 GB | 32 | 51,200 IOPS or 768 MB/s | | |
| **Standard_DS14-4v2** | 4 | 112 GB | 32 | 51,200 IOPS or 768 MB/s | 75% lower | 57% lower |

> [!IMPORTANT]
> This is a point-in-time example. For the most recent specifications, refer to the machine sizes articles and the Azure pricing page for [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) and [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/).

In the previous example, you can see that the specifications for **Standard_DS14v2** and **Standard_DS14-4v2** are identical except for vCPUs. The suffix **-4v2** at the end of the **Standard_DS14-4v2** machine size indicates the number of active vCPUs. Because SQL Server licensing costs are tied to the number of cores, this significantly reduces the cost of the VM in scenarios where the extra vCPUs are not needed. This is one example, and there are many machine sizes with constrained vCPUs that are identified with this suffix pattern. For more information, see the blog post [Announcing new Azure VM sizes for more cost-effective database work](https://azure.microsoft.com/blog/announcing-new-azure-vm-sizes-for-more-cost-effective-database-workloads/).

### Shut down your VM when possible

If you are using any workloads that do not run continuously, consider shutting down the virtual machine during the inactive periods. You only pay for what you use.

For example, if you are simply trying out SQL Server on an Azure VM, you would not want to incur charges by accidentally leaving it running for weeks. One solution is to use the [automatic shutdown feature](https://azure.microsoft.com/blog/announcing-auto-shutdown-for-vms-using-azure-resource-manager/).

![SQL VM autoshutdown](./media/virtual-machines-windows-sql-server-pricing-guidance/sql-vm-auto-shutdown.png)

Automatic shutdown is part of a larger set of similar features provided by [Azure DevTest Labs](https://azure.microsoft.com/services/devtest-lab).

For other workflows, consider automatically shutting down and restarting Azure VMs with a scripting solution, such as [Azure Automation](https://azure.microsoft.com/services/automation/).

> [!IMPORTANT]
> Shutting down and deallocating your VM is the only way to avoid charges. Simply stopping or using power options to shut down the VM still incurs usage charges.

## Next steps

For general Azure pricing guidance, see [Prevent unexpected costs with Azure billing and cost management](../../../billing/billing-getting-started.md).

For the latest Virtual Machines pricing, including SQL Server, see the [Azure VM pricing page](https://azure.microsoft.com/pricing/details/virtual-machines/sql-server-standard).

Learn more about SQL Server virtual machines for both [SQL Server Windows VMs](virtual-machines-windows-sql-server-iaas-overview.md) and [SQL Server Linux VMs](../../linux/sql/sql-server-linux-virtual-machines-overview.md).
