---
title: Configure conditional access to your Azure Container Registry
description: Learn how to configure conditional access to your Azure container registry by using Azure CLI and Azure Portal.
ms.topic: how-to
ms.date: 09/13/2021
ms.author: tejaswikolli
ms.reviewer: cegraybl 
---
# About the Conditional Access Policy

The [Conditional Access Policy](/azure/active-directory/conditional-access/overview.md) is designed to enforce strong authentication based on the location, the trusted and compliant devices, user assigned roles, authorization method and the client applications. 

The policy enables the security to meet the organizations compliance requirements and keep the data and user accounts safe.

Learn more about [Conditional Access Policy](conditional-access/overview.md), [Conditional Access Policy Conditions](conditional-access/overview.md#common-signals), and [Conditional Access Policy Decisions](/azure/active-directory/conditional-access/overview.md#common-decisions) 
## Prerequisites

* [Install or upgrade Azure CLI](/cli/azure/install-azure-cli) version 2.0.76 or later. To find the version, run `az --version`.
[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]
* Sign into [Azure portal](https://portal.azure.com) 
.
## Azure Container Registry (ACR) introduces the conditional access policy

You can refer to the ACR's conditional access policy in the
[azure-policy-reference-rp-containerreg](../../includes/policy/reference/byrp/microsoft.containerregistry.md). The [Policy Assignment Structure](/azure/governance/policy/concepts/assignment-structure#enforcement-mode) determines the 

Also learn more about:

>*  Enable or disable [policy enforcement](../governance/policy/concepts/assignment-structure.md#enforcement-mode) at any time.
>* Enable conditional policy using [Azure portal](../governance/policy/assign-policy-portal.md) 
>* Enable conditional policy using [Azure CLI](../governance/policy/assign-policy-azurecli.md)

# Enable conditional access policy in ACR - portal

You can enable registry's conditional policy in the [Azure portal](https://portal.azure.com). 

>* Sign in to the [Azure portal](https://portal.azure.com) 
>* Navigate to your **Azure Container Registry** > **Resource Group** > **Settings** > **Policies** > **Assign policy**
 
:::image type="content" source="../media/container-registry-enable-conditional-policy/01-policies.png" alt-text="Screenshot to navigate ACR built-in policies." border="false":::

>* On the **Assign policy** page, set **Scope** with **Subscription** and **ResourceGroup**
>* Use filters to confirm and select **Scope**, **Policy definition**, **Assignment name**

:::image type="content" source="../media/container-registry-enable-conditional-policy/02-assign-policy.png" alt-text="Screenshot to search and filter ACR built-in policy definition." border="false":::

>* Use the filters to limit compliance states or to search for policies.
>* Confirm your settings and set Policy enforcement as **enabled**
>* Select **Review+Create**

# Enable conditional access in ACR - Azure CLI

You can use Azure CLI version 2.0.76 or later, run `az --version` to find the version. 

1. Run the [az policy assignment create ](cli/azure/policy/assignment#az-policy-assignment-create) command in the Azure CLI to create a resource policy assignment, as below:

```azurecli-interactive
az policy assignment create --name 'ACR_AADAuthenticationAsArmPolicy_AuditDeny' --display-name 'Container registries should have ARM audience token authentication disabled' --scope '<scope>' --policy '<policy definition ID>'
```

1. Run the [az policy assignment list](/cli/azure/policy/assignment#az-policy-assignment-list) command in the Azure CLI to get the policy IDs of the ACR policies that are applied, as below:


```azurecli-interactive
az policy assignment list --query "[?displayName=='Container registries should have ARM audience token authentication disabled'].id"
```

## Clean up resources

1. Run the [az policy assignment delete](/cli/azure/policy/assignment#az-policy-assignment-delete) command in the Azure CLI to remove the assignment created, as below:

```azurecli-interactive
az policy assignment delete --name 'audit-vm-manageddisks' --scope '/subscriptions/<subscriptionID>/<resourceGroupName>'
```

## Next steps

To learn more about assigning policies to validate that new resources are compliant, continue to the
tutorial for:

> [!div class="nextstepaction"]
> [Creating and managing policies](./tutorials/create-and-manage.md)