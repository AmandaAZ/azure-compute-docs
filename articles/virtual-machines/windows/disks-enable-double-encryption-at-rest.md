---
title: Enable double encryption at rest - managed disks
description: Azure Storage protects your data by encrypting it at rest before persisting it to Storage clusters. You can rely on Microsoft-managed keys for the encryption of your managed disks, or you can use customer-managed keys to manage encryption with your own keys.
author: roygara

ms.date: 07/08/2020
ms.topic: conceptual
ms.author: rogarana
ms.service: virtual-machines
ms.subservice: disks
---

1. Make sure that you have installed the latest [Azure CLI](/cli/azure/install-az-cli2) and logged to an Azure account in with [az login](/cli/azure/reference-index).

1. Create an instance of Azure Key Vault and encryption key.

    When creating the Key Vault instance, you must enable soft delete and purge protection. Soft delete ensures that the Key Vault holds a deleted key for a given retention period (90 day default). Purge protection ensures that a deleted key cannot be permanently deleted until the retention period lapses. These settings protect you from losing data due to accidental deletion. These settings are mandatory when using a Key Vault for encrypting managed disks.

    > [!IMPORTANT]
    > Do not camel case the region, if you do so you may experience problems when assigning additional disks to the resource in the Azure portal.

    ```azurecli
    subscriptionId=yourSubscriptionID
    rgName=yourResourceGroupName
    location=westcentralus
    keyVaultName=yourKeyVaultName
    keyName=yourKeyName
    diskEncryptionSetName=yourDiskEncryptionSetName
    diskName=yourDiskName

    az account set --subscription $subscriptionId

    az keyvault create -n $keyVaultName -g $rgName -l $location --enable-purge-protection true --enable-soft-delete true

    az keyvault key create --vault-name $keyVaultName -n $keyName --protection software
    ```

1.    Create an instance of a DiskEncryptionSet. 
    
    ```azurecli
    New-AzResourceGroupDeployment -ResourceGroupName CMKTesting `
        -TemplateUri "https://raw.githubusercontent.com/ramankumarlive/manageddisksdoubleencryptionpreview/master/CreateDiskEncryptionSetForDoubleEncryption.json" `
        -diskEncryptionSetName $diskEncryptionSetName `
        -keyVaultId "subscriptions/yourSubscriptionID/resourceGroups/yourResourceGroupName/providers/Microsoft.KeyVault/vaults/yourKeyVaultName" `
        -keyVaultKeyUrl "https://yourKeyVaultName.vault.azure.net/keys/yourKeyName/yourKeyVaultID" `
        -encryptionType "EncryptionAtRestWithPlatformAndCustomerKeys" `
        -region "CentralUSEUAP"
    ```

1.    Grant the DiskEncryptionSet resource access to the key vault. 

    > [!NOTE]
    > It may take few minutes for Azure to create the identity of your DiskEncryptionSet in your Azure Active Directory. If you get an error like "Cannot find the Active Directory object" when running the following command, wait a few minutes and try again.

    ```azurecli
    desIdentity=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [identity.principalId] -o tsv)

    az keyvault set-policy -n $keyVaultName -g $rgName --object-id $desIdentity --key-permissions wrapkey unwrapkey get
    ```
