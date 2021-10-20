---
title: Edv5 and Edsv5-series 
description: Specifications for the Edv5 and Edsv5-series VMs.
author: styli365
ms.author: sttsinar
ms.reviewer: joepell
ms.custom: mimckitt
ms.service: virtual-machines
ms.subservice: vm-sizes-memory
ms.topic: conceptual
ms.date: 10/20/2021
---

# Edv5 and Edsv5-series

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

The Edv5 and Edsv5-series virtual machines run on the 3rd Generation Intel&reg; Xeon&reg; Platinum 8370C (Ice Lake) processors in a [hyper-threaded](https://www.intel.com/content/www/us/en/architecture-and-technology/hyper-threading/hyper-threading-technology.html) configuration featuring an all core Turbo clock speed of 3.5 GHz. Thes VMs are ideal for various memory-intensive enterprise applications and feature up to 672GiB of RAM. These VMs also feature fast and large local SSD storage (up to 3,600 GiB).

## Edv5-series

Edv5-series sizes run on the 3rd Generation Intel&reg; Xeon&reg; Platinum 8370C (Ice Lake) processors with [Intel&reg; Turbo Boost Technology 2.0](https://www.intel.com/content/www/us/en/architecture-and-technology/turbo-boost/turbo-boost-technology.html), [Intel&reg; Hyper-Threading Technology](https://www.intel.com/content/www/us/en/architecture-and-technology/hyper-threading/hyper-threading-technology.html), [Intel&reg; Advanced Vector Extensions 512 (Intel&reg; AVX-512)](https://www.intel.com/content/www/us/en/architecture-and-technology/avx-512-overview.html) and [Intel&reg; Deep Learning Boost](https://software.intel.com/content/www/us/en/develop/topics/ai/deep-learning-boost.html). The Edv5 virtual machine sizes feature up to 672 GiB of RAM, in addition to fast and large local SSD storage (up to 3,600 GiB). 

These virtual machines are ideal for memory-intensive enterprise applications and applications that benefit from low latency, high-speed local storage. You can attach Standard SSDs and Standard HDDs disk storage to these VMs. To use Premium SSD disk storage, select Edsv5 VMs. Disk storage is billed separately from virtual machines. [See pricing for disks](https://azure.microsoft.com/pricing/details/managed-disks/).

[Premium Storage](premium-storage-performance.md): Not Supported<br>
[Premium Storage caching](premium-storage-performance.md): Not Supported<br>
[Live Migration](maintenance-and-updates.md): Supported<br>
[Memory Preserving Updates](maintenance-and-updates.md): Supported<br>
[VM Generation Support](generation-2.md): Generation 1 and 2<br>
[Accelerated Networking](../virtual-network/create-vm-accelerated-networking-cli.md): Supported <br>
[Ephemeral OS Disks](ephemeral-os-disks.md): Not Supported <br>
<br>

| Size | vCPU | Memory: GiB | Temp storage (SSD) GiB | Max data disks | <sup>**</sup> Max cached and temp storage throughput: IOPS/MBps | Max NICs|Expected Network bandwidth (Mbps) |
|---|---|---|---|---|---|---|---|
| Standard_E2d_v5<sup>1</sup>  | 2  | 16  | 75   | 4  | 19000/120   | 2 | 1000  |
| Standard_E4d_v5              | 4  | 32  | 150  | 8  | 38500/242   | 2 | 2000  |
| Standard_E8d_v5              | 8  | 64  | 300  | 16 | 77000/485   | 4 | 4000  |
| Standard_E16d_v5             | 16 | 128 | 600  | 32 | 154000/968  | 8 | 8000  |
| Standard_E20d_v5             | 20 | 160 | 750  | 32 | 193000/1211 | 8 | 10000 |
| Standard_E32d_v5             | 32 | 256 | 1200 | 32 | 308000/1936 | 8 | 16000 |
| Standard_E48d_v5             | 48 | 384 | 1800 | 32 | 462000/2904 | 8 | 24000 |
| Standard_E64d_v5             | 64 | 512 | 2400 | 32 | 615000/3872 | 8 | 30000 |
| Standard_E96d_v5             | 96 | 672 | 3600 | 32 | 615000/3872 | 8 | 30000 |


<sup>**</sup> These IOPs values can be guaranteed by using [Gen2 VMs](generation-2.md)<br>
<sup>1</sup> Accelerated networking can only be applied to a single NIC.

## Edsv5-series

Edsv5-series sizes run on the 3rd Generation Intel&reg; Xeon&reg; Platinum 8370C (Ice Lake) processors with [Intel&reg; Turbo Boost Technology 2.0](https://www.intel.com/content/www/us/en/architecture-and-technology/turbo-boost/turbo-boost-technology.html), [Intel&reg; Hyper-Threading Technology](https://www.intel.com/content/www/us/en/architecture-and-technology/hyper-threading/hyper-threading-technology.html), [Intel&reg; Advanced Vector Extensions 512 (Intel&reg; AVX-512)](https://www.intel.com/content/www/us/en/architecture-and-technology/avx-512-overview.html) and [Intel&reg; Deep Learning Boost](https://software.intel.com/content/www/us/en/develop/topics/ai/deep-learning-boost.html). The Edsv5 virtual machine sizes feature up to 672 GiB of RAM, in addition to fast and large local SSD storage (up to 3,600 GiB).

These virtual machines are ideal for memory-intensive enterprise applications and applications that benefit from low latency, high-speed local storage. You can attach Standard SSDs, Standard HDDs, and Premium SSDs disk storage to these VMs. You can also attach Ultra Disk storage based on its regional availability. Disk storage is billed separately from virtual machines. [See pricing for disks](https://azure.microsoft.com/pricing/details/managed-disks/).

[Premium Storage](premium-storage-performance.md): Supported<br>
[Premium Storage caching](premium-storage-performance.md): Supported<br>
[Live Migration](maintenance-and-updates.md): Supported<br>
[Memory Preserving Updates](maintenance-and-updates.md): Supported<br>
[VM Generation Support](generation-2.md): Generation 1 and 2<br>
[Accelerated Networking](../virtual-network/create-vm-accelerated-networking-cli.md): Supported <br>
[Ephemeral OS Disks](ephemeral-os-disks.md): Supported <br>
<br>

| Size | vCPU | Memory: GiB | Temp storage (SSD) GiB | Max data disks | <sup>**</sup> Max cached and temp storage throughput: IOPS/MBps (cache size in GiB) | Max uncached disk throughput: IOPS/MBps | Max NICs|Expected Network bandwidth (Mbps) |
|---|---|---|---|---|---|---|---|---|
| Standard_E2ds_v5<sup>1</sup>  | 2  | 16  | 75   | 4  | 19000/120(50)     | 3200/48    | 2 | 1000  |
| Standard_E4ds_v5              | 4  | 32  | 150  | 8  | 38500/242(100)    | 6400/96    | 2 | 2000  |
| Standard_E8ds_v5              | 8  | 64  | 300  | 16 | 77000/485(200)    | 12800/192  | 4 | 4000  |
| Standard_E16ds_v5             | 16 | 128 | 600  | 32 | 154000/968(400)   | 25600/384  | 8 | 8000  |
| Standard_E20ds_v5             | 20 | 160 | 750  | 32 | 193000/1211(500)  | 32000/480  | 8 | 10000 |
| Standard_E32ds_v5             | 32 | 256 | 1200 | 32 | 308000/1936(800)  | 51200/768  | 8 | 16000 |
| Standard_E48ds_v5             | 48 | 384 | 1800 | 32 | 462000/2904(1200) | 76800/1152 | 8 | 24000 |
| Standard_E64ds_v5             | 64 | 512 | 2400 | 32 | 615000/3872(1600) | 80000/1200 | 8 | 30000 |
| Standard_E96ds_v5             | 96 | 672 | 3600 | 32 | 615000/3872(1600) | 80000/1200 | 8 | 30000 |

<sup>1</sup> Accelerated networking can only be applied to a single NIC.

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## Other sizes and information

- [General purpose](sizes-general.md)
- [Memory optimized](sizes-memory.md)
- [Storage optimized](sizes-storage.md)
- [GPU optimized](sizes-gpu.md)
- [High performance compute](sizes-hpc.md)
- [Previous generations](sizes-previous-gen.md)

Pricing Calculator: [Pricing Calculator](https://azure.microsoft.com/pricing/calculator/)

More information on Disks Types : [Disk Types](./disks-types.md#ultra-disks)


## Next steps

Learn more about how [Azure compute units (ACU)](acu.md) can help you compare compute performance across Azure SKUs.
