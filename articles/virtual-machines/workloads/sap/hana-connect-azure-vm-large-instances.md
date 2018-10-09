---
title: Connectivity setup from virtual machines to SAP HANA on Azure (Large Instances) | Microsoft Docs
description: Connectivity setup from virtual machines for using SAP HANA on Azure (Large Instances).
services: virtual-machines-linux
documentationcenter: 
author: RicksterCDN
manager: jeconnoc
editor:

ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: rclaus
ms.custom: H1Hack27Feb2017

---

# Connecting Azure VMs to HANA Large Instances

As already mentioned in [SAP HANA (Large Instances) overview and architecture on Azure](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture), the minimal deployment of HANA Large Instances with the SAP application layer in Azure looks like the following:

![Azure VNet connected to SAP HANA on Azure (Large Instances) and on-premises](./media/hana-overview-architecture/image3-on-premises-infrastructure.png)

Looking closer at the Azure virtual network side, we realize the need for:

- The definition of an Azure virtual network into which you're going to deploy the VMs of the SAP application layer.
- The definition of a default subnet in the Azure virtual network that is really the one into which the VMs are deployed.
- The Azure virtual network that's created needs to have at least one VM subnet and one ExpressRoute virtual network gateway subnet. These subnets should be assigned the IP address ranges as specified and discussed in the following sections.

So, let's look a bit closer at the Azure virtual network creation for HANA Large Instances.

## Create the Azure virtual network for HANA Large Instances

>[!Note]
>The Azure virtual network for HANA Large Instances must be created by using the Azure Resource Manager deployment model. The older Azure deployment model, commonly known as the classic deployment model, isn't  supported by the HANA Large Instance solution.

