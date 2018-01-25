---
title: Deploying Azure Policy for Extensions | Microsoft Docs
description: Using Azure Policy to Restrict Extension Deployments
services: virtual-machines-linux 
documentationcenter: ''
author: danielsollondon 
manager: timlt 
editor: ''

ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 01/25/2018
ms.author: danis

---

# Deploying Extension Policy

If you want to prevent extensions, or certain extensions being deployed to your VMs, you can use ARM Policy to restrict VMs having specific or all extensions installed.

This can be scoped to a resource group.

For more information please refer to [Azure Policy](../azure-policy/azure-policy-introduction).

# Example CLI
Ensure you are using the latest Azure CLI.

## Create the policy definition
```azurecli-interactive
az policy definition create --name 'not-allowed-vmextension' --display-name 'Not allowed VM Extensions' --description 'This policy governs which VM extensions that are explicitly denied.' --rules 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/Compute/not-allowed-vmextension/azurepolicy.rules.json' --params 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/Compute/not-allowed-vmextension/azurepolicy.parameters.json' --mode All
```
The rules show how you can block 'Microsoft.Compute' extensions, for example, Windows Custom Script extension, or password resets.

## Assign the policy
Scope is to be set a resource group, it needs to be specified in this format, for example:
```text
/subscriptions/<subID>/resourceGroups/resourceGroupName
```
### Defining the extensions to block
Also note, you need to pass in a parameters file that has the AzureCompute extensions you wish to block. This is an example:

```json
{
    "notAllowedExtensions": {
        "value": [
            "VMAccessAgent",
            "CustomScriptExtension",
            "RunCommandWindows",
            "RunCommandLinux",
            "DSCForLinux",
            "CustomScript",
            "CustomScriptForLinux",
            "DSC"
        ]
    }
}
```
Save the above JSON to file:  'notAllowed.extensions.json'.

### Final assignment
```azurecli-interactive
az policy assignment create --name 'not-allowed-vmextension' --scope /subscriptions/qqqqqqq-yyyy-xxxx-tttt-wwwwwwwww/resourceGroups/myVMs --policy "not-allowed-vmextension" --params /Users/dansol/extensionPolicy/notAllowed.extensions.json
```
## Removing the assignment
```azurecli-interactive
az policy assignment delete --name 'not-allowed-vmextension' --resource-group myVMs
```
## Removing the policy
```azurecli-interactive
az policy definition delete --name 'not-allowed-vmextension'
```


# Example PowerShell
## Update to the latest version of AzureRM PSCmdLets
```powershell
Install-Module AzureRM
```
## Authenticate to Azure & Select Subscription
```powershell
Add-AzureRmAccount

Get-AzureRmSubscription

Set-AzureRmContext -SubscriptionId "<subid>"
```
### Double check this is in the right subscription, test listing some VMs

```powershell
$resourceGroupName = "myVMs"
Get-AzureRmVM -ResourceGroupName $resourceGroupName
```
## Create the policy definition
```powershell
$policyDefinitionName = "not-allowed-vmextension"
$definition = New-AzureRmPolicyDefinition -Name $policyDefinitionName -DisplayName "Not allowed VM Extensions" -description "This policy governs which VM extensions that are explicitly denied." -Policy 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/Compute/not-allowed-vmextension/azurepolicy.rules.json' -Parameter 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/Compute/not-allowed-vmextension/azurepolicy.parameters.json' -Mode All
$definition
```
## Assign the policy
Scope is to be set a resource group, it needs to be specified in this format, for example:
```text
/subscriptions/<subID>/resourceGroups/resourceGroupName 
```
### Defining the extensions to block
Also note, you need to pass in a parameters file that has the AzureCompute extensions you wish to block. This is an example:

```json
{
    "notAllowedExtensions": {
        "value": [
            "VMAccessAgent",
            "CustomScriptExtension",
            "RunCommandWindows",
            "RunCommandLinux",
            "DSCForLinux",
            "CustomScript",
            "CustomScriptForLinux",
            "DSC"
        ]
    }
}
```
Save the above JSON to file:  'notAllowed.extensions.json'.

### Final assignment
```powershell
$scope = "/subscriptions/qqqqqqq-yyyy-xxxx-tttt-wwwwwwwww/resourceGroups/myVMs"
$assignmentName = "not-allowed-vmextension"
$extensions = ".....\notAllowed.extensions.json"
$assignment = New-AzureRMPolicyAssignment -Name $assignmentName -Scope $scope -PolicyDefinition $definition -PolicyParameter $extensions
$assignment
```
## Removing the assignment
```powershell
Remove-AzureRMPolicyAssignment -Name $assignmentName -Scope $scope
```
## Removing the policy
```powershell
Remove-AzureRmPolicyDefinition -Name $policyDefinitionName 
```
# Testing
The fastest way to test, is to try to reset the password on a VM or test executing a script with the Windows Custom Script Extension, this should return an error very quickly.


## NOTES
Only one single hash