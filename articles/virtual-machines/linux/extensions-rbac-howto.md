---
title: Use RBAC to Restrict Installation | Microsoft Docs
description: Example to show using Azure RBAC to restrict installing extensions on Azure.
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
ms.date: 02/05/2018
ms.author: danis

---

# How to use Azure RBAC to Restrict Extension Installation
In the scenario where you wish to have an Azure user with a VM monitoring role and restrict extension installation you can use a custom RBAC role.

## Create a Custom RBAC Role
Firstly you need to create the role metadata, the JSON example below allows a set of Actions and the 'NotActions' of restricting extension installation.


```json
{
    "Name": "Virtual Machine Maintainer",
    "IsCustom":true,
    "Description": "Can monitor, restart virtual machines, but not install extensions",
    "Actions":[
        "Microsoft.Storage/*/read",
        "Microsoft.Network/*/read",
        "Microsoft.Compute/*/read",
        "Microsoft.Compute/virtualMachines/start/action",
        "Microsoft.Compute/virtualMachines/restart/action",
        "Microsoft.Authorization/*/read",
        "Microsoft.Resources/subscriptions/resourceGroups/read",
        "Microsoft.Insights/alertRules/*",
        "Microsoft.Support/*"
    ],
    "NotActions":[
        "Microsoft.Compute/virtualMachines/extensions/write"
    ],
    "AssignableScopes":[
        "/subscriptions/e049fcf1-c84b-4de4-ba9a-a168a4cbab7a"
    ]

}
```
### Create The Role
```bash
az role definition create --role-definition @/Users/dansol/Data/extensions/howto/role.json
```
## Check Role Definition Exists
```bash
az role definition list --output table | grep Maintainer
```
## Assign the Role
```bash
az role assignment create --assignee <user, group, or service principal> --role "Virtual Machine Maintainer"
```
## Test
Try to install an extension. If using the portal, the extension 'Add' button is grayed out.

Using CLI:
```bash
az vm extension set \
  --resource-group operational \
  --vm-name pythonbottle \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{"commandToExecute": "echo hello "}'
```

Example of the error, after the RBAC assignment:
```text
The client 'xxxxxxx' with object id 'b000000-000c-000c-0000-d00000000a' does not have authorization to perform action 'Microsoft.Compute/virtualMachines/extensions/write' over scope '/subscriptions/oooooo-xxxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/operational/providers/Microsoft.Compute/virtualMachines/pythonbottle/extensions/customScript'.

```
## Next steps
This is just a small example on how to restrict extension provisioning, but you can define the scope and a whole host of more options, please refer to:

[Azure Policy Introduction](https://docs.microsoft.com/en-us/azure/azure-policy/azure-policy-introduction)

[Role Assignment](https://docs.microsoft.com/en-us/cli/azure/role/assignment?view=azure-cli-latest#az_role_assignment_create)