The virtual network can be created by using the Azure portal, PowerShell, an Azure template, or the Azure CLI. (For more information, see [Create a virtual network using the Azure portal](../../../virtual-network/manage-virtual-network.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#create-a-virtual-network)). In the following example, we look at a virtual network created by using the Azure portal.

If we look into the definitions of an Azure virtual network through the Azure portal, we'll see how these definitions relate to what we list as different IP address ranges. When we're talking about the **address space**, we mean the address space that the Azure virtual network is allowed to use. This address space is also the address range that the virtual network uses for BGP route propagation. This **address space** can be seen here:

![Address space of an Azure virtual network displayed in the Azure portal](./media/hana-overview-connectivity/image1-azure-vnet-address-space.png)

In previous example, with 10.16.0.0/16, the Azure virtual network was given a rather large and wide IP address range to use. This means that all the IP address ranges of subsequent subnets within this virtual network can have their ranges within that address space. Usually, we don't  recommend such a large address range for single virtual network in Azure. But going a step further, let's look into the subnets that are defined in the Azure virtual network:

![Azure virtual network subnets and their IP address ranges](./media/hana-overview-connectivity/image2b-vnet-subnets.png)

We look at a virtual network with a first VM subnet (here called "default") and a subnet called "GatewaySubnet."

In the following section, we refer to the IP address range of the subnet, which was called "default" in the graphics, as **Azure VM subnet IP address range**. In the following sections, we refer to the IP address range of the gateway subnet as **virtual network gateway subnet IP address range**. 

In the example of the two previous graphics graphics, you see that that the **virtual network address space** covers both the **Azure VM subnet IP address range** and the **virtual network gateway subnet IP address range**. 

In other cases where you need to conserve or be specific with your IP address ranges, you can restrict the **virtual network address space** of a virtual network to the specific ranges that are being used by each subnet. In this case, you can define the **virtual network address space** of a virtual network as multiple specific ranges, as shown here:

![Azure virtual network addresss space with two spaces](./media/hana-overview-connectivity/image3-azure-vnet-address-space_alternate.png)

In this case, the **virtual network address space** has two spaces defined. These two spaces are identical to the IP address ranges that are defined for **Azure VM subnet IP address range** and the **virtual network gateway subnet IP address range.**

You can use any naming standard you like for these tenant subnets (VM subnets). However, **there must always be one, and only one, gateway subnet for each virtual network** that connects to the SAP HANA on Azure (Large Instances) ExpressRoute circuit. And **this gateway subnet must always be named "GatewaySubnet"** to ensure proper placement of the ExpressRoute gateway.

> [!WARNING] 
> It is critical that the gateway subnet always is named "GatewaySubnet."

You can use multiple VM subnets, and even utilize non-contiguous address ranges. But as mentioned previously, these address ranges must be covered by the **virtual network address space** of the virtual network, either in an aggregated form or in a list of the exact ranges of the VM subnets and the gateway subnet.

Following is a summary of the important fact about an Azure virtual network that connects to HANA Large Instances:

- You need to submit the **virtual network address space** to Microsoft  when you're doing an initial deployment of HANA Large Instances. 
- The **virtual network address space** can be one larger range that covers the range for Azure VM subnet IP address range(s) and the VNet gateway subnet IP address range.
- Or you can submit as **virtual network address space** multiple ranges that cover the different IP address ranges of VM subnet IP address range(s) and the virtual network gateway IP address range.
- The defined **virtual network address space** is used for BGP routing propagation.
- The name of the gateway subnet must be: **"GatewaySubnet."**
- The  address space is used as a filter on the HANA Large Instance side to allow or disallow traffic to the HANA Large Instance units from Azure. If the BGP routing information of the Azure virtual network and the IP address ranges that are configured for filtering on the HANA Large Instance side do not match, connectivity issues can occur.
- There are some details about the gateway subnet that are discussed later, in the section **Connecting a virtual network to HANA Large Instance ExpressRoute.**



## Different IP address ranges to be defined 

We already introduced some of the IP address ranges that are necessary to deploy HANA Large Instances in earlier sections. But there are more IP address ranges that are also important. Let's go through some more  details. The following IP addresses, which don't all need to be submitted to Microsoft, need to be defined before sending a request for initial deployment:

- **Virtual network address space** As introduced earlier, the **virtual network address space** is the IP address range (or ranges) that you have assigned (or plan to assign) to your address space parameter in the Azure virtual network(s) that connect to the SAP HANA Large Instance environment.

 We recommend that this address space parameter is a multi-line value comprised of the Azure VM subnet range(s) and the Azure gateway subnet range as shown in the previous graphics. This range must NOT overlap with your on-premises or Server IP Pool or ER-P2P address ranges. 
 
How do you get these IP address range(s)? 

Your corporate network team or service provider should provide one or multiple IP address range(s) that aren't  used inside your network. For example, if your Azure VM subnet (see earlier) is 10.0.1.0/24, and your Azure gateway subnet (see the following) is 10.0.2.0/28, then we recommend that your Azure VNet address space is two lines: 10.0.1.0/24 and 10.0.2.0/28. 

Although the  address space values can be aggregated, we recommend matching them to the subnet ranges to avoid accidentally reusing unused IP address ranges within larger address spaces in the future elsewhere in your network. **The virtual network address space is an IP address range, which needs to be submitted to Microsoft when you ask for an initial deployment**.

- **Azure VM subnet IP address range:** This IP address range, as discussed earlier, is the one you have assigned (or plan to assign) to the Azure virtual network subnet parameter in your Azure virtual network that connects to the SAP HANA Large Instance environment. This IP address range is used to assign IP addresses to your Azure VMs. The IP addresses out of this range are allowed to connect to your SAP HANA Large Instance server(s). If needed, you can use multiple Azure VM subnets. We recommend a /24 CIDR block for each Azure VM subnet. This address range must be a part of the values that are used in the Azure virtual network address space. 

How do you get this IP address range? 

Your corporate network team or service provider should provide an IP address range that is not currently used inside your network.

- **VNet Gateway Subnet IP address range:** Depending on the features that you plan to use, the recommended size would be:
   - Ultra-performance ExpressRoute gateway: /26 address block--required for Type II class of SKUs.
   - Co-existence with VPN and ExpressRoute using a high-performance ExpressRoute virtual network gateway (or smaller): /27 address block.
   - All other situations: /28 address block. This address range must be a part of the values used in the "VNet address space" values. This address range must be a part of the values that are used in the Azure virtual network address space values that you submit to Microsoft. How do you get this IP address range? Your corporate network team or service provider should provide an IP address range that's not currently used inside your network. 

- **Address range for ER-P2P connectivity:** This range is the IP range for your SAP HANA Large Instance ExpressRoute (ER) P2P connection. This range of IP addresses must be a /29 CIDR IP address range. This range must NOT overlap with your on-premises or other Azure IP address ranges. This IP address range is used to set up the ER connectivity from your ExpressRoute virtual gateway to the SAP HANA Large Instance servers. How do you get this IP address range? Your corporate network team or service provider should provide an IP address range that is not currently used inside your network. **This range is an IP address range, which needs to be submitted to Microsoft when asking for an initial deployment**.
  
- **Server IP Pool address range:** This IP address range is used to assign the individual IP address to HANA large instance servers. The recommended subnet size is a /24 CIDR block. If needed, it can be smaller, with a  minimum of  64 IP addresses. From this range, the first 30 IP addresses are reserved for use by Microsoft. Ensure that this is accounted for when you choose the size of the range. This range must NOT overlap with your on-premises or other Azure IP addresses. How do you get this IP address range? Your corporate network team or service provider should provide an IP address range that is not currently used inside your network. A /24 (recommended) unique CIDR block to be used for assigning the specific IP addresses is needed for SAP HANA on Azure (Large Instances). **This range is an IP address range, which needs to be submitted to Microsoft when asking for an initial deployment**.
 
Though you need to define and plan the IP address ranges that were described previously, not all them need to be transmitted to Microsoft. To summarize, the IP address ranges that you are required to name to Microsoft are:

- Azure virtual network address space(s)
- Address range for ER-P2P connectivity
- Server IP pool address range

If you add additional virtual networks that need to connect to HANA Large Instances, you have to submit the new Azure virtual network address space that you're adding to Microsoft. 

Following is an example of the different ranges and some example ranges as you need to configure and eventually provide them to Microsoft. As you can see, the value for the Azure VNet address space is not aggregated in the first example, but is defined from the ranges of the first Azure VM subnet IP address range and the virtual network gateway subnet IP address range. 

Using multiple VM subnets within the Azure virtual network works when you configure and submit the additional IP address ranges of the additional VM subnet(s) as part of the Azure VNet address space.

![IP address ranges required in SAP HANA on Azure (Large Instances) minimal deployment](./media/hana-overview-connectivity/image4b-ip-addres-ranges-necessary.png)

You also have the possibility of aggregating the data that you submit to Microsoft. In that case, the address space of the Azure virtual network only includes one space. Using the IP address ranges used in the example earlier. This aggregated virtual network address space could look like the following:

![Second possibility of IP address ranges required in SAP HANA on Azure (Large Instances) minimal deployment](./media/hana-overview-connectivity/image5b-ip-addres-ranges-necessary-one-value.png)

As you can see in the example, instead of two smaller ranges that defined the address space of the Azure virtual network, we have one larger range that covers 4096 IP addresses. Such a large definition of the address space leaves some rather large ranges unused. Since the virtual network address space value(s) are used for BGP route propagation, usage of the unused ranges on-premises or elsewhere in your network can cause routing issues. So  we recommended that you keep the address space tightly aligned with the actual subnet address space that you use. If needed, without incurring downtime on the virtual network, you can always add new address space values later.
 
> [!IMPORTANT] 
> Each IP address range in ER-P2P, Server IP Pool, and Azure virtual network address space must **NOT** overlap with one another or with any other range that's used in your network. Each must be discrete. As the two previous graphics show, they also can't be a subnet of any other range. If overlaps occur between ranges, the Azure virtual network might not connect to the ExpressRoute circuit.

## Next steps after address ranges have been defined
After the IP address ranges have been defined, the following things need to happen:

1. Submit the IP address ranges for Azure virtual network address space, the ER-P2P connectivity, and Server IP Pool Address Range, together with other data that has been listed at the beginning of the document. At this point, you could also start to create the virtual network and the VM Subnets. 
2. An Express Route circuit is created by Microsoft between your Azure subscription and the HANA Large Instance stamp.
3. A tenant network is created on the Large Instance stamp by Microsoft.
4. Microsoft configures networking in the SAP HANA on Azure (Large Instances) infrastructure to accept IP addresses from your Azure virtual network address space that communicates with HANA Large Instances.
5. Depending on the specific SAP HANA on Azure (Large Instances) SKU that you purchased, Microsoft assigns a compute unit in a tenant network, allocates and mounts storage, and installs the operating system (SUSE or Red Hat Linux). IP addresses for these units are taken out of the Server IP Pool address range that you submitted to Microsoft.

At the end of the deployment process, Microsoft delivers the following data to you:
- Information is needed to connect your Azure virtual network(s) to the ExpressRoute circuit that connects Azure virtual networks to HANA Large Instances:
     - Authorization key(s)
     - ExpressRoute PeerID
- Data for accessing HANA Large Instances after you establish ExpressRoute circuit and Azure virtual network.

You can also find the sequence of connecting HANA Large Instances in the document [SAP HANA on Azure (Large Instances) Setup](https://azure.microsoft.com/resources/sap-hana-on-azure-large-instances-setup/). Many of the following steps are shown in an example deployment in that document. 

**Next steps**

- Refer to [Connecting a virtual network to HANA Large Instance ExpressRoute](hana-connect-vnet-express-route.md).