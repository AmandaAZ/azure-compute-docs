---
title: Configure a Deployer UX Web Application for SAP Deployment Automation Framework
description: Configure a web app as a part of the control plane to assist in creating and deploying SAP workload zones and systems on Azure.
author: wsheehan
ms.author: wsheehan
ms.reviewer: wsheehan
ms.date: 06/21/2022
ms.topic: conceptual
ms.service: virtual-machines-sap
---

# Configure the Control Plane UX Web Application

As a part of the SAP automation framework control plane, you can optionally create an interactive web application that will assist you in creating and deploying SAP workload zones and systems.

:::image type="content" source="./media/automation-deployment-framework/webapp-front-page.png" alt-text="Web app front page":::

## Create an app registration 

If you would like to use the web app, you must first create an app registration for authentication purposes. Open the Azure cloud shell and execute the following commands:

# [Linux](#tab/linux)
Replace MGMT with your environment as necessary.
```bash
echo '[{"resourceAppId":"00000003-0000-0000-c000-000000000000","resourceAccess":[{"id":"e1fe6dd8-ba31-4d61-89e7-88639da4683d","type":"Scope"}]}]' >> manifest.json 

TF_VAR_app_registration_app_id=$(az ad app create \
    --display-name MGMT-webapp-registration \
    --available-to-other-tenants false \
    --required-resource-access @manifest.json \
    --query "appId" | tr -d '"')

TF_VAR_webapp_client_secret=$(az ad app credential reset \
    --id $TF_VAR_app_registration_app_id --append               \
    --query "password" | tr -d '"')

rm manifest.json
```
# [Windows](#tab/windows)
Replace MGMT with your environment as necessary.
```powershell
Add-Content -Path manifest.json -Value '[{"resourceAppId":"00000003-0000-0000-c000-000000000000","resourceAccess":[{"id":"e1fe6dd8-ba31-4d61-89e7-88639da4683d","type":"Scope"}]}]'

$TF_VAR_app_registration_app_id=(az ad app create `
    --display-name $region_code-webapp-registration     `
    --required-resource-accesses ./manifest.json        `
    --query "appId").Replace('"',"")

$TF_VAR_webapp_client_secret=(az ad app credential reset `
    --id $env:TF_VAR_app_registration_app_id --append            `
    --query "password").Replace('"',"") 

rm ./manifest.json
```
---

## Deploy via Azure Devops (pipelines)

For full instructions on setting up the web app using Azure Devops, see [Use SAP Deployment Automation Framework from Azure DevOps Services](https://review.docs.microsoft.com/en-us/azure/virtual-machines/workloads/sap/automation-configure-devops?branch=main)

### Summary of additional steps required to set up the web app before deploying the control plane:
1. Add the web app deployment pipeline (deploy/pipelines/21-deploy-web-app.yaml).
2. Add the variables TF_VAR_app_registration_app_id and TF_VAR_webapp_client_secret to your environment specific variable group before deployment.
3. Assign the administrator role to the build service using the Security tab in your environment specific variable group.
4. Check the box next to "deploy the web app infrastructure" when running the deploy control plane pipeline.

### Summary of steps required to access the web app after deploying the control plane:
1. Update the app registration reply URLs.
2. Run the webb app deployment pipeline you set up in step 1 above.
3. (Optionally) add an additional access policy to the app service.

## Deploy via Azure CLI (cloudshell)

For full instructions on setting up the web app using the Azure CLI, see [Deploy the control plane](https://review.docs.microsoft.com/en-us/azure/virtual-machines/workloads/sap/automation-deploy-control-plane?branch=main&tabs=linux)

### Summary of additional steps required to set up the web app before deploying the control plane:
1. Export the environment variables TF_VAR_app_registration_app_id, TF_VAR_webapp_client_secret, and TF_VAR_use_webapp="true".

### Summary of steps required to access the web app after deploying the control plane:
1. Update the app registration reply URLs.
2. Generate a zip file of the web app code.
3. Deploy the software to the app service.
4. Configure the application settings.
5. (Optionally) add an additional access policy to the app service.


## Using the web app

The web app allows you to create SAP workload zone (landscape) objects and system infrastructure objects. These are essentially another representation of a configuration file.
In the case of deploying using Azure Devops, you have ability to deploy these workload zones and system infrastructures right from the web app.
In the case of deploying using the Azure CLI, you can download the parameter file for any landscape or system object you create, and use that in your command line deployments.

### Creating a landscape or system object from scratch
1. Navigate to the "Landscape" or "System" tab at the top of the website.
2. Click "Create New" in the bottom left corner.
3. Fill out the required parameters in the "Basic" and "Advanced" tabs, and any additional parameters you desire.
4. Certain parameters will be dropdowns populated with existing azure resources.
   * If no results are shown for a dropdown, you probably need to specify another dropdown before you can see any options.
       - The subscription parameter must be specified before any other dropdown functionality is enabled
       - The network_arm_id parameter must be specified before any subnet dropdown functionality is enabled
5. Click submit in the bottom left hand corner

### Creating a landscape or system object from a file
1. Navigate to the "File" tab at the top of the website.
2. Your options are
   * Create a new file from scratch there in browser. It should be in the .tfvars file format. Click save.
   * Import an existing .tfvars file, and (optionally) edit it before saving.
   * Use an existing template, and (optionally) edit it before saving.
3. Make sure your file conforms to the correct naming conventions.
4. Next to the file you would like to convert to a landscape or system object, click "Convert".
5. The landscape or system object will appear in its respective tab.

### Deploying a landscape or system object (Azure Devops deployment)
1. Navigate to the Landscape or System tab.
2. Next to the landscape or system you would like to deploy, click "Deploy".
   * If you would like to deploy a file, first convert it to a landscape or system object.
4. Specify the necessary parameters, and confirm it is the correct object.
5. Click deploy.
6. The web app will automatically generate a .tfvars file from the object, update your Devops repository, and kick off the workload zone or system (infrastructure) pipeline. Monitor the deployment back in Azure Devops.
