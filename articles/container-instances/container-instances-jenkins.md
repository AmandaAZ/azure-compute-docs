---
title: Use Azure Container Instances as a Jenkins build server
description: Learn how to use Azure Container Instances as a Jenkins build server.
services: container-instances
author: neilpeterson
manager: timlt

ms.service: container-instances
ms.topic: article
ms.date: 04/20/2018
ms.author: nepeters
---

# Use Azure Container Instances as a Jenkins build server

## Deploy Jenkins server

In the Azure portal, select **Create a resource** and search for Jenkins. Select the Jenkins offering with a publisher of **Microsoft** and click **create**.

Enter the following information in the basics for and click *OK* when done.

- Name - name for the Jenkins deployment.
- User name - this user name is used as the admin use for the Jenkins virtual machine.
- Authentication type - SSH public key is recommended. If selected, copy in an SSH public key to be used when logging into the Jenkins server.
- Subscription - select an Azure subscription.
- Resource group - create a new or select an existing resource group.
- Location - select a location for the Jenkins server.

![Jenkins portal deployment basic settings](./media/container-instances-jenkins/jenkins-portal-01.png)

On the additional settings form, complete the following items:

- Size - Select the appropriate sizing option for your Jenkins virtual machine.
- VM disk type - Specify either HDD (hard-disk drive) or SSD (solid-state drive) for the Jenkins server.
- Virtual network - (Optional) Select Virtual network to modify the default settings.
- Subnets - Select Subnets, verify the information, and select OK.
- Public IP address - Selecting the Public IP address allows you to give it a custom name, configure SKU, and assignment method.
- Domain name label - Specify the value for the fully qualified URL to the Jenkins virtual machine.
- Jenkins release type - Select the desired release type from the options: LTS, Weekly build, or Azure Verified.

![Jenkins portal deployment additional settings](./media/container-instances-jenkins/jenkins-portal-02.png)

For service principal integration, select Auto(MSI) to have Azure Managed Service Identity create auto create an authentication identity. Select Manual to provider your own service principal credentials.

Cloud agents you to configure a build agent platform for Jenkins build jobs.

- ACI: Each build job is run in an Azure Container Instance. For more information on using ACI as a build platform, see.
- VM: Each build job is run in an Azure Container Instance.
- None: Each build job is run on the Jenkins instance itself. This configuration is not recommended.

Once done with the integration settings, click ok, and then OK again on the validation summary. The Jenkins server takes a few minutes to deploy.

## Configure Jenkins

![Jenkins portal deployment cloud integration settings](./media/container-instances-jenkins/jenkins-portal-03.png)

![Jenkins login instructions](./media/container-instances-jenkins/jenkins-portal-04.png)

![Unlock Jenkins](./media/container-instances-jenkins/jenkins-portal-05.png)

![Install Jenkins plugin](./media/container-instances-jenkins/jenkins-portal-06.png)

![Create Jenkins user](./media/container-instances-jenkins/jenkins-portal-07.png)

## Create build job

![Create Jenkins job](./media/container-instances-jenkins/jenkins-job-01.png)

![Add source code to Jenkins job](./media/container-instances-jenkins/jenkins-job-02.png)

![Add Jenkins build step](./media/container-instances-jenkins/jenkins-job-03.png)


