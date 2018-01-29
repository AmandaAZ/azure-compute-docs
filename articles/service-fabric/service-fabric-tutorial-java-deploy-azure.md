---
title: Deploy a Service Fabric application to a cluster in Azure | Microsoft Docs
description: Learn how to deploy an application to an Azure Service Fabric cluster.
services: service-fabric
documentationcenter: java
author: suhuruli
manager: msfussell
editor: ''

ms.assetid: 
ms.service: service-fabric
ms.devlang: java
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 01/23/2018
ms.author: suhuruli
ms.custom: mvc

---

# Deploy an application to a Service Fabric cluster in Azure
This tutorial is part three of a series and shows you how to deploy a Service Fabric application to a cluster in Azure.

In this tutorial series you learn how to:
> [!div class="checklist"]
> * [Create a Linux cluster](service-fabric-tutorial-create-dotnet-app.md)
> * Deploy and manage an application to a remote cluster
> * [Upgrade your application](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)
> * [Observe a fail over of the application](service-fabric-tutorial-monitoring-aspnet.md)

## Prerequisites
Before you begin this tutorial:
- If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- [Install Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- Install the Service Fabric SDK ([Mac](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-get-started-mac) , [Linux](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-get-started-linux))
- Install Python 3 

## Download the Voting sample application
If you did not build the Voting sample application in [part one of this tutorial series](service-fabric-tutorial-create-java-app.md), you can download it. In a command window, run the following command to clone the sample app repository to your local machine.

```bash
git clone https://github.com/Azure-Samples/service-fabric-java-quickstart
```

## Create a Service Fabric cluster in Azure

The steps below will create the Service Fabric cluster and necessary resources required to deploy your application a Service Fabric cluster and monitor the health of your solution using the ELK (Elasticache, Logstash, Kibana) stack. This will be useful in the upcoming parts of this tutorial. If you are simply interested in seeing your Java application running in Azure, please go to the [Set up a party cluster](service-fabric-tutorial-java-deploy-azure#Set up a Party Cluster). In addition to creating a Service Fabric cluster, [Event Hubs](https://azure.microsoft.com/en-us/services/event-hubs/) will be provisioned to pipe logs from the Service Fabric cluster to your Logstash instance. 

1. Download the following pckage which contains necessary helper scripts and the templates to create the resources in Azure. 

```bash
git clone https://github.com/Azure-Samples/service-fabric-java-quickstart.git
```

2. Log in to your Azure account from Terminal 
```bash
az login
```
3. Set your Azure subscription that you want to use to create the resources 
```bash
az account set --subscription [SUBSCRIPTION-ID]
``` 
4. Run the following command to create a cluster certificate in Key Vault that will be used to secure your Service Fabric cluster. You will have to provide the region (must be the same as your Service Fabric cluster), key vault resource group name, key vault name, certificate password and cluster dns name. 
```bash
./new-service-fabric-cluster-certificate.sh [REGION] [KEY-VAULT-RESOURCE-GROUP] [KEY-VAULT-NAME] [CERTIFICATE-PASSWORD] [CLUSTER-DNS-NAME-FOR-CERTIFICATE]

Example: ./new-service-fabric-cluster-certificate.sh 'westus' 'testkeyvaultrg' 'testkeyvault' '<password>' 'testservicefabric.westus.cloudapp.azure.com'
```

The above command will return the following information which should be noted for use later,

```
Source Vault Resource Id: /subscriptions/<subscription_id>/resourceGroups/testkeyvaultrg/providers/Microsoft.KeyVault/vaults/<name>
Certificate URL: https://<name>.vault.azure.net/secrets/<cluster-dns-name-for-certificate>/<guid>
Certificate Thumbprint: <THUMBPRINT>
```
5. Create a resource group for the storage account which will store your logs
``bash
az group create --location [REGION] --name [RESOURCE-GROUP-NAME]

Example: az group create --location westus --name teststorageaccountrg
```
6. Create a storage account which will be used to store the logs that will be 
```bash
az storage account create -g [RESOURCE-GROUP-NAME] -l [REGION] --name [STORAGE-ACCOUNT-NAME] --kind StorageV2

Example: az storage account create -g teststorageaccountrg -l westus --name teststorageaccount --kind StorageV2
```
6. Access the [Azure portal](portal.azure.com) and navigate to the **Shared Access Signature** tab for your Storage account.
**Insert Picture**
7. Copy the Table service SAS URL and set it aside for use when creating your Service Fabric cluster. It will resemble the URL below. 
```
https://<storage_account_name>.table.core.windows.net/?sv=2017-04-17&ss=bfqt&srt=sco&sp=rwdlacup&se=2018-01-25T05:09:51Z&st=2018-01-24T21:09:51Z&spr=https&sig=U0cPmJWZqoEKS4%2FFaaFP%2B8ggAy7Nlxx2qkQZhmKUFPA%3D
```

8. Create a resource group that will contain the Event Hub resources. 
```bash
az group create --location [REGION] --name [RESOURCE-GROUP-NAME]

Example: az group create --location westus --name testeventhubsrg
```

8. Create an Event Hubs resource in a separate resource group using the command below,

```bash
az group deployment create -g [RESOURCE-GROUP-NAME] --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-event-hubs-create-event-hub-and-consumer-group/azuredeploy.json --parameters namespaceName=[EVENT-HUB-NAMESPACE] eventHubName=[EVENT-HUB-NAME] consumerGroupName=[EVENT-HUB-CONSUMER-GROUP]

Example: az group deployment create -g testeventhubsrg --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-event-hubs-create-event-hub-and-consumer-group/azuredeploy.json --parameters namespaceName=testeventhubnamespace eventHubName=testeventhubs consumerGroupName=testeventhubconsumergroup

```

9. Access the Azure Portal and navigate to the **Shared access policy** tab for your Event Hubs resource created. 
**InsertPicture**

10. Copy the name of the SAS policy and the Primary Key for use in the next command. 

11. Run the python script provided in the ```service-fabric-java-quickstart/AzureCluster``` directory to create a SAS token for your Event Hubs resource. The script will return the SAS URL for the Event Hubs resource which will be used in the next step. 

```python
python3 eventhubssastoken.py 'testeventhubs' 'testeventhubs' '<name_of_SAS_policy>' '<primary_key>'
```

12. Open the ```azuredeploy.parameters.json``` file and replace the following contents from the steps above 
```
"applicationDiagnosticsStorageAccountName": {
    "value": "teststorageaccount"
},
"applicationDiagnosticsStorageAccountSasToken": {
    "value": "<sas_url_from_step_7"
},
"loggingEventHubSAS": {
    "value": "<sas_from_step_11"
}
```
13. Run the following command to create your Service Fabric cluster
```bash
az sf cluster create --location 'westus' --resource-group 'testlinux' --template-file azuredeploy.json --parameter-file azuredeploy.parameters.json --secret-identifier <certificate_url_from_step4>
```

### Deploy your application to the cluster
1. Before deploying your application, you will need to add the following snippet to the ```Voting/VotingApplication/ApplicationManifest.xml``` file. The **X509FindValue** field is the thumbprint returned from Step 4 of the **Create a Service Fabric cluster in Azure** section. This snippet is nested under the **ApplicationManifest** field (the root field). 
```xml
<Certificates>
      <SecretsCertificate X509FindType="FindByThumbprint" X509FindValue="<thumbprint_from_step_4>" />
</Certificates>
```
2. To deploy your application to this cluster, you must use SFCTL to establish a connection to the cluster. SFCTL requires a PEM file with both the public and private key to connect to the cluster and as as result, run the following command to produce a PEM file with both the public and private key. 
```bash
openssl pkcs12 -in testservicefabric.westus.cloudapp.azure.com.pfx -out sfctlconnection.pem -nodes -passin pass:<password>
```
3. Run the following command to connect to the cluster
```bash
sfctl cluster select --endpoint https://testlinuxcluster.westus.cloudapp.azure.com:19080 --pem sfctlconnection.pem --no-verify
```
4. To deploy your application, navigate to the ```Voting/Scripts``` folder and run the **install.sh** script. 
```bash
./install.sh
```
5. Open your favorite browser and type in https://testlinuxcluster.westus.cloudapp.azure.com:19080 to access Service Fabric Explorer. You will have to choose the certificate from the certificate store that you want to use to connect to this endpoint. If you are using a Linux machine, the certificates that were generated by the ```new-service-fabric-cluster-certificate.sh``` script will have to be imported into Chrome to view Service Fabric Explorer. If you are using a Mac, you will have to install the PFX file into your Keychain. You will notice your application has been installed on the cluster. 

**Insert Picture**

6. To access your application, type in https://testlinuxcluster.westus.cloudapp.azure.com:19080 
**Insert Picture**


7. To uninstall your application from the cluster, run the ```uninstall.sh``` script in the **Scripts** folder 
```bash
./uninstall.sh
```

## Set up and deploy to a Party Cluster

This is an optional section for those who want to get a free, quick trial of a secure Service Fabric cluster in Azure. 
Party clusters are free, limited-time Service Fabric clusters hosted on Azure and run by the Service Fabric team where anyone can deploy applications and learn about the platform. For free!

To get access to a Party Cluster, browse to this site: http://aka.ms/tryservicefabric and follow the instructions in the **ReadMe** heading to get access to the cluster. You need a Facebook or GitHub account to get access to a Party Cluster.

1. Sign in to the party cluster site with your Github or Facebook and choose Linux Cluster
2. Download the PFX file on your machine and make note of the endpoint of the cluster. It should resemble the following: 
```
https://zlnx2us7j1vb.westus.cloudapp.azure.com:19080
```
3. Run the following command in your Terminal to convert the PFX file to a PEM file. 
```bash 
openssl pkcs12 -in party-cluster-1910992993-client-cert.pfx -out party-cluster-client-cert.pem -nodes -passin pass:
```
4. Navigate to the **<path_to_your_eclipse_workspace>/Voting/Scripts** directory and run the following command to connect to the party cluster 
```bash
sfctl cluster select --endpoint https://zlnx2us7j1vb.westus.cloudapp.azure.com:19080 --pem <path_to_pem_from_step_3> --no-verify
```
5. Run the ```install.sh``` script to deploy your application. 
```bash
./install.sh
```
6. Open you preferred browser and type in the cluster address along with the port the application is listening on (for example - https://zlnx2us7j1vb.westus.cloudapp.azure.com:8080). You should see the same result as when the application was running locally. 

7. To uninstall your application from the cluster, run the ```uninstall.sh``` script in the **Scripts** folder 
```bash
./uninstall.sh
```

**Insert Picture**

## Next steps
In this tutorial, you learned how to:

> [!div class="checklist"]
> * Create a cluster in Azure fully set up to host your Java application
> * Create and deploy to a Party Cluster

Advance to the next tutorial:
> [!div class="nextstepaction"]
> [Set up Monitoring & Diagnostics](service-fabric-tutorial-java-monitor-diagnostics.md)