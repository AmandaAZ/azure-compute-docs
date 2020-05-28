---
title: Edv4 and Edsv4-series - Azure Virtual Machines
description: Specifications for the Ev4, Edv4, Esv4 and Edsv4-series VMs.
author: brbell
ms.author: brbell
ms.reviewer: cynthn
ms.custom: mimckitt
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 02/04/2020
---

# Edv4 and Edsv4-series

The Edv4 and Edsv4-series runs on the Intel&reg; Xeon&reg; Platinum 8272CL (Cascade Lake) processors in a hyper-threaded configuration, and are ideal for various memory-intensive enterprise applications and feature up to 504 GiB of RAM.

> [!NOTE]
> We are aware of an issue in the Azure host software that currently causes RHEL 7.x and CentOS 7.x Linux distributions to not boot or run on Edv4/Edsv4 VM sizes. Viewing the Azure serial console log of the VM will show a Linux kernel panic early in the boot process.  Other Linux distributions or virtual appliances based on a Linux kernel version 4.6 or earlier will similarly not boot or run.  An update to the Azure host is planned in the next few weeks that will resolve this issue and enable the full range of Linux distros and virtual appliances to operate properly.

## Edv4-series

Edv4-series sizes run on the Intel&reg; Xeon&reg; Platinum 8272CL (Cascade Lake) processors. The Edv4 virtual machine sizes feature up to 504 GiB of RAM, in addition to fast and large local SSD storage (up to 2,400 GiB). These virtual machines are ideal for memory-intensive enterprise applications and applications that benefit from low latency, high-speed local storage. You can attach Standard SSDs and Standard HDDs disk storage to the Edv4 VMs. Edv4-series VMs feature [Intel&reg; Hyper-Threading Technology](https://www.intel.com/content/www/us/en/architecture-and-technology/hyper-threading/hyper-threading-technology.html).

> [!NOTE]
> To use Ultra Disk or Premium SSD disk storage, select the Edsv4-series.

ACU: 195 - 210

Premium Storage:  Not Supported

Premium Storage caching:  Not Supported

Live Migration: Supported

Memory Preserving Updates: Supported

| Size | vCPU | Memory: GiB | Temp storage (SSD) GiB | Max data disks | Max cached and temp storage throughput: IOPS/MBps (cache size in GiB) | Max uncached disk throughput: IOPS/MBps | Max NICs/Expected Network bandwidth (Mbps) |
|---|---|---|---|---|---|---|---|
| Standard_E2d_v4  | 2 | 16 | 75 | 4 | 19000/120(50) | 3200/48 | 2/1000 |
| Standard_E4d_v4  | 4 | 32 | 150 | 8 | 38500/242(100) | 6400/96 | 2/2000 |
| Standard_E8d_v4 | 8 | 64 | 300 | 16 | 77000/485(200) | 12800/192 | 4/4000 |
| Standard_E16d_v4 | 16 | 128 | 600 | 32 | 154000/968(400) | 25600/384 | 8/8000 |
| Standard_E20d_v4 | 20 | 160 | 750 | 32 | 193000/1211(500)  | 32000/480  | 8/10000 |
| Standard_E32d_v4 | 32 | 256 | 1200 | 32 | 308000/1936(800) | 51200/768  | 8/16000 |
| Standard_E48d_v4 | 48 | 384 | 1800 | 32 | 462000/2904(1200) | 76800/1152 | 8/24000 |
| Standard_E64d_v4 | 64 | 504 | 2400 | 32 | 615000/3872(1600) | 80000/1200 | 8/30000 |


## Edsv4-series

Edsv4-series sizes run on the Intel&reg; Xeon&reg; Platinum 8272CL (Cascade Lake) processors. The Edsv4 virtual machine sizes feature up to 504 GiB of RAM, in addition to fast and large local SSD storage (up to 2,400 GiB). These virtual machines are ideal for memory-intensive enterprise applications and applications that benefit from low latency, high-speed local storage. Edsv4-series VMs feature [Intel&reg; Hyper-Threading Technology](https://www.intel.com/content/www/us/en/architecture-and-technology/hyper-threading/hyper-threading-technology.html).

ACU: 195-210

Premium Storage:  Supported

Premium Storage caching:  Supported

Live Migration: Supported

Memory Preserving Updates: Supported

| Size | vCPU | Memory: GiB | Temp storage (SSD) GiB | Max data disks | Max cached and temp storage throughput: IOPS/MBps (cache size in GiB) | Max uncached disk throughput: IOPS/MBps | Max NICs/Expected Network bandwidth (Mbps) |
|---|---|---|---|---|---|---|---|
| Standard_E2ds_v4  | 2 | 16 | 75 | 4 | 19000/120(50) | 3200/48 | 2/1000 |
| Standard_E4ds_v4  | 4 | 32 | 150 | 8 | 38500/242(100) | 6400/96 | 2/2000 |
| Standard_E8ds_v4 | 8 | 64 | 300 | 16 | 77000/485(200) | 12800/192 | 4/4000 |
| Standard_E16ds_v4 | 16 | 128 | 600 | 32 | 154000/968(400) | 25600/384 | 8/8000 |
| Standard_E20ds_v4 | 20 | 160 | 750 | 32 | 193000/1211(500)  | 32000/480  | 8/10000 |
| Standard_E32ds_v4 | 32 | 256 | 1200 | 32 | 308000/1936(800) | 51200/768  | 8/16000 |
| Standard_E48ds_v4 | 48 | 384 | 1800 | 32 | 462000/2904(1200) | 76800/1152 | 8/24000 |
| Standard_E64ds_v4 <sup>1</sup> | 64 | 504 | 2400 | 32 | 615000/3872(1600) | 80000/1200 | 8/30000 |

<sup>1</sup> Constrained core sizes available.


[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## Other sizes

- [General purpose](sizes-general.md)
- [Memory optimized](sizes-memory.md)
- [Storage optimized](sizes-storage.md)
- [GPU optimized](sizes-gpu.md)
- [High performance compute](sizes-hpc.md)
- [Previous generations](sizes-previous-gen.md)

## Next steps

Learn more about how [Azure compute units (ACU)](acu.md) can help you compare compute performance across Azure SKUs.
