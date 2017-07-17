---
title: Create and Manage an Azure Virtual Machine Using Java | Microsoft Docs
description: Use Java and Azure Resource Manager to deploy a virtual machine and all its supporting resources.
services: virtual-machines-windows
documentationcenter: ''
author: davidmu1
manager: timlt
editor: tysonn
tags: azure-resource-manager

ms.assetid: 
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 07/17/2017
ms.author: davidmu

---
# Create and manage Windows VMs in Azure using Java

An [Azure Virtual Machine](overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (VM) needs several supporting Azure resources. This article covers creating, managing, and deleting VM resources using Java. You learn how to:

> [!div class="checklist"]
> * Create a Maven project
> * Add dependencies
> * Create credentials
> * Create resources
> * Perform management tasks
> * Delete resources
> * Run the application

It takes about 20 minutes to do these steps.

## Create a Maven project

1. If you haven't already done so, install [Java](http://www.oracle.com/technetwork/java/javase/downloads/index.html).
2. Install [Maven](http://maven.apache.org/download.cgi).
3. Create a new folder for the project.
    
    ```
    mkdir java-azure-test
    cd java-azure-test
    ```
4. Create the Maven project.

   ```
   mvn archetype:generate -DgroupId=com.myResourceGroup -DartifactId=createVM  \ 
     -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
   ```

## Add dependencies

To set the dependencies that you need in Maven, do these steps:

1. Change the directory to *createVM*.

2. Open the pom.xml file and add the build coonfiguration to <project> to enable the bulding of your application:

    ```xml
    <build>
      <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <configuration>
                <mainClass>com.fabrikam.testAzureApp.AzureApp</mainClass>
            </configuration>
        </plugin>
      </plugins>
    </build>
    ```

2. Add the dependencies that are needed to access the Azure Java SDK.

    ```xml
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-mgmt-compute</artifactId>
      <version>1.1.0</version>
    </dependency>
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-mgmt-resources</artifactId>
      <version>1.1.0</version>
    </dependency>
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-mgmt-network</artifactId>
      <version>1.1.0</version>
    </dependency>
    ```

3. Save the pom.xml file.

## Create credentials

Before you start this step, make sure that you have access to an [Active Directory service principal](../../azure-resource-manager/resource-group-create-service-principal-portal.md). You should also record the application ID, the authentication key, and the tenant ID that you need in a later step.

### create the authorization file

1. Create a file named *azureauth.properties* and add these properties to it:

    ```
    subscription=<subscription-id>
    client=<application-id>
    key=<authentication-key>
    tenant=<tenant-id>
    managementURI=https://management.core.windows.net/
    baseURL=https://management.azure.com/
    authURL=https://login.windows.net/
    graphURL=https://graph.windows.net/
    ```

    Replace **&lt;subscription-id&gt;** with your subscription identifier, **&lt;application-id&gt;** with the Active Directory application identifier, **&lt;authentication-key&gt;** with the application key, and **&lt;tenant-id&gt;** with the tenant identifier.

2. Save the azureauth.properties file.
3. Set an environment variable AZURE_AUTH_LOCATION with the full path to the authentication file in your shell. 

    ```
    export AZURE_AUTH_LOCATION=/Users/raisa/azureauth.properties
    ```

### create the management client

1. Open the App.java file that was created in the project, and then add these import statements to the top of the file:
   
    ```java
    import com.microsoft.azure.management.Azure;
    import com.microsoft.azure.management.compute.AvailabilitySet;
    import com.microsoft.azure.management.compute.AvailabilitySetSkuTypes;
    import com.microsoft.azure.management.compute.InstanceViewStatus;
    import com.microsoft.azure.management.compute.DiskInstanceView;
    import com.microsoft.azure.management.compute.VirtualMachine;
    import com.microsoft.azure.management.network.PublicIPAddress;
    import com.microsoft.azure.management.network.Network;
    import com.microsoft.azure.management.network.NetworkInterface;
    import com.microsoft.azure.management.resources.ResourceGroup;
    import com.microsoft.azure.management.resources.fluentcore.arm.Region;
    import com.microsoft.azure.management.resources.fluentcore.model.Creatable;
    import com.microsoft.rest.LogLevel;
    import java.io.File;
    ```

3. To create the Active Directory credentials that you need to make requests, add this code to the main method of the App class:
   
    ```java    
    final File credFile = new File(System.getenv("AZURE_AUTH_LOCATION"));
    Azure azure = Azure.configure()
        .withLogLevel(LogLevel.BASIC)
        .authenticate(credFile)
        .withDefaultSubscription();
    ```

## Create resources

### Create the resource group

All resources must be contained in a [Resource group](../../azure-resource-manager/resource-group-overview.md).

To specify values for the application and create the resource group, add this code to the main method:

```java
String groupName = "myResourceGroup";
String vmName = "myVM";
Region location = Region.US_WEST;

System.out.println("Creating resource group...");
ResourceGroup resourceGroup = azure.resourceGroups()
    .define(groupName)
    .withRegion(location)
    .create();
```

### Create the availability set

[Availability sets](tutorial-availability-sets.md) make it easier for you to maintain the virtual machines used by your application.

To create the availability set, add this code to the main method:

```java
System.out.println("Creating availability set...");
AvailabilitySet availabilitySet = azure.availabilitySets()
    .define("myAvailabilitySet")
    .withRegion(location)
    .withExistingResourceGroup(groupName)
    .withSku(AvailabilitySetSkuTypes.MANAGED)
    .create();
```
### Create the public IP address

A [Public IP address](../../virtual-network/virtual-network-ip-addresses-overview-arm.md) is needed to communicate with the virtual machine.

To create the public IP address for the virtual machine, add this code to the main method:

```java
System.out.println("Creating public IP address...");
PublicIPAddress publicIPAddress = azure.publicIPAddresses()
    .define("myPublicIP")
    .withRegion(location)
    .withExistingResourceGroup(groupName)
    .withDynamicIP()
    .create();
```

### Create the virtual network

A virtual machine must be in a subnet of a [Virtual network](../../virtual-network/virtual-networks-overview.md).

To create a subnet and a virtual network, add this code to the main method:

```java
System.out.println("Creating virtual network...");
Network network = azure.networks()
    .define("myVN")
    .withRegion(location)
    .withExistingResourceGroup(groupName)
    .withAddressSpace("10.0.0.0/16")
    .withSubnet("mySubnet","10.0.0.0/24")
    .create();
```

### Create the network interface

A virtual machine needs a network interface to communicate on the virtual network.

To create a network interface, add this code to the main method:

```java
System.out.println("Creating network interface...");
NetworkInterface networkInterface = azure.networkInterfaces()
    .define("myNIC")
    .withRegion(location)
    .withExistingResourceGroup(groupName)
    .withExistingPrimaryNetwork(network)
    .withSubnet("mySubnet")
    .withPrimaryPrivateIPAddressDynamic()
    .withExistingPrimaryPublicIPAddress(publicIPAddress)
    .create();
```

### Create the virtual machine

Now that you created all the supporting resources, you can create a virtual machine.

To create the virtual machine, add this code to the main method:

```java
System.out.println("Creating virtual machine...");
VirtualMachine virtualMachine = azure.virtualMachines()
    .define(vmName)
    .withRegion(location)
    .withExistingResourceGroup(groupName)
    .withExistingPrimaryNetworkInterface(networkInterface)
    .withLatestWindowsImage("MicrosoftWindowsServer", "WindowsServer", "2012-R2-Datacenter")
    .withAdminUsername("azureuser")
    .withAdminPassword("Azure12345678")
    .withComputerName(vmName)
    .withExistingAvailabilitySet(availabilitySet)
    .withSize("Standard_DS1")
    .create();
Scanner input = new Scanner(System.in);
System.out.println("Press enter to get information about the VM...");
input.nextLine();
```

> [!NOTE]
> This tutorial creates a virtual machine running a version of the Windows Server operating system. To learn more about selecting other images, see [Navigate and select Azure virtual machine images with Windows PowerShell and the Azure CLI](../linux/cli-ps-findimage.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
> 
>

If you want to use an existing disk instead of a marketplace image, use this code: 

```java
ManagedDisk managedDisk = azure.disks.define("myosdisk") 
    .withRegion(location) 
    .withExistingResourceGroup(groupName) 
    .withWindowsFromVhd("https://mystorage.blob.core.windows.net/vhds/myosdisk.vhd") 
    .withSizeInGB(128) 
    .withSku(DiskSkuTypes.PremiumLRS) 
    .create(); 

azure.virtualMachines.define("myVM") 
    .withRegion(location) 
    .withExistingResourceGroup(groupName) 
    .withExistingPrimaryNetworkInterface(networkInterface) 
    .withSpecializedOSDisk(managedDisk, OperatingSystemTypes.Windows) 
    .withExistingAvailabilitySet(availabilitySet) 
    .withSize(VirtualMachineSizeTypes.StandardDS1) 
    .create(); 
``` 

## Perform management tasks

During the lifecycle of a virtual machine, you may want to run management tasks such as starting, stopping, or deleting a virtual machine. Additionally, you may want to create code to automate repetitive or complex tasks.

### Get information about the VM

To get information about the virtual machine, add this method to the Program class:

```java
VirtualMachine vm = azure.virtualMachines().getByResourceGroup(groupName, vmName);
System.out.println("hardwareProfile");
System.out.println("    vmSize: " + vm.size());
System.out.println("storageProfile");
System.out.println("  imageReference");
System.out.println("    publisher: " + vm.storageProfile().imageReference().publisher());
System.out.println("    offer: " + vm.storageProfile().imageReference().offer());
System.out.println("    sku: " + vm.storageProfile().imageReference().sku());
System.out.println("    version: " + vm.storageProfile().imageReference().version());
System.out.println("  osDisk");
System.out.println("    osType: " + vm.storageProfile().osDisk().osType());
System.out.println("    name: " + vm.storageProfile().osDisk().name());
System.out.println("    createOption: " + vm.storageProfile().osDisk().createOption());
System.out.println("    caching: " + vm.storageProfile().osDisk().caching());
System.out.println("osProfile");
System.out.println("    computerName: " + vm.osProfile().computerName());
System.out.println("    adminUserName: " + vm.osProfile().adminUsername());
System.out.println("    provisionVMAgent: " + vm.osProfile().windowsConfiguration().provisionVMAgent());
System.out.println("    enableAutomaticUpdates: " + vm.osProfile().windowsConfiguration().enableAutomaticUpdates());
System.out.println("networkProfile");
System.out.println("    networkInterface: " + vm.primaryNetworkInterfaceId());
System.out.println("vmAgent");
System.out.println("  vmAgentVersion: " + vm.instanceView().vmAgent().vmAgentVersion());
System.out.println("    statuses");
for(InstanceViewStatus status : vm.instanceView().vmAgent().statuses()) {
    System.out.println("    code: " + status.code());
    System.out.println("    displayStatus: " + status.displayStatus());
    System.out.println("    message: " + status.message());
    System.out.println("    time: " + status.time());
}
System.out.println("disks");
for(DiskInstanceView disk : vm.instanceView().disks()) {
    System.out.println("  name: " + disk.name());
    System.out.println("  statuses");
    for(InstanceViewStatus status : disk.statuses()) {
        System.out.println("    code: " + status.code());
        System.out.println("    displayStatus: " + status.displayStatus());
        System.out.println("    time: " + status.time());
    }
}
System.out.println("VM general status");
System.out.println("  provisioningStatus: " + vm.provisioningState());
System.out.println("  id: " + vm.id());
System.out.println("  name: " + vm.name());
System.out.println("  type: " + vm.type());
System.out.println("VM instance status");
for(InstanceViewStatus status : vm.instanceView().statuses()) {
    System.out.println("  code: " + status.code());
    System.out.println("  displayStatus: " + status.displayStatus());
}
System.out.println("Press enter to continue...");
input.nextLine();   
```

### Stop the VM

You can stop a virtual machine and keep all its settings, but continue to be charged for it, or you can stop a virtual machine and deallocate it. When a virtual machine is deallocated, all resources associated with it are also deallocated and billing ends for it.

To stop the virtual machine without deallocating it, add this code to the main method:

```java
System.out.println("Stopping vm...");
vm.powerOff();
System.out.println("Press enter to continue...");
input.nextLine();
```

If you want to deallocate the virtual machine, change the PowerOff call to this code:

```java
vm.deallocate();
```

### Start the VM

To start the virtual machine, add this code to the main method:

```java
System.out.println("Starting vm...");
vm.start();
System.out.println("Press enter to continue...");
input.nextLine();
```

### Resize the VM

Many aspects of deployment should be considered when deciding on a size for your virtual machine. For more information, see [VM sizes](sizes.md).  

To change size of the virtual machine, add this code to the main method:

```java
System.out.println("Resizing vm...");
vm.update()
    .withSize(VirtualMachineSizeTypes.STANDARD_DS2)
    .apply();
System.out.println("Press enter to continue...");
input.nextLine();
```

### Add a data disk to the VM

To add a data disk to the virtual machine, add this code to the Main method to add a data disk that is 2 GB in size, han a LUN of 0 and a caching type of ReadWrite:

```java
System.out.println("Adding data disk...");
vm.update()
    .withNewDataDisk(2, 0, CachingTypes.READ_WRITE)
    .apply();
System.out.println("Press enter to continue...");
input.nextLine();
```

## Delete resources

Because you are charged for resources used in Azure, it is always good practice to delete resources that are no longer needed. If you want to delete the virtual machines and all the supporting resources, all you have to do is delete the resource group.

To delete the resource group, add this code to the main method:
   
```java
System.out.println("Deleting resources...");
azure.resourceGroups().deleteByName(groupName);
```

4. Save the App.java file.

## Run the application

It should take about five minutes for this console application to run completely from start to finish.

1. To run the application, run this Maven command:

    ```
    mvn compile exec:java
    ```

2. Before you press **Enter** to start deleting resources, you could take a few minutes to verify the creation of the resources in the Azure portal. Click the deployment status to see information about the deployment.


## Next steps
* Learn more about using the [Azure libraries for Java](https://docs.microsoft.com/en-us/java/azure/java-sdk-azure-overview).

