---
title: SAP deployment automation framework DevOps hands-on lab
description: DevOps Hands-on lab for the SAP Deployment Automation Framework on Azure
author: mimergel
ms.author: mimergel
ms.reviewer: kimforss
ms.date: 12/14/2021
ms.topic: tutorial
ms.service: virtual-machines-sap
---

# SAP Deployment Automation Framework DevOps - Hands-on lab

This tutorial shows how to perform the deployment activities of the [SAP deployment automation framework on Azure](automation-deployment-framework.md) using Azure DevOps Services.

You will perform the following tasks during this lab:

> [!div class="checklist"]
> * Deploy the Control Plane (Deployer Infrastructure & Library)
> * Deploy the Workload Zone (Landscape, System)
> * Deploy the SAP Infrastructure
> * Install HANA Database
> * Install SCS server
> * Load HANA Database
> * Install Primary Application Server
> * Download the SAP software
> * Install SAP

## Prerequisites

1. An Azure subscription. If you don't have an Azure subscription, you can [create a free account here](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

1. A configured Azure DevOps instance, follow the steps here [Configure Azure DevOps Services for SAP Deployment Automation](automation-configure-devops.md)

1. A configured self hosted agent, see [Configure a self-hosted agent for SAP Deployment Automation](automation-configure-devops.md#register-the-deployer-as-an-self-hosted-agent-for-azure-devops)

1. SAP software, see [download of the SAP software](automation-software.md) in your Azure environment.

> [!Note]
> The free Azure account may not be sufficient to run the deployment

## Overview

These steps reference and use the [default naming convention](automation-naming.md) for the automation framework. Example values are also used for naming throughout the configurations. In this tutorial, the following names are used:
- Azure DevOps project name is `SAP-Deployment` 
- Azure DevOps repository name is `sap-automation` 
- The control plane environment is named `MGMT`, in the region West Europe (`WEEU`) and installed in the virtual network `DEP00`, giving a deployer configuration name: `MGMT-WEEU-DEP00-INFRASTRUCTURE`

- The SAP workload zone has the environment name `DEV` and is in the same region as the control plane using the virtual network `SAP01`, leading to the SAP workload zone configuration name: `DEV-WEEU-SAP01-INFRASTRUCTURE`
- The SAP System with SID `X00` will be installed in this SAP workload zone. This leads to the configuration name for the SAP System: `DEV-WEEU-SAP01-X00`

| Artifact type | Configuration name              | Location        |
| ------------- | ------------------------------- | --------------- |
| Control Plane | MGMT-WEEU-DEP00-INFRASTRUCTURE  | westeurope      |
| Workload Zone | DEP-WEEU-SAP01-INFRASTRUCTURE   | westeurope      |
| SAP System    | DEP-WEEU-SAP01-X00              | westeurope      |

The deployed infrastructure is shown in the diagram below.

 :::image type="content" source="media/automation-devops/automation-devops-tutorial-design.png" alt-text="DevOps tutorial infrastructure design":::


> [!Note]
> In this tutorial the X00 SAP system will be deployed with the following configuration:
> * Standalone deployment
> * HANA DB VM SKU: Standard_M32ts
> * ASCS VM SKU: Standard_D4s_v3
> * APP VM SKU: Standard_D4s_v3

## Deploying the Control Plane

The deployment will use the configuration defined in the Terraform variable files located in the 'samples/WORKSPACES/DEPLOYER/MGMT-WEEU-DEP00-INFRASTRUCTURE' and 'samples/WORKSPACES/LIBRARY/MGMT-WEEU-SAP_LIBRARY' folders. 

Update the 'Deployment_Configuration_Path' variable in the 'SDAF-General' variable group to 'samples/WORKSPACES'

Run the pipeline by navigating to the Pipelines section in the [Azure DevOps Portal](https://dev.azure.com) and selecting the _Deploy control plane_ pipeline. Run the pipeline and when prompted enter the following values:

Deployer configuration name: MGMT-WEEU-DEP00-INFRASTRUCTURE
SAP Library configuration name: MGMT-WEEU-SAP_LIBRARY

:::image type="content" source="media/automation-devops/automation-run-pipeline.png" alt-text="DevOps tutorial, run pipeline dialog":::

You can track the progress in the Azure DevOps portal. Once the deployment is complete you can see the Control Plane details in the _Extensions_ tab.

 :::image type="content" source="media/automation-devops/automation-run-pipeline-control-plane.png" alt-text="DevOps tutorial, run pipeline results":::

## Deploying the Workload zone

The deployment will use the configuration defined in the Terraform variable file located in the 'samples/WORKSPACES/LANDSCAPE/DEV-WEEU-SAP01-INFRASTRUCTURE' folder.

Run the pipeline by navigating to the Pipelines section in the [Azure DevOps Portal](https://dev.azure.com) and selecting the _Deploy workload zone_ pipeline. Run the pipeline and when prompted enter the following values:

Workload zone configuration name: DEV-WEEU-SAP01-INFRASTRUCTURE
Deployer Environment Name: MGMT

You can track the progress in the Azure DevOps portal. Once the deployment is complete you can see the Workload Zone details in the _Extensions_ tab.

## Deploying the SAP System

The deployment will use the configuration defined in the Terraform variable file located in the 'samples/WORKSPACES/SYSTEM/DEV-WEEU-SAP01-X00' folder.

Run the pipeline by navigating to the Pipelines section in the [Azure DevOps Portal](https://dev.azure.com) and selecting the _SAP system deployment_ pipeline. Run the pipeline and when prompted enter the following values:

SAP System configuration name: DEV-WEEU-SAP01-X00

You can track the progress in the Azure DevOps portal. Once the deployment is complete you can see the SAP System details in the _Extensions_ tab.

## Downloading the SAP Software

Run the pipeline by navigating to the Pipelines section in the [Azure DevOps Portal](https://dev.azure.com) and selecting the _SAP software acquisition_ pipeline. Run the pipeline and when prompted enter the following values:

Name of Bill of Materials (BoM): S41909SPS03_v0010ms
Control Plane Environment name: MGMT
Control Plane (SAP Library) location code: WEEU

You can track the progress in the Azure DevOps portal. 

## Running the Configuration and SAP Installation pipeline

Run the pipeline by navigating to the Pipelines section in the [Azure DevOps Portal](https://dev.azure.com) and selecting the _Configuration and SAP installation_ pipeline. Run the pipeline and when prompted enter the following values:

SAP System configuration name: DEV-WEEU-SAP01-X00
Bill of Materials name: S41909SPS03_v0010ms

Choose the playbooks to execute.

:::image type="content" source="media/automation-devops/automation-os-sap.png" alt-text="DevOps tutorial, OS and SAP configuration":::

You can track the progress in the Azure DevOps portal. 

## Running the Repository update pipeline

Run the pipeline by navigating to the Pipelines section in the [Azure DevOps Portal](https://dev.azure.com) and selecting the _Repository updater_ pipeline. Run the pipeline and when prompted enter the following values:

Source repository: https://github.com/Azure/sap-automation.git
Source branch to update from: main

Only choose 'Force the update' if the update fails.


## Running the removal pipeline

Run the pipeline by navigating to the Pipelines section in the [Azure DevOps Portal](https://dev.azure.com) and selecting the _Deployment removal _ pipeline. Run the pipeline and select which deployment to remove.

### SAP System removal

When prompted enter the following values:

SAP System configuration name: DEV-WEEU-SAP01-X00

### SAP Workload Zone removal

When prompted enter the following values:

SAP workload zone configuration name: DEV-WEEU-SAP01-INFRASTRUCTURE

### Control Plane removal

When prompted enter the following values:

Deployer configuration name: MGMT-WEEU-DEP00-INFRASTRUCTURE
SAP Library configuration name: MGMT-WEEU-SAP_LIBRARY
## Next step

> [!div class="nextstepaction"]
> [Configure Control Plane](automation-configure-control-plane.md)




