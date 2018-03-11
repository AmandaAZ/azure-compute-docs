---
title: Azure Virtual Machine Serial Console | Microsoft Docs
description: Bi-Directional Serial console for Azure Virtual Machines.
services: virtual-machines-windows
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

Azure's Virtual Machine Serial Console provides access to a text-based console for Linux and Windows Virtual Machines on Azure. This serial connection is to COM1 serial port of the virtual machine and provides access to the virtual machine regardless of that virtual machine's network / operating system state. Access to serial console for a virtual machine can be done only via Azure portal currently and allowed only for those users who have VM Contributor or above access to the virtual machine. 

### Important information
Currently this service is in **preview** and access to serial console for virtual machines is available to global Azure regions. At this point serial console is not available Azure Government, Azure Germany, and Azure China cloud.

### Requirements for accessing serial console 

1. Virtual Machine that requires access to serial console MUST have [boot diagnostics](boot-diagnostics.md) enabled 
2. User Accessing Serial console MUST have [Contributor role](../../active-directory/role-based-access-built-in-roles.md) for Virtual Machine and its [boot diagnostics](boot-diagnostics.md) storage account. 

## Accessing Serial Console
Serial Console for Virtual Machines is only accessible via [Azure portal](https://portal.azure.com). Below are the steps to access Serial Console for Virtual Machines via portal 

1. Open the Virtual Machine that you want to access in Azure portal 
2. Scroll down onto the Support + Troubleshooting section 
3. Click on Serial Console (Preview) option to access serial console 

![](../media/virtual-machines-serial-console/virtual-machine-windows-serial-console-connect.gif)

> [!NOTE] 
> Note that serial console requires a local user with a password configured. At this time, VMs only configured with SSH public key will not have access to serial console. To, create a local user with password follow [VM Access Extension](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/using-vmaccess-extension) and create local user with password.

## Serial Console  Security 

### Access Security 
Access to Serial console is limited to users who have [VM Contributors](../../active-directory/role-based-access-built-in-roles#virtual-machine-contributor) or above access to the Virtual Machine. If your AAD tenant requires Multi-Factor Authentication then access to serial console will also need MFA as its access is via [Azure portal](https://portal.azure.com).

### Channel Security
All data is sent back and forth is encrypted on the wire.

### Audit logs
All access to serial console is currently logged in the [boot diagnostics](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/boot-diagnostics) logs of the virtual machine. Access to these logs are owned and controlled by the Azure Virtual Machine administrator.  

>[!CAUTION] 
While no access passwords for the console are logged, if commands run within the console contain or output passwords, secrets, user names or any other form of Personally Identifiable Information (PII), those will be written to the Virtual Machine boot diagnostics logs, along with all other visible text, as part of the implementation of the serial console's scrollback functionality. These logs are circular and only individuals with read permissions to the diagnostics storage account have access to them, however we recommend following the best practice of using the SSH console for anything that may involve secrets and/or PII. 

### Concurrent usage
If a user is connected to Serial Console and another user successfully requests access to that same virtual machine, the first user will be disconnected and the second user connected in a manner akin to the first user standing up and leaving the physical console and a new user sitting down.

>[!CAUTION] 
This means that the user who gets disconnected will not be logged out! The ability to enforce a logout upon disconnect (via SIGHUP or similar mechanism) is still in the roadmap.For Windows there is an automatic timeout enabled in SAC, however for Linux you can configure terminal timeout setting. 

### Disable feature
The serial console functionality can be deactivated for specific VMs by disabling that VM's boot diagnostics setting.

## Common Scenarios for accessing Windows serial console 
Scenario          | Actions in serial console                
------------------|:----------------------------------------
Incorrect firewall rules | Access serial console and fix iptables or Windows firewall rules 
Filesystem corruption/check | Access serial console and recover filesystem after logging in to SAC CMD
RDP configuration issues | Access serial console and log in to cmd channel. Now you can check health of the Terminal services and restart if needed.
Network lock down system| Access Serial Console and log in to cmd channel. Now you can the firewall status by [netsh](https://docs.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh-contexts) command line. 

## Accessing Serial Console for Windows 
Newer Windows Server images on Azure  have [Special Administrative Console](https://technet.microsoft.com/en-us/library/cc787940(v=ws.10).aspx) (SAC) enabled by default. To enable Serial console for Windows Virtual Machines created with using Feb2018 or lower images please the following steps: 

1. Connect to your Windows virtual machine via Remote Desktop
2. From an Administrative command prompt run the following commands 
* `bcdedit /ems {current} on`
* `bcdedit /emssettings EMSPORT:1 EMSBAUDRATE:115200`
3. Reboot the system for the SAC console to be enabled

![](https://github.com/Microsoft/azserialconsole/blob/master/images/SerialConsole-PrivatePreviewWindows.gif)

### How do I know if SAC is enabled or not 

If [SAC] (https://technet.microsoft.com/en-us/library/cc787940(v=ws.10).aspx) is not enabled the serial console will not show the SAC prompt. It can show a VM Health information in some cases or it would be blank.  

### Enabling boot menu to show in serial console 

If you need to enable Windows boot loader prompts to show in serial console you can add the following additional options to bcd.

1. Connect to your Windows virtual machine via Remote Desktop
2. From an Administrative command prompt run the following commands 
* `bcdedit /set {bootmgr} displaybootmenu yes`
* `bcdedit /set {bootmgr} timeout 5`
* `bcdedit /set {bootmgr} bootems yes`
3. Reboot the system for the boot menu to be enabled

## Errors
Most errors are transient in nature and retrying connection address these. Below table shows a list of errors and mitigation 

Error                            |   Mitigation 
---------------------------------|:--------------------------------------------:|
Unable to retrieve boot diagnostics settings for ''. To use serial console, ensure that boot diagnostics is enabled for this VM. | Ensure that the VM has [boot diagnostics](boot-diagnostics.md)) enabled. 
The VM is in a stopped deallocated state. Start the VM and retry the serial console connection. | Virtual machine must be in a started state to access serial console
You do not have the required permissions to use this VM serial console. Ensure you have at least VM Contributor role permissions.| Serial console access requires certain permission to access. See [access requirements](#requirements-for-accessing-serial-console) for details
Unable to determine the resource group for the boot diagnostics storage account ''. Verify that boot diagnostics is enabled for this VM and you have access to this storage account. | Serial console access requires certain permission to access.See [access requirements](#requirements-for-accessing-serial-console) for details

## Known Issues 
As we are still in the preview stages for Serial Console access, we are working through some known issues, below is the list of these with possible workarounds 

Issue                           |   Mitigation 
---------------------------------|--------------------------------------------|
There is no option with virtual machine scale set instance Serial Console | At the time of preview, access to serial console for virtual machine scale set instances is not supported.
Hitting enter after the connection banner does not show a log in prompt | [Hitting enter does nothing](https://github.com/Microsoft/azserialconsole/blob/master/Known_Issues/Hitting_enter_does_nothing.md)
Only health information is shown when connecting to a Windows VM| [Windows Health Signals](https://github.com/Microsoft/azserialconsole/blob/master/Known_Issues/Windows_Health_Info.md)

## Availability 
The current preview is available in global Azure regions. At this point serial console is not available in Azure Germany, Azure Government, and Azure China. 

## FAQ
1. How can I send feedback?
* Provide feedback as an issue by going to aka.ms/serialconsolefeedback . Alternatively(less preferred) Send feedback via azserialhelp@microsoft.com or in the virtual machine category of http://feedback.azure.com 
2. I get an Error "Existing console has conflicting OS type "Windows" with the requested OS type of Linux
* This is a known issue to fix this, simply open [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview) in bash mode and retry.
3. I am not able to access serial console, where can I file a support case?
* This is a preview feature is covered via Azure Preview Terms. Support for this is best handled via channels mentioned above. 

## Next Steps
Learn more about [bootdiagnostics](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/boot-diagnostics)