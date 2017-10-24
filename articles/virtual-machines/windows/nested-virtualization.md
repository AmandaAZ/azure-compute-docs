---
title: How to Enable Nested Virtualization in Azure Virtual Machines | Microsoft Docs 
description: How To - Enable Nested Virtualization in Azure Virtual Machines
services: virtual-machines-windows
documentationcenter: virtual-machines
author: philmea
manager: timlt

ms.author: philmea
ms.date: 10/09/2017
ms.topic: howto
ms.service: virtual-machines-windows
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.author: philmea

---
# How to enable nested virtualization in an Azure VM

Nested virtualization is supported in the Dv3 and Ev3 series of Azure virtual machines. This capability provides great flexibility in supporting scenarios such as development, testing, training, and demonstration environments. 

This article steps through enabling nested virtualization on an Azure VM and configuring Internet connectivity to that guest virtual machine.

## Create a Dv3 or Ev3 series Azure VM

Create a new Windows Server 2016 Azure VM and choose a size from the Dv3 or Ev3 series. Ensure you choose a size large enough to support the demands of a guest virtual machine. In this example, we are using a D3_v3 size Azure VM. 

You can view the regional availability of Dv3 or Ev3 series virtual machines [here](https://azure.microsoft.com/en-us/regions/services/).

    >[!NOTE]
    >
    >For detailed instructions on creating a new virtual machine, see [Create and Manage Windows VMs with the Azure PowerShell module](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/tutorial-manage-vm)
    
## Connect to your virtual machine

Create a remote desktop connection to the virtual machine.

1. Click the **Connect** button on the virtual machine properties. A Remote Desktop Protocol file (.rdp file) is created and downloaded.

2. To connect to your VM, open the downloaded RDP file. If prompted, click **Connect**. On a Mac, you need an RDP client such as this [Remote Desktop Client](https://itunes.apple.com/us/app/microsoft-remote-desktop/id715768417?mt=12) from the Mac App Store.

3. Enter the user name and password you specified when creating the virtual machine, then click **Ok**.

4. You may receive a certificate warning during the sign-in process. Click **Yes** or **Continue** to proceed with the connection.

## Enable the Hyper-V Feature on the Azure VM

1. On the Azure VM, open PowerShell as an Administrator. 

2. Enable the Hyper-V feature and Management Tools.

    ```powershell
    Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
    ```

    >[!WARNING] 
    >
    >This command restarts the Azure VM. You will lose your RDP connection during the restart process.
    
3. After the Azure VM restarts, reconnect to your VM using RDP.

## Set up Internet connectivity for the guest virtual machine
Create a new virtual network adapter for the guest virtual machine and configure a NAT Gateway to enable Internet connectivity.

You can either manually assign an IP address to the guest virtual machine or use DHPC to dynamically assign an address. Instructions for configuring a DHCP server are included in the article.

### Create a NAT virtual network switch

1. On the Azure VM, open PowerShell as an Administrator.
   
2. Create an internal switch.

    ```powershell
    New-VMSwitch -SwitchName "InternalNATSwitch" -SwitchType Internal
    ```

3. View the properties of the switch and note the ifIndex for the new adapter.

    ```powershell
    Get-NetAdapter
    ```

    ![NetAdapter](./media/virtual-machines-nested-virtualization/get-netadapter.png)

    >[!NOTE] 
    >
    >Take note of the "ifIndex" for the virtual switch you just created.
    
4. Create an IP address for the NAT Gateway.
    
In order to configure the gateway, you need some information about your network:    
  * IPAddress - The NAT Gateway IP specifies the IPv4 or IPv6 address to use as the default gateway address for the virtual network subnet. The generic form is a.b.c.1 (for example, "192.168.0.1"). While the final position doesn’t have to be .1, it usually is (based on prefix length). Typically you should use an RFC 1918 private network address space. 
  * PrefixLength - The subnet prefix length defines the local subnet size (subnet mask). The subnet prefix length will be an integer value between 0 and 32. 0 would map the entire internet, 32 would only allow one mapped IP. Common values range from 24 to 12 depending on how many IPs need to be attached to the NAT. A common PrefixLength is 24 -- this is a subnet mask of 255.255.255.0.
  * InterfaceIndex - **ifIndex** is the interface index of the virtual switch created in the previous step. 

    ```powershell
    New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 13
    ```

### Create the NAT network.

In order to configure the gateway, you will need to provide information about the network and NAT Gateway:
  * Name - This is the name of the NAT network. 
  * InternalIPInterfaceAddressPrefix - The NAT subnet prefix describes both the NAT Gateway IP prefix from above as well as the NAT Subnet Prefix Length from above. The generic form will be a.b.c.0/NAT Subnet Prefix Length. 

1. In PowerShell, create a new NAT network.

    ```powershell
    New-NetNat -Name "InternalNat" -InternalIPInterfaceAddressPrefix 192.168.0.0/24
    ```

## (Optional) Use DHCP to dynamically assign an IP address
Follow the steps below to configure DHCP on the host virtual machine for dynamic address assignment.

### Install DCHP Server on the Azure VM

1. Open Server Manager. On the Dashboard, click **Add roles and features**. The Add Roles and Features Wizard appears.
  
2. In wizard, click **Next** until the Server Roles page.
  
3. Click to select the **DHCP Server** checkbox, click **Add Features**, and then click**Next** until you complete the wizard.
  
4. Click **Install**.

### Configure a new DHCP Scope

1. Open DHCP Manager.
  
2. In the navigation pane, expand the server name, right-click **IPv4**, and click **New Scope**. The New Scope Wizard appears, click **Next**.
  
3. Enter a Name and Description for the scope and click **Next**.
  
4. Define an IP Range for your DCHP Server (for example, 192.168.0.100 to 192.168.0.200).
  
5. Click **Next** until the Default Gateway page. Enter the IP Address you created earlier (for example, 192.168.0.1) as the Default Gateway.
  
6. Click **Next** until the wizard completes, leaving all default values, then click **Finish**.

## Test connectivity in guest virtual machine

1. Open Hyper-V Manager and create a new virtual machine. Configure the virtual machine to use the new Internal network you created.
    
    ![NetworkConfig](./media/virtual-machines-nested-virtualization/configure-networking.png)
    
2. Install a guest operating system on the guest virtual machine.
    
    >[!NOTE] 
    >
    >You need installation media for an operating system to install on the VM. In this case we are using Windows 10 Enterprise.
    
3. Connect to the guest virtual machine.
    
    ![GuestVM](./media/virtual-machines-nested-virtualization/guest-virtual-machine.png)
    
## Use a PowerShell script to configure nested virtualization
A PowerShell script to enable nested virtualization on a Windows Server 2016 host is available on [GitHub](https://github.com/Microsoft/Virtualization-Documentation/tree/master/hyperv-tools/Nested). The script checks pre-requisites and then configures nested virtualization on the Azure VM. A restart of the Azure VM is necessary to complete the configuration. This script may work in other environments but is not guaranteed. 
Check out the Azure blog post with a live video demonstration on nested virtualization running on Azure! https://aka.ms/AzureNVblog.