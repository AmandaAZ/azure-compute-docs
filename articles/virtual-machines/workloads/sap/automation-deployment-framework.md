---
title: About SAP deployment automation framework on Azure
description: Overview of the framework and tooling for automated deployment for SAP on Azure.
author: kimforss
ms.author: kimforss
ms.reviewer: kimforss
ms.date: 10/13/2021
ms.service: virtual-machines-sap
ms.topic: conceptual
---
# SAP deployment automation framework on Azure

The [SAP on Azure Deployment Automation Framework](https://github.com/Azure/sap-hana) is an open-source solution that can be used to deploy SAP Infrastructures and install SAP applications in Azure. You can create infrastructure for SAP landscapes based on SAP HANA and NetWeaver with AnyDB on any of the SAP-supported operating system versions and deploy them into any Azure region. 

The [SAP on Azure Deployment Automation Framework](https://github.com/Azure/sap-hana) has two main components:
-	Deployment infrastructure (control plane) 
-	SAP Infrastructure (SAP Workload)

The dependency between the control plane and the application plane is illustrated in the diagram below

:::image type="content" source="./media/automation-deployment-framework/control-plane-sap-infrastructure.png" alt-text="Diagram of dependency between the Control Plane and Application plane.":::


The framework uses [Terraform](https://www.terraform.io/) for infrastructure deployment, and [Ansible](https://www.ansible.com/) for the operating system and application configuration.

> [!NOTE]
> This automation framework is based on Microsoft best practices and principles for SAP on Azure. Review the [get-started guide for SAP on Azure virtual machines (Azure VMs)](get-started.md) to understand how to use certified virtual machines and storage solutions for stability, reliability, and performance.
> 
> This automation framework also follows the [Microsoft Cloud Adoption Framework for Azure](/azure/cloud-adoption-framework/).

You will use the control plane of the SAP Deployment Automation Framework to deploy the SAP Infrastructure and the SAP application infrastructure. The deployment uses Terraform templates to create the [infrastructure as a service (IaaS)](https://azure.microsoft.com/overview/what-is-iaas) defined infrastructure to host the SAP Applications.

The application configuration will be performed from the Ansible Controller in the Control plane using a set of pre-defined playbooks. These playbooks will:

- Configure base operating system settings
- Configure SAP-specific operating system settings
- Make the installation media available in the system
- Install the SAP system
- Install the SAP database (SAP HANA, AnyDB)
- Configure high availability (HA) using Pacemaker
- Configure high availability (HA) for your SAP database


## About the control plane

The control plane provides the following services
-	Terraform Deployment Infrastructure
-	Ansible Controller
-	Persistent storage for the Terraform state files
-	Persistent storage for the Downloaded SAP Software
-	Secure storage for deployment credentials
-	Private DNS zone (optional)

The control plane is typically a regional resource deployed in to the hub subscription in a hub and spoke architecture. 

The key components of the control plane are
- Deployment virtual machine 
- Storage account for Terraform state files
- Storage account for SAP installation media
- Keyvault for deployment credentials


## About the SAP Workload

The SAP Workload contains all the Azure infrastructure resources for the SAP Deployments. These resources are deployed from the control plane. 
The SAP Workload has two main components:
-	SAP Workload Zone
-	SAP System

### About the SAP Workload Zone

The SAP Workload Zone provides the following services to the SAP Systems
-	Virtual Networking infrastructure
-	Secure storage for system credentials (Virtual Machines and SAP)
-	Shared Storage (optional)


### About the SAP System

The SAP System provides the following services
-	Virtual machine, storage, and supporting infrastructure to host the SAP application

## Glossary

The following terms are important concepts for understanding the automation framework.

### SAP concepts

| Term | Description |
| ---- | ----------- |
| System | An instance of an SAP application that contains the resources the application needs to run. Defined by a unique three-letter identifier, the **SID**.
| Landscape | A collection of systems in different environments within an SAP application. For example, SAP ERP Central Component (ECC), SAP customer relationship management (CRM), and SAP Business Warehouse (BW). |
| Workload zone | Partitions the SAP applications to environments, such as non-production and production environments or development, quality assurance, and production environments. Provides shared resources, such as virtual networks and key vault, to all systems within. |

The following diagram shows the relationships between SAP systems, workload zones (environments), and landscapes. In this example setup, the customer has three SAP landscapes: ECC, CRM, and BW. Each landscape contains three workload zones: production, quality assurance, and development. Each workload zone contains one or more systems.

:::image type="content" source="./media/automation-deployment-framework/sap-terms.png" alt-text="Diagram of SAP configuration with landscapes, workflow zones, and systems.":::

### Deployment components

| Term | Description | Scope |
| ---- | ----------- | ----- |
| Deployer | A virtual machine that can execute Terraform and Ansible commands. Deployed to a virtual network, either new or existing, that is peered to the SAP virtual network. | Region |
| Library | Provides storage for the Terraform state files and SAP installation media. | Region |
| Workload zone | Contains the virtual network into which you deploy the SAP system or systems. Also contains a key vault that holds the credentials for the systems in the environment. | Workload zone |
| System | The deployment unit for the SAP application (SID). Contains virtual machines and supporting infrastructure artifacts, such as load balancers and availability sets. | Workload zone |

## Abbreviations in code

The following terms are common acronyms and abbreviations used in the automation framework's code.

| Term | Description |
| ---- | ----------- |
| ALB | Azure Load Balancer |
| AVSET | Azure Availability Set |
| B&D | Build and destroy. An alternate term for fall forward. |
| DR | Disaster recovery |
| Fall forward | See description for B&D |
| HA | High availability |
| NIC | Network interface component |
| NSG | Network security group |
| SA | Stand-alone instances, which aren't high availability. |
| SDU | SAP deployment unit. The SAP system deployment. |
| UDR | User-defined route |
| VM | Virtual machine |
| VNET | Virtual network |


## Next steps

> [!div class="nextstepaction"]
> [Get started with the automation framework](automation-get-started.md)
