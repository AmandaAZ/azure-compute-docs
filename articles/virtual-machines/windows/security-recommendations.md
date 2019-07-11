---
title: Security recommendations for Windows virtual machines in Azure
description: Security recommendations for the virtual machines in Azure. Implementing these recommendations will help you fulfill your security obligations as described in our shared responsibility model and will improve the overall security for your deployments
services: vm-windows
author: barclayn
manager: barbkess

ms.service: vm-service
ms.topic: conceptual
ms.date: 07/11/2019
ms.author: barclayn

---
<!-- We have simplified and limited the number of possible headers to:
- General
    - topics that don't fit into any other category but that are not coming up 
- Identity and access management
    - Authentication, authorization, MFA, RBAC, any other identity management topics
- Data security
    - data in transit, data at rest, TLS, HTTPS, any other topics that discuss protecting data from unauthorized access 
- Networking
    - 
- Monitoring
    - Auditing 
    - Logging 
    - Sentinel

Security center 
- business continuity 
-->

# Security recommendations for Windows Virtual machines in Azure

This article contains security recommendations for virtual machines in Azure. Implementing these recommendations will help you fulfill your security obligations as described in our shared responsibility model. For more information on what Microsoft does to fulfill service provider responsibilities, read [Azure infrastructure security](../../security/azure-security-infrastructure.md).

## General

| Recommendation | Comments |
|-|-|----|
| Build VM images using the latest updates | Before you create images ensure that all the latest updates are installed |
| Keep your VMs current | You can use the [Update Management](../../automation/automation-update-management.md) solution in Azure Automation to manage operating system updates for your Windows and Linux computers in Azure |

## Identity and access management

| Recommendation | Comments |
|-|----|
| Centralize VM authentication | You can centralize the authentication of your Windows and Linux virtual machines using [Azure AD authentication](../../active-directory/develop/authentication-scenarios.md). |
| Use multi-factor authentication  | Azure Multi-Factor Authentication (MFA) helps safeguard access to data and applications while maintaining simplicity for users. It requires a second form of authentication for more information, see [How it works: Azure Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md)|

## Data security

| Recommendation | Comments |
|-|-|
| Encrypt VM disks | [Azure Disk Encryption](../../security/azure-security-disk-encryption-overview.md) helps you encrypt your Windows and Linux IaaS virtual machine disks. Encrypting disks make the contents unreadable without the necessary keys. Disk encryption protects stored data from unauthorized access that would otherwise be possible if the disk was copied|
| Limit installed software | Limiting installed software to what is required to successfully implement your solution |
| Use antivirus/Antimalware | In Azure you can use antimalware software from security vendors such as Microsoft, Symantec, Trend Micro, and Kaspersky. This software helps protect your virtual machines from malicious files, adware, and other threats. You can deploy Microsoft antimalware protection based on your application workloads, with either basic secure-by-default or advanced custom configuration. For more information on Microsoft Antimalware for Azure, see [Microsoft Antimalware for Azure Cloud Services and Virtual Machines](../../security/azure-security-antimalware.md) |
| Securely store keys and secrets | Simplifying the management of your secrets and keys by providing your application owners with a secure centrally managed option allows you to reduce the risk of accidental compromise or leaks. Azure Key Vault can securely store your keys in hardware security modules (HSMs) certified to FIPS 140-2 Level 2. If you need to store your keys and secrets using FIPs 140.2 Level 3, you can use [Azure Dedicated HSM](../../dedicated-hsm/overview.md) |
 

## Networking

| Recommendation | Comments |
|-|-|
| Restrict access to management ports | Attackers scan public cloud IP ranges for open management ports and attempt “easy” attacks like common passwords and known unpatched vulnerabilities. [Just-in-time (JIT) virtual machine (VM) access](../../security-center/security-center-just-in-time.md) can be used to lock down inbound traffic to your Azure VMs, reducing exposure to attacks while providing easy access to connect to VMs when needed. |
| Limit network access | Network security groups allow you to restrict network access and control the number of exposed endpoints. For more information, see [Create, change, or delete a network security group](../../virtual-network/manage-network-security-group.md) |

## Monitoring

| Recommendation | Comments |
|-|-|
| Monitor your virtual machines | You can use [Azure monitor for VMs](../../azure-monitor/insights/vminsights-overview.md) to monitor your Azure virtual machines and virtual machine scale sets. |

## Business continuity

| Recommendation | Comments |
|-|-|
| Back-up your virtual machines | [Azure Backup](../../backup/backup-overview.md) helps protect your application data with minimal operating costs. Application errors can corrupt your data, and human errors can introduce bugs into your applications. With Azure Backup, your virtual machines running Windows and Linux are protected. |
| Use multiple VMs for greater resilience and availability | If your VM runs applications that need to have high availability you should multiple VMs or [availability sets](manage-availability.md) |
| Adopt a business continuity and disaster recovery (BCDR) strategy | Azure Site Recovery allows you to choose from different options designed to support your business continuity needs. It supports different replication and fails over scenarios. For more information, see  [About Site Recovery](../../site-recovery/site-recovery-overview.md) |
