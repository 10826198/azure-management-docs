---
title: Transfer images
description: Transfer images in bulk from one container registry to another registry by creating a transfer pipeline using Azure storage accounts
ms.topic: article
ms.date: 04/07/2020
ms.custom: 
---

# Transfer artifacts to another registry

This article shows how to transfer images or other registry artifacts in bulk from one Azure container registry to another registry. The source and target registries can be in the same or different subscriptions, or potentially different Active Directory tenants or Azure clouds. 

To transfer artifacts, you create a *transfer pipeline* that replicates artifacts between registries by using blob storage:

* Artifacts from a source registry are exported to a blob in a source storage account 
* The blob is copied from the source storage account to a target storage account
* The blob in the target storage account gets imported as artifacts in the target registry. You can set up the import pipeline to trigger whenever the artifact blob updates in the target storage.

Transferring registry artifacts is a more general, scalable alternative to [importing images](container-registry-import-images.md) from one container registry to another.

In this article, you use the Azure CLI and Azure Resource Manager templates to create the resources and transfer pipeline. If you need to install or upgrade the CLI, see [Install Azure CLI][azure-cli].

This feature is available in the **Premium** container registry service tier. For information about registry service tiers and limits, see [Azure Container Registry SKUs](container-registry-skus.md).

## Prerequisites

* **Container registries** - You need an existing source registry with artifacts to transfer, and a target registry. The source and target registries can be in the same or a different Azure subscription, Active Directory tenant, or cloud. If you need to create a registry, see [Quickstart: Create a private container registry using the Azure CLI](container-registry-get-started-azure-cli.md). 
* **Storage accounts** - Create source and target storage accounts in a subscription and location of your choice. If needed, create the storage accounts with the [Azure CLI](../storage/common/storage-account-create.md?tabs=azure-cli) or other tools. In each account, create a blob container for artifact transfer. For example, create a container named *transfer*.
* **Key vaults** Create key vaults to store secrets in the same Azure subscription or subscriptions as your source and target registries. If needed, create source and target key vaults with the [Azure CLI](../key-vault/quick-create-cli.md) or other tools.

> [!NOTE]
> The Azure CLI examples in this article use a `resourceGroup` environment variable for the resource group. Depending on your scenario, you may need separate resource groups for your source and target environments. All examples are formatted for the Bash shell.

## Scenario overview

You create the following three resources for ACR Transfer. All are created using PUT operations. These resources operate on your *source* and *target* registries and storage accounts.

* **ExportPipeline** - Long-lasting resource that contains high-level information about the *source* registry and storage account. This information includes the source storage blob container URI and the key vault secret URI of the storage SAS token. 
* **ImportPipeline** - Long-lasting resource that contains high-level information about the *target* registry and storage account. This information includes the target storage blob container URI and the key vault secret URI of the storage SAS token. An import trigger is enabled by default, so the pipeline runs automatically when artifacts land in the target storage container. 
* **PipelineRun** - Resource used to invoke either an ExportPipeline or ImportPipeline resource.  
  * You run the ExportPipeline manually by creating a PipelineRun resource and specify the artifacts to export.  
  * If an import trigger is enabled, the ImportPipeline runs automatically. It can also be run manually using a PipelineRun. 

### Things to know
* The ImportPipeline and ExportPipeline may be located in different Active Directory tenants, or different Azure clouds. If run in different tenants or clouds, you need separate managed identities and key vaults for the export and import resources. 
* ExportPipelines and ImportPipelines also support system-assigned identities. In this case, assign the identity permissions to your key vault after the ExportPipeline resource is created and before running. 

## Create and store SAS tokens

Transfer uses shared access signature (SAS) tokens to export to and import from the designated storage accounts. Generate SAS tokens as described in the following sections.  

### SAS token for export

Generate a SAS token for the source storage account, used for artifact export.

SAS properties:
* **Allowed services** - Blob 
* **Allowed resource types** - Container, Object 
* **Allowed permissions** - Read, Write, List, Add, Create

You can accept default values for other settings.

Generate the SAS using the Azure portal or the [az storage account generate-sas][az-storage-account-generate-sas] command.

Copy the generated SAS token and use it to set the EXPORT_SAS environment variable:

```console
EXPORT_SAS='?sv=2019-02-02&...'
```

Store the SAS token in your source Azure key vault using [az keyvault secret set][az-keyvault-secret-set]:

```azurecli
az keyvault secret set \
  --name acrexportsas \
  --value $EXPORT_SAS \
  --vault-name sourcekeyvault 
```

In the command output, take note of the secret's URI (`id`). You use the URI in the export pipelines. Example:

```azurecli
https://sourcekeyvault.vault-int.azure-int.net/secrets/acrexportsas/xxxxxxxxxxxxxxx
```

