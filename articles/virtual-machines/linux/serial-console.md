---
title: Azure Virtual Machine Serial Console | Microsoft Docs
description: Bi-Directional Serial console for Azure Virtual Machines.
services: virtual-machines-linux
documentationcenter: ''
author: harijayms
manager: timlt
editor: ''
tags: azure-resource-manager

ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 03/05/2017
ms.author: harijayms
---

# Azure's Virtual Machine Serial Console -- Preview 

> [!NOTE] 
> Previews are made available to you on the condition that you agree to the terms of use. For more information, see [Microsoft Azure Supplemental Terms of Use for Microsoft Azure Previews.] (https://azure.microsoft.com/en-us/support/legal/preview-supplemental-terms/)
> ACCESS VIA SERIAL CONSOLE SHOULD BE LIMITED TO DEV-TEST VIRTUAL MACHINES. PREVIEW FUNCTIONALITY IS SUPPORTED ON A AS IS BASIS. WE STRONGLY RECOMMEND NOT TO USE THIS FOR ANY PRODUCTION VIRTUAL MACHINE.
>

Azure's Virtual Machine Serial Console provides access to a text-based console for Linux and Windows Virtual Machines on Azure. This serial connection is to COM1 serial port of the virtual machine and provides access to the virtual machine regardless of that virtual machine's network / operating system state. Access to serial console for a virtual machine can be done only via Azure Portal currently and allowed only for those users who have VM Contributor or above access to the virtual machine. 

### Important information
Currently this service is in **preview** and access to serial console for virtual machines is available to public azure regions except UK regions. At this point serial console is not available Azure Government, Azure Germany and Azure China cloud.

### Requirements for Accessing Serial Console 

1. Virtual Machine that requires Serial Console MUST have [boot diagnostics](boot-diagnostics.md) enabled 
2. User Accessing Serial console MUST have [Contributor role](..\..\active-directory\role-based-access-built-in-roles.md) for Virtual Machine and its [boot diagnostics](boot-diagnostics.md) storage account. 


## Accessing Serial Console
Serial Console for Virtual Machines is only accessible via [Azure portal](https://portal.azure.com). Below are the steps to access Serial Console for Virtual Machines via portal 

1. Open the Virtual Machine that you want to access in Azure Portal 
2. Scroll down onto the Support + Troubleshooting section 
3. Click on Serial Console (Preview) option to access serial console 

![](../media/virtual-machines-serial-console/virtual-machine-windows-serial-console-connect.gif)

> [!NOTE] 
> Please also note that serial console requires a local user with a password configured. At this time, VMs only configured with SSH public key will not have access to serial console. To create a local user with password follow [VM Access Extension](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/using-vmaccess-extension) and create local user with password.

## Serial Console  Security 

### Access Security 
Access to Serial console is limited to users who have [VM Contributors](https://docs.microsoft.com/en-us/azure/active-directory/role-based-access-built-in-roles#virtual-machine-contributor) or above access to the Virtual Machine. If your AAD tenant requires Multi-Factor Authentication then access to serial console will also need MFA as its access is via [Azure Portal](https://portal.azure.com).

### Channel Security
All data is sent back and forth is encrypted on the wire.

### Audit logs
All access to Serial Console are currently logged in the [boot diagnostics](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/boot-diagnostics) logs of the virtual machine. Access to these logs are owned and controlled by the Azure Virtual Machine administrator.  

>[!CAUTION] While no access passwords for the console are logged, if commands run within the console contain or output passwords, secrets, user names or any other form of Personally Identifiable Information (PII), those will be written to the Virtual Machine boot diagnostics logs, along with all other visible text, as part of the implementation of the serial console's scrollback functionality. These logs are circular and only individuals with read permissions to the diagnostics storage account have access to them, however we recommend following the best practice of using the SSH console for anything that may involve secrets and/or PII. 

### Concurrent usage
If a user is connected to Serial Console and another user successfully requests access to that same virtual machine, the first user will be disconnected and the second user connected in a manner akin to the first user standing up and leaving the physical console and a new user sitting down.

>[!CAUTION] This means that the user who gets disconnected will not be logged out! The ability to enforce a logout upon disconnect (via SIGHUP or similar mechanism) is still in the roadmap.For Windows there is an automatic timeout enabled in SAC, however for Linux you can configure terminal timeout setting. 

### Disable feature
The serial console functionality can be deactivated for specific VMs by disabling that VM's boot diagnostics setting.

## Common Scenarios for accessing Serial Console 
Scenario          | Actions in Serial Console                |  OS Applicability 
------------------|:----------------------------------------:|------------------:
Broken FSTAB file | Enter key to continue and fix fstab file using a text editor. See [how to fix fstab issues](https://support.microsoft.com/en-us/help/3206699/azure-linux-vm-cannot-start-because-of-fstab-errors) | Linux 
Incorrect firewall rules | Access Serial Console and fix iptables or Windows firewall rules | Linux/Windows 
Filesystem corruption/check | Access Serial Console and recover filesystem | Linux/Windows 
SSH/RDP configuration issues | Access Serial Console and change settings | Linux/Windows 
Network lock down system| Access Serial Console via portal to manage system | Linux/Windows 
Interacting with bootloader | Access GRUB/BCD via serial console | Linux/Windows 

## Accessing Serial Console for Linux ( Distro Specific Scenarios ) 
In order for Serial Console to function properly, the guest operating system must be configured to read and write console messages to the serial port. Most [Endorsed Azure Linux Distributions](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/endorsed-distros) have serial console configured by default. Just by clicking in the portal on the Serial console section will provide access to the console. 

### Access for RedHat 
RedHat Images available on Azure have serial console enabled by default. Single User Mode in Red Hat requires root user to be enabled, which is disabled by default. If you have a need to enable Single user mode, use the following instructions 

1. Login to the Red Hat system via SSH
2. Enable password for root user 
 * `passwd root` ( set a strong root password )
3. Ensure root user can only login via ttyS0
 * `edit /etc/ssh/sshd_config` and ensure PermitRootLogin is set to no
 * `edit /etc/securetty file` to only allow logins via ttyS0 

Now if the system boots into single user mode you can login via root password.

Alternatively for RHEL 7.4+ or 6.9+ you can enable single user mode in the GRUB prompts , see instructions [here](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/installation_guide/s1-rescuemode-booting-single)

### Access for Ubuntu 
Ubuntu images available on Azure have serial console enabled by default. If the system boots into Single User Mode you can access without additional credentials. 

### Access for CoreOS
CoreOS images available on Azure have serial console enabled by default. If required system can be booted into Single User Mode via changing GRUB parameters and adding coreos.autologin=ttyS0 would enable core user to login and available in Serial Console. 

### Access for SUSE
SLES images available on Azure have serial console enabled by default. If you are using older versions of SLES on Azure please follow the [KB article](https://www.novell.com/support/kb/doc.php?id=3456486) to enable serial console. Newer Images of SLES 12 SP3+ also allows access via serial console in case the system boots into emergency mode.

### Access for CentOS
CentOS images available on Azure have serial console enabled by default. For Single User Mode please follow instructions similar to Red Hat Images above. 

### Access for Oracle Linux
Oracle Linux images available on Azure have serial console enabled by default. For Single User Mode please follow instructions similar to Red Hat Images above.

### Access for Custom Linux Image
To enable Serial Console for your custom Linux VM image, enable console in /etc/inittab to run a terminal on ttyS0. Below is an example to add this in the inittab file 

`S0:12345:respawn:/sbin/agetty -L 115200 console vt102` 

## Errors
Most errors are transient in nature and retrying connection address these. Below table shows a list of errors and mitigation 

Error                            |   Mitigation 
---------------------------------|:--------------------------------------------:|
Unable to retrieve boot diagnostics settings for ''. To use serial console, ensure that boot diagnostics is enabled for this VM. | Ensure that the VM has [boot diagnostics](boot-diagnostics.md)) enabled. 
The VM is in a stopped deallocated state.  Please start the VM and retry the serial console connection. | Virtual machine must be in a started state to access serial console
You do not have the required permissions to use this VM serial console.  Please ensure you have at least VM Contributor role permissions.| Serial console access requires certain permission to access. Please see [access requirements](#requirements-for-accessing-serial-console) for details
Unable to determine the resource group for the boot diagnostics storage account ''. Verify that boot diagnostics is enabled for this VM and you have access to this storage account. | Serial console access requires certain permission to access. Please see [access requirements](#requirements-for-accessing-serial-console) for details

## Known Issues 
As we are still in the preview stages for Serial Console access, we are working through some known issues, below is the list of these with possible workarounds 

Issue                           |   Mitigation 
---------------------------------|--------------------------------------------|
There is no option with VMSS instance Serial Console | At the time of preview we do not support access to serial console for VMSS instances 
Hitting enter after the connection banner does not show a login prompt | [Hitting enter does nothing](https://github.com/Microsoft/azserialconsole/blob/master/Known_Issues/Hitting_enter_does_nothing.md)


## Availability 
The current preview is available in global Azure regions. At this point serial console is not available in Azure Germany, Azure Government and Azure China. 

## FAQ
1. How can I send feedback?
* Please provide feedback as an issue by going to aka.ms/serialconsolefeedback . Alternatively(less preferred) Send feedback via azserialhelp@microsoft.com or in the virtual machine category of http://feedback.azure.com 
2. I am not a part of early access preview but I want to participate, how can I?
* We will be opening access to more subscriptions shortly.
3. I get an Error "Existing console has conflicting OS type "Windows" with the requested OS type of Linux
* This is a know issue to fix this, simply open [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview) in bash mode and retry.

## Next Steps
Learn more about [bootdiagnostics](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/boot-diagnostics)