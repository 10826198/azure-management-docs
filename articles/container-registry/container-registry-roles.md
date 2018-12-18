---
title: Azure Container Registry - roles and permissions
description: Use Azure role-based access control (RBAC) and identity and access management (IAM) to provide fine-grained permissions to resources in an Azure container registry.
services: container-registry
author: dlepow

ms.service: container-registry
ms.topic: article
ms.date: 12/17/2018
ms.author: danlep
---

# Azure Container Registry roles and permissions

The Azure Container Registry service supports a set of Azure roles that provide different levels of permissions to an Azure container registry. Use Azure [role-based access control](../role-based-access-control/index.yml) (RBAC) to assign specific permissions to users or service principals that need to interact with a registry.

The following table lists the Azure roles that you can [assign](../role-based-access-control/role-assignments-portal.md) to an Azure identity, and the corresponding permissions of each role in the registry.

| Role/Permission       | [Access Resource Manager](#access-resource-manage)| [Create/delete registry](#create/delete-registry) | [Push image](#pull-image) | [Pull image](#pull-image) | [Change policies](#change-polices) | [Change quarantine state](#change-quarantine-state) | [Pull quarantine images](#pull-quarantine-images) | [Sign images](#sign-images)  |
| ---------| --------- | --------- | --------- | --------- | --------- | --------- | --------- | ---------  |
| Owner | X | X | X | X | X |  |  |   |
| Contributor | X | X | X | X | X |  |  |   |
| Reader | X |  |  | X |  |  |  |   |
| AcrPush |  |  | X | X |  |  |  |   |
| AcrPull |  |  |  | X |  |  |  |   |
| AcrQuarantineWriter |  |  |  |  |  | X | X |   |
| AcrQuarantineReader |  |  |  |  |  |  | X |   |
| AcrImageSigner |  |  |  |  |  |  |  | X |

## Differentiate users and services

Any time permissions are applied, a best practice is to provide the most limited set of permissions for a person, or service, to accomplish a task. The following permission sets represent a set of capabilities that may be used by humans and headless services.

### CI/CD solutions

When automating `docker build` commands from CI/CD solutions, you need `docker push` capabilities. For these headless service scenarios, we suggest assigning the **AcrPush** role. This role, unlike the broader **Contributor** role, prevents the account from using Azure Resource Manager for other registry operations.

### Container host nodes

Likewise, nodes running your containers need the **AcrPull** role, but shouldn't require **Reader** capabilities.

### Visual Studio Code Docker extension

For tools like the Visual Studio Code [Docker extension](https://code.visualstudio.com/docs/azure/docker), additional resource provider access is required to list the available Azure container registries. In this case, provide your users access to the **Reader** or **Contributor** role. These roles allow `docker pull`, `docker push`, `az acr list`, `az acr build`, and other capabilities. 

## Access Resource Manager

Azure Resource Manager access is required for the Azure portal and [Azure CLI](/cli/azure/). For example, to get a list of registries by using the `az acr list` command, you need this permission set. 

## Create/delete registry

The ability to create and delete Azure container registries.

## Push image

The ability to `docker push` an image to a registry.

## Pull image

The ability to `docker pull` an image, that has not been quarantined, from a registry.

## Change policies

The ability to configure policies on a registry. Policies include image purging, enabling quarantine, and image signing.

## Change quarantine state

The ability to set the quarantine state of an image. 

This role should only be assigned to vulnerability scanners using service principals. We recommend that individual users, including operations people, only use a vulnerability scanning solution to override the quarantine state.

## Pull quarantine images

The ability to `docker pull` images by a digest, allowing a vulnerability scan. 

This role should only be assigned to vulnerability scanners using service principals. We recommend that individual users, including operations people, only use a vulnerability scanning solution to override the quarantine state.

## Sign images

The ability to sign images, usually assigned to an automated process, which would use a service principal.


## Next steps

* Learn about [authentication options](container-registry-authentication.md) for Azure Container Registry.
