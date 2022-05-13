---
title: Build, Sign and Verify a container image using notation and certificate in Azure Key Vault
description: Create a signing certificate, build container image, remote sign image with notation and Azure Key Vault, verify the container image using  Azure container registry.
author: dtzar
ms.author: davete
ms.service: container-registry
ms.topic: how-to
ms.date: 05/08/2022
---

# Build, Sign, and Verify container images using Notary and Azure Key Vault

Signed containers enables users to assure deployments are built by the entities they trust.

In this article you learn:

> [!div class="checklist"]
> * How to store a signing certificate in Azure Key Vault
> * How to remotely sign container images with notation
> * How to verify container image signature    
> TODO: Include a diagram of AKV --> Build --> Sign --> ACR --> Verify

## Prerequisites

This article requires the following to be completed or installed:

- [Complete the steps in this article at least up to **Sign in with ORAS**](/articles/container-registry/container-registry-oras-artifacts#sign-in-with-oras-1)
- [An Azure Key Vault](/azure/key-vault/general/quick-create-cli)
- [Install the notation CLI and Azure Key Vault plugin](#install-the-notation-cli-and-akv-plugin)

This article can be run in the [Azure Cloud Shell](https://portal.azure.com/#cloudshell/)

## Install the notation CLI and AKV plugin

> NOTE: The walk-through uses pre-released versions of notation and notation plugins.  

1. Install notation with plugin support from [this release](https://github.com/notaryproject/notation/releases/tag/feat-kv-extensibility)

    ```bash
    # Choose a binary
    timestamp=20220121081115
    commit=17c7607

    # Download, extract and install
    curl -Lo notation.tar.gz https://github.com/notaryproject/notation/releases/download/feat-kv-extensibility/notation-feat-kv-extensibility-$timestamp-$commit.tar.gz
    tar xvzf notation.tar.gz
    tar xvzf notation_0.0.0-SNAPSHOT-${commit}_linux_amd64.tar.gz -C ~/bin notation
        
    # Copy the notation cli to the desired bin directory in your PATH
    cp ~/bin/notation /usr/local/bin
    ```

2. Install the notation-Azure-kv plugin for remote signing and verification

    ```bash
    # Create a directory for the plugin
    mkdir -p ~/.config/notation/plugins/azure-kv
    
    # Download the plugin
    curl -Lo notation-azure-kv.tar.gz \
        https://github.com/Azure/notation-azure-kv/releases/download/v0.1.0-alpha.1/notation-azure-kv_0.1.0-alpha.1_Linux_amd64.tar.gz
    
    # Extract to the plugin directory    
    tar xvzf notation-azure-kv.tar.gz -C ~/.config/notation/plugins/azure-kv notation-azure-kv
    ```

3. Configure the Azure Key Vault plugin for notation

    ```bash
    notation plugin add azure-kv ~/.config/notation/plugins/azure-kv/notation-azure-kv
    ```

4. List the available plugins and verify that the plug in is available

    ```bash
    notation plugin ls
    ```

## Configure Environment Variables

To ease the execution of the commands to complete this article, provide values for the Azure resources.

>  NOTE: This should match the existing Azure Container Registry and Azure Key Vault resources already created in the pre-requisite steps.

1. Configure Azure Key Vault resource names

    ```bash
    # Name of the existing AKV Resource Group
    AKV_RG=myResourceGroup
    # Name of the existing Azure Key Vault used to store the signing keys
    AKV_NAME=<your-unique-keyvault-name>
    # New desired key name used to sign and verify
    KEY_NAME=wabbit-networks-io
    KEY_SUBJECT_NAME=wabbit-networks.io
    ```

1. Configure Azure Container Registry and image resource names

    ```bash
    # Name of the existing registry example: myregistry.azurecr.io
    ACR_NAME=myregistry
    # Existing full domain of the ACR
    REGISTRY=$ACR_NAME.azurecr.io
    # Container name inside ACR where image will be stored
    REPO=net-monitor
    TAG=v1
    IMAGE=$REGISTRY/${REPO}:$TAG
    # Source code directory containing Dockerfile to build
    IMAGE_SOURCE=https://github.com/wabbit-networks/net-monitor.git#main
    ```

## Create a service principal and assign permissions to the key

1. Create a service principal

    ```bash
    # Service Principal Name
    SP_NAME=https://${AKV_NAME}-sp

    # Create the service principal, capturing the password
    export AZURE_CLIENT_SECRET=$(az ad sp create-for-rbac --skip-assignment --name $SP_NAME --query "password" --output tsv)

    # Capture the service principal appId
    export AZURE_CLIENT_ID=$(az ad sp list --display-name $SP_NAME --query "[].appId" --output tsv)

    # Capture the Azure Tenant ID
    export AZURE_TENANT_ID=$(az account show --query "tenantId" -o tsv)
    ```

1. Assign key and certificate permissions to the service principal object id

    ```azure-cli
    az keyvault set-policy --name $AKV_NAME --key-permissions get sign --spn $AZURE_CLIENT_ID

    az keyvault set-policy --name $AKV_NAME --certificate-permissions get --spn $AZURE_CLIENT_ID

## Store the signing certificate in Azure Key Vault

In this step, create or provide an x509 signing certificate, storing it in Azure Key Vault for remote signing.

If you have an existing certificate, upload to Azure Key Vault and skip to [Create a service principal and assign permissions to the key](#create-a-service-principal-and-assign-permissions-to-the-key)

### Create a self-signed Certificate (Azure Azure CLI)

1. Create a certificate policy file

    ```bash
    cat <<EOF > ./my_policy.json
    {
        "issuerParameters": {
        "certificateTransparency": null,
        "name": "Self"
        },
        "x509CertificateProperties": {
        "ekus": [
            "1.3.6.1.5.5.7.3.1",
            "1.3.6.1.5.5.7.3.2",
            "1.3.6.1.5.5.7.3.3"
        ],
        "subject": "CN=${KEY_SUBJECT_NAME}",
        "validityInMonths": 12
        }
    }
    EOF
    ```

2. Create the certificate
    ```azure-cli
    az keyvault certificate create -n $KEY_NAME --vault-name $AKV_NAME -p @my_policy.json
    ```

3. Get the Key Id for the certificate

    ```bash
    KEY_ID=$(az keyvault certificate show --vault-name $AKV_NAME \
                        --name $KEY_NAME \
                        --query "kid" -o tsv)
    ```
3. Add the Key Id to the kms keys and certs

    ```bash
    notation key add --name $KEY_NAME --plugin azure-kv --id $KEY_ID --kms
    notation cert add --name $KEY_NAME --plugin azure-kv --id $KEY_ID --kms
    ```

4. List the keys and certs to confirm

    ```bash
    notation key ls
    notation cert ls
    ```

## Build and sign a container image

1. Build and Push a new image with ACR Tasks

    ```azure-cli
    az acr build -r $ACR_NAME -t $IMAGE $IMAGE_SOURCE
    ```

2. Create an ACR Token for notation signing and the ORAS CLI to access the registry

    ```azure-cli
    export NOTATION_USERNAME="00000000-0000-0000-0000-000000000000"
    export NOTATION_PASSWORD=$(az acr login --name $ACR_NAME --expose-token --output tsv --query accessToken)
    ```

3. Sign the container image

    ```bash
    notation sign $IMAGE --key $KEY_NAME
    ```

## View the Graph of Artifacts with the ORAS CLI

ACR support for ORAS Artifacts creates a linked graph of supply chain artifacts that can be viewed through the ORAS CLI, the Azure CLI, or the Azure portal.

1. Signed images can be view with the ORAS CLI

    ```azure-cli
    oras login -u $NOTATION_USERNAME -p $NOTATION_PASSWORD $REGISTRY
    oras discover -o tree $IMAGE
    ```

## View the Graph of Artifacts with the Azure CLI

> TODO: Call out the expected `az acr repository show` commands and output  
> NOTE: This work is not yet complete, rather the target experience

1. List the manifests within the repo

    ```azure-cli
    az acr repository show-manifests \
      --name $ACR_NAME \
      --repository $REPO \
      --detail -o jsonc
    ```

    generates a result, showing the `subject` in the artifact, representing the notary v2 signature, that points to the container image. Notice, the `"tags": []` collection is empty.

    ```json
    [
    {
        "changeableAttributes": {
        "deleteEnabled": true,
        "listEnabled": true,
        "readEnabled": true,
        "writeEnabled": true,
        "quarantineState": "Passed"
        },
        "artifactType": "application/vnd.cncf.notary.v2",
        "createdTime": "2021-11-09T23:36:26.3394051Z",
        "digest": "sha256:348d322ed14675993e0926bc6a681ff03d03dc8372c4c4d0c09e08fa7cbeccbe",
        "imageSize": 32,
        "lastUpdateTime": "2021-11-09T23:36:26.3394051Z",
        "mediaType": "application/vnd.cncf.oras.artifact.manifest.v1+json",
        "size": 32,
        "subject": "sha256:a0fc570a245b09ed752c42d600ee3bb5b4f77bbd70d8898780b7ab43454530eb"
    },
    "tags": []
    ]
    ```

## View the Graph of Artifacts in the Azure Portal

1. Navigate to the Azure Portal for the configured ACR
2. Select repositories
3. Search for the image name
4. Expand the node

![Viewing ORAS Artifacts in the Azure Portal](./media/container-registry-tutorial-sign-build-push/portal-oras-artifacts.png)

## Summary

In step 1, the following were completed:

- 1
- 2
- 3
## Next steps

- Enable policy to only deploy signed images to AKS 

[notation-cli]:         https://github.com/notaryproject/notation/releases
[notation-akv-plugin]: https://github.com/Azure/notation-akv/releases
[oras-cli]:             https://github.com/oras-project/oras/releases/tag/v0.2.1-alpha.1