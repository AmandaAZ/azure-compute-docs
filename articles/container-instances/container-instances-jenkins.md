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

![Jenkins portal deployment cloud integration settings](./media/container-instances-jenkins/jenkins-portal-03.png)

## Configure Jenkins

Browse to the Jenkins Resource Group, select the Jenkins virtual machine, and take note of the DNS name.

![Jenkins login instructions](./media/container-instances-jenkins/jenkins-portal-fqdn.png)

Browser to the DNS name of the Jenkins VM and copy the returned SSH string.

![Jenkins login instructions](./media/container-instances-jenkins/jenkins-portal-04.png)

Open up a terminal session on your development system, and paste in the SSH string from the last step. Update the username to the username specified when deploying the Jenkins server.

Once connected, run the following command to retrieve the initial admin password.

```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Leave the SSH session and tunnel running, and navigate to http://localhost:8080 in your browser. Paste the initial admin password into the field as seen below. Select **Continue** when done.

![Unlock Jenkins](./media/container-instances-jenkins/jenkins-portal-05.png)

Select **Install suggested plugins** to install all recommended Jenkins plugins.

![Install Jenkins plugin](./media/container-instances-jenkins/jenkins-portal-06.png)

Create a new admin user account. This account is used for logging into and working with your Jenkins instance.

![Create Jenkins user](./media/container-instances-jenkins/jenkins-portal-07.png)

## Create build job

Jenkins is now configured and ready to build and deploy code. For this example, a simple Java application is used to demonstrate Jenkins builds on Azure Container Instances.

Select **Manage Jenkins** > **Configure System** > scroll down to the **Cloud** section. Update the Docker image to `microsoft/java-on-azure-jenkins-slave`. Once done, click **Apply** and then return to the Jenkins admin dashboard.

![Jenkins cloud configuration](./media/container-instances-jenkins/jenkins-aci-image.png)

Now create a Jenkins build job. Select **New Item**, give the build project a name such as `aci-java-demo`, and select **Freestyle Projet**. Click **OK** when done.

![Create Jenkins job](./media/container-instances-jenkins/jenkins-new-job.png)

Under **General**, ensure that Restrict where this project can be run is selected, and enter `linux` for the Label Expression. This configuration ensures that this build job runs on the ACI cloud.

![Create Jenkins job](./media/container-instances-jenkins/jenkins-job-01.png)

Under source code management, select `git` and enter `https://github.com/spring-projects/spring-petclinic.git` for the repository URL.

![Add source code to Jenkins job](./media/container-instances-jenkins/jenkins-job-02.png)

Under Build, add the build step named `Invoke top-level Maven targets`, and enter `package` as the goal. Select **Save** when done.

![Add Jenkins build step](./media/container-instances-jenkins/jenkins-job-03.png)

## Run the build job

Select **Build Now** to start a build job.

![Add Jenkins build step](./media/container-instances-jenkins/jenkins-job-status.png)

While the job is running, open up the Azure portal and look at the Jenkins resource group. You should see that an Azure Container Instance has been created. It is inside of this instance that the Jenkins job is running.

![Jenkins builds in ACI](./media/container-instances-jenkins/jenkins-aci.png)

As Jenkins beings to run more jobs than the configured number of executors (default 2), multiple Azure Container Instances are created.

![Jenkins builds in ACI](./media/container-instances-jenkins/jenkins-aci-multi.png)

Once all build jobs have been completed, the Azure Container Instances are removed.

![Jenkins builds in ACI](./media/container-instances-jenkins/jenkins-aci-none.png)