### SAS token for import

Generate a SAS token for the target storage account, used for artifact import.

SAS properties:
* **Allowed services** - Blob 
* **Allowed resource types** - Service, Container, Object 
* **Allowed permissions** - Read, Delete, List

You can accept default values for other settings.

Generate a SAS using the Azure portal or the [az storage account generate-sas][az-storage-account-generate-sas] command.

Copy the generated SAS token and use it to set the IMPORT_SAS environment variable:

```console
IMPORT_SAS='?sv=2019-02-02&...'
```

Store the SAS token in your target Azure key vault using [az keyvault secret set][az-keyvault-secret-set] command:

```azurecli
az keyvault secret set \
  --name acrimportsas \
  --value $IMPORT_SAS \
  --vault-name targetkeyvault 
```
In the command output, take note of the secret's URI (`id`). You use the URI in the import pipeline. Example:

```azurecli
https://targetkeyvault.vault-int.azure-int.net/secrets/acrimportsas/xxxxxxxxxxxxxxx
```

## Create identities 

Create user-assigned managed identities to access source and target key vaults by running the [az identity create][az-identity-create] command. 

```azurecli
# Managed identity to access source vault
az identity create \
  --resource-group $resourceGroup \
  --name sourceId  

# Managed identity to access target vault
az identity create \
  --resource-group $resourceGroup \
  --name targetId 
```

Set the following variables using the [az identity show][az-identity-show] command:

```azurecli
sourcePrincipalID=$(az identity show \
  --resource-group $resourceGroup \
  --name sourceId --query principalId --output tsv) 

sourceResourceID=$(az identity show \
  --resource-group $resourceGroup \
  --name sourceId --query id --output tsv) 

targetPrincipalID=$(az identity show \
  --resource-group $resourceGroup \
  --name targetId --query principalId --output tsv) 

targetResourceID=$(az identity show \
  --resource-group $resourceGroup \
  --name targetId --query id --output tsv) 
```

## Grant each identity access to key vault 

Run the [az keyvault set-policy][az-keyvault-set-policy] command to grant the source and target identities access to their respective key vaults:

```azurecli
# Source key vault
az keyvault set-policy --name sourcekeyvault \
  --resource-group $resourceGroup \
  --object-id $sourcePrincipalID \
  --secret-permissions get

# Target key vault
az keyvault set-policy --name targetkeyvault \
  --resource-group $resourceGroup \
  --object-id $targetPrincipalID \
  --secret-permissions get
```

## Export

### Create the ExportPipeline resource 

Create an ExportPipeline resource for your source container registry using Azure Resource Manager template deployment. The ExportPipeline resource is provisioned with the source user-assigned identity you created in the previous section.

Copy ExportPipeline Resource Manager template files from [here](add link - TBD).

Enter the following parameter values in the file `azuredeploy.parameters.json`:

|Parameter  |Value  |
|---------|---------|
|registryName     | Name of your source container registry      |
|exportPipelineName     |  Name you choose for the export pipeline       |
|targetUri     |  URI of the container in your source storage account.<br/>Example: `https://sourcestorage.blob.core.windows.net/transfer`       |
|keyVaultUri     |  URI of the SAS token secret in the source key vault.<br/>Example: `https://sourcevault.vault-int.azure-int.net/secrets/acrexportsas/xxxxxxxxxx`       |

Run [az deployment group create][az-deployment-group-create] to create the resource.

```azurecli
az deployment group create \
  --resource-group $resourceGroup \
  --template-file azuredeploy.json \
  --parameters azuredeploy.parameters.json \
  --parameters userAssignedIdentity=$sourceResourceID
```

Take note of the resource ID (`id`) of the pipeline, which is used in later steps. Example:

```
"/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.ContainerRegistry/registries/<sourceRegistryName>/exportPipelines/myExportPipeline"
```

### Run the ExportPipeline resource 

Create a PipelineRun resource for your source container registry using Azure Resource Manager template deployment. This resource runs the ExportPipeline resource you created in the previous step, and exports specified artifacts from your container registry to your source storage account.

Copy ExportPipeline Resource Manager template files from [here](add link - TBD).

Enter the following parameter values in the file `azuredeploy.parameters.json`:

|Parameter  |Value  |
|---------|---------|
|registryName     | Name of your source container registry      |
|pipelineRunName     |  Name you choose for the run       |
|pipelineResourceId     |  Resource ID of the export pipeline.<br/>Example: `/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.ContainerRegistry/registries/<sourceRegistryName>/exportPipelines/myExportPipeline`|
|targetName     |  Name you choose for the artifacts blob exported to your source storage account, such as *myblob*
|artifacts | Array of source artifacts to transfer.<br/>Example: `[samples/hello-world:v1", "samples/nginx:v1"]`

Run [az deployment group create][az-deployment-group-create] to create the PipelineRun resource.

```azurecli
az deployment group create \
  --resource-group $resourceGroup \
  --template-file azuredeploy.json \
  --parameters azuredeploy.parameters.json
```

> [!IMPORTANT]
> For artifact export, when prompted, leave the `sourceName` blank. You can also leave `catalogDigest` and `forceUpdateTag` values blank.

When deployment completes successfully, verify artifact export by viewing the exported blob in the *transfer* container of the source storage account.

## Transfer blob (optional) 

Use the AzCopy command to [transfer blob data](../storage/common/storage-use-azcopy-blobs.md#copy-blobs-between-storage-accounts) from the source storage account to the target storage account.

For example, the following [`azcopy sync`](/azure/storage/common/storage-ref-azcopy-sync) command synchronizes the *transfer* container from the source storage account with the *transfer* container in the target account. Authentication uses the export and import SAS tokens previously saved in environment variables:

```console
azcopy sync \
  'https://<source-storage-account-name>.blob.core.windows.net/transfer/'$EXPORT_SAS \
  'https://<destination-storage-account-name>.blob.core.windows.net/transfer/'$IMPORT_SAS \
  --recursive 
```

## Import 

### Create the ImportPipeline resource 

Create an ImportPipeline resource in your target container registry using Azure Resource Manager template deployment. The ImportPipeline resource is provisioned with the target user-assigned identity you created previously. By default, the pipeline is triggered to import automatically when the target storage account has artifacts.

Copy ImportPipeline Resource Manager template files from [here](add link - TBD).

Enter the following parameter values in the file `azuredeploy.parameters.json`:

Parameter  |Value  |
|---------|---------|
|registryName     | Name of your target container registry      |
|importPipelineName     |  Name you choose for the import pipeline       |
|sourceUri     |  URI of the container in your target storage account (the source for the import pipeline).<br/>Example: `https://targetstorage.blob.core.windows.net/transfer`       |
|keyVaultUri     |  URI of the SAS token secret in the target key vault.<br/>Example: `https://targetvault.vault-int.azure-int.net/secrets/acrimportsas`       |

Run [az deployment group create][az-deployment-group-create] to create the resource.

```azurecli
az deployment group create \ 
  --resource-group $resourceGroup \ 
  --template-file azuredeploy.json \ 
  --parameters azuredeploy.parameters.json \
  --parameters userAssignedIdentity=$targetResourceID 
```

When deployment completes successfully, verify artifact import by listing the repositories in the target container registry. For example, run [az acr repository list][az-acr-repository-list]:

```azurecli
az acr repository list --name <target-registry-name>
```

### Run the ImportPipeline resource manually (optional) 
 
You can also use a PipelineRun resource to trigger an ImportPipeline for artifact import to your target container registry.

Copy ExportPipeline Resource Manager template files from [here](add link - TBD).

Enter the following parameter values in the file `azuredeploy.parameters.json`:

|Parameter  |Value  |
|---------|---------|
|registryName     | Name of your target container registry      |
|pipelineRunName     |  Name you choose for the run       |
|pipelineResourceId     |  Resource ID of the import pipeline.<br/>Example: `/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.ContainerRegistry/registries/<sourceRegistryName>/importPipelines/myImportPipeline`       |
|sourceName     |  Name of the existing blob for exported artifacts in your storage account, such as *myblob*

Run [az deployment group create][az-deployment-group-create] to run the resource.

```azurecli
az deployment group create \
  --resource-group $resourceGroup \
  --template-file azuredeploy.json \
  --parameters azuredeploy.parameters.json
```

> [!IMPORTANT]
> For artifact import to your registry, when prompted, leave the `targetName` blank. You can also leave `catalogDigest` and `forceUpdateTag` values blank.

When deployment completes successfully, verify artifact import by listing the repositories in the target container registry. For example, run [az acr repository list][az-acr-repository-list]:

```azurecli
az acr repository list --name <target-registry-name>
```

## Manage pipeline resources

[TODO: How to delete, etc.]


<!-- LINKS - External -->


<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-identity-create]: /cli/azure/identity#az-identity-create
[az-identity-show]: /cli/azure/identity#az-identity-show
[az-login]: /cli/azure/reference-index#az-login
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az-keyvault-secret-set
[az-keyvault-set-policy]: /cli/azure/keyvault#az-keyvault-set-policy
[az-storage-account-generate-sas]: /cli/azure/storage/account#az-storage-account-generate-sas
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create
[az-acr-repository-list]: /cli/azure/acr/repository#az-acr-repository-list



