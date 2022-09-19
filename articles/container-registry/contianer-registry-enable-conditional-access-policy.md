---
title: Configure conditional access to your Azure Container Registry
description: Learn how to configure conditional access to your registry by using Azure CLI and Azure portal.
ms.topic: how-to
ms.date: 09/13/2021
ms.author: tejaswikolli
ms.reviewer: cegraybl 
---

# Azure Container Registry (ACR) introduces the Conditional Access policy

Azure Container Registry (ACR) gives you the option to create and configure the *Conditional Access policy*. You can refer to the ACR's conditional access policy here in the [azure-built-in-policy definition's](policy-reference.md)  

The [Conditional Access Policy](/azure/active-directory/conditional-access/overview.md) is designed to enforce strong authentication. The authentication is based on the location, trusted and compliant devices, user assigned roles, authorization method, and the client applications. The policy enables the security to meet the organizations compliance requirements and keep the data and user accounts safe.

Learn more about [Conditional Access Policy](/azure/active-directory/conditional-access/overview.md), the [conditions](/azure/active-directory/conditional-access/overview.md#common-signals,) you'll take it into consideration to make [policy decisions.](/azure/active-directory/conditional-access/overview.md#common-decisions)

In this article, you'll learn to

>* Create and configure Conditional Access policy for Azure Container Registry.
>* Enable Conditional Access policy in your Azure Container Registry - Azure portal, Azure CLI.

## Prerequisites

>* [Install or upgrade Azure CLI](/cli/azure/install-azure-cli) version 2.0.76 or later. To find the version, run `az --version`.
>* [!INCLUDE [Use Azure Cloud Shell](cloud-shell-try-it-no-header.md)]
>* Sign into [Azure portal.](https://portal.azure.com) 

## Create and configure a Conditional Access policy - Azure portal

The Azure Active Directory allows you to create a Conditional Access policy for both Active Directory users, and Service Principal users.

In this tutorial, we create a basic Conditional Access policy for Azure Container Registry from the Azure portal.

First, create a Conditional Access policy and assign your test group of users as follows:

1. Sign in to the [Azure portal](https://portal.azure.com) by using an account with *global administrator* permissions.

1. Search for and select **Azure Active Directory**. Then select **Security** from the menu on the left-hand side.

1. Select **Conditional Access**, select **+ New policy**, and then select **Create new policy**.
 
   :::image type="content" alt-text="A screenshot of the Conditional Access page, where you select 'New policy' and then select 'Create new policy'." source="media/container-registry-enable-conditional-policy/01-create-conditional-access.png":::

1. Enter a name for the policy, such as *demo*.

1. Under **Assignments**, select the current value under **Users or workload identities**.
 
   :::image type="content" alt-text="A screenshot of the Conditional Access page, where you select the current value under 'Users or workload identities'." source="media/container-registry-enable-conditional-policy/02-conditional-access-users-and-groups.png":::

1. Under **What does this policy apply to?**, verify and select **Users and groups**.

1. Under **Include**, choose **Select users and groups**, and then select **All users**.
 
   :::image type="content" alt-text="A screenshot of the page for creating a new policy, where you select options to specify users." source="media/container-registry-enable-conditional-policy/03-conditional-access-users-groups-select-users.png":::

1. Under **Exclude**, choose **Select users and groups**, to exclude any choice of selection.

1. Under **Cloud apps or actions**, choose **Cloud apps**.

1. Under **Include**, choose **Select apps**.

    :::image type="content" alt-text="A screenshot of the page for creating a new policy, where you select options to specify cloud apps." source="media/container-registry-enable-conditional-policy/04-select-cloud-apps-select-apps.png":::

1.  Browse for and select apps to apply Conditional Access, in this case *Azure Container Registry*, then choose **Select**.

   :::image type="content" alt-text="A screenshot of the list of apps, with results filtered, and 'Azure Container Registry' selected." source="media/container-registry-enable-conditional-policy/05-select-azure-container-registry-app.png":::

1.  Under **Conditions** , configure control access level with options such as *User risk level*, *Sign-in risk level*, *Sign-in risk detections (Preview)*, *Device platforms*, *Locations*, *Client apps*, *Time (Preview)*, *Filter for devices*.

1. Under **Grant**, filter and choose from options to enforce grant access or block access, during a sign-in event to the Azure portal. In this case grant access with *Require multifactor authentication*, then choose **Select**.




1. Under **Session**, filter and choose from options to enable any control on session level experience of the cloud apps.

1. After selecting and confirming, Under **Enable policy**, select **On**.

1. To apply and activate the policy, Select **Create**.

    :::image type="content" alt-text="A screenshot to activate the conditional access policy." source="media/container-registry-enable-conditional-policy/06-enable-conditional-access-policy.png":::

We have now completed creating and configuring the Conditional Access policy for the Azure Container Registry.

## Enable Conditional Access policy in the Azure Container Registry (ACR) 

To assign the Conditional Access policy, you have to sign in to the [Azure portal](https://portal.azure.com) after configuring the Conditional Access policy with Azure Container Registry.

You can Assign the Conditional Access policy in two ways, as below:
 
>* Assign a built-in Conditional Access policy definition - Azure portal.
>* Disable the azureADAuthenticationAsArmPolicy - Azure CLI.

### Assign a built-in policy definition to disable ARM audience token authentication - Azure portal.
  
You can enable registry's conditional access policy in the [Azure portal](https://portal.azure.com). 

1. Sign in to the [Azure portal](https://portal.azure.com) 

1. Navigate to your **Azure Container Registry** > **Resource Group** > **Settings** > **Policies** .
 
    :::image type="content" source="media/container-registry-enable-conditional-policy/01-azure-policies.png" alt-text="Screenshot to navigate Azure policies":::

1. Navigate to  **Azure Policy**, On the **Assignments**, select **Assign policy**.
   
   :::image type="content" source="media/container-registry-enable-conditional-policy/02-Assign-policy.png.png" alt-text="Screenshot to assign a policy":::

1. Under the **Assign policy** , use filters to search and find the **Scope**, **Policy definition**, **Assignment name**.

    :::image type="content" source="media/container-registry-enable-conditional-policy/03-Assign-policy-tab.png" alt-text="Screenshot of assign policy tab":::

1. Select **Scope** to filter and search for the **Subscription** and **ResourceGroup** and choose **Select**.
 
     :::image type="content" source="media/container-registry-enable-conditional-policy/04-select-scope.png" alt-text="Screenshot of Scope tab":::

1. Select **Policy definition** to filter and search the built-in policy definitions for the Conditional Access policy.
   
     :::image type="content" source="media/container-registry-enable-conditional-policy/05-built-in-policy-definitions.png" alt-text="Screenshot of built-in-policy-definitions":::

Azure Container Registry has two built-in policy definitions for Conditional Access Policy

>* `Container registries should have ARM audience token authentication disabled.` - This policy will report, block any non-compliant resources, and also sends a request to update non-compliant to compliant.
>* `Configure container registries to disable ARM audience token authentication.` - This policy offers remediation and updates non-compliant to compliant resources.

1. Use filters to select and confirm  **Scope**, **Policy definition**, and **Assignment name**.

1. Use the filters to limit compliance states or to search for policies.

1. Confirm your settings and set policy enforcement as **enabled**.

1. Select **Review+Create**

   :::image type="content" source="media/container-registry-enable-conditional-policy/06-enable-policy.png" alt-text="Screenshot to assign a policy":::

### Disable the authentication-as-arm in ACR - Azure CLI

Disabling the `azureADAuthenticationAsArmPolicy` will force the registry to use ACR audience token. You can use Azure CLI version 2.0.76 or later, run `az --version` to find the version. 

1. Run the command to show the current configuration of the registry's policy for authentication using ARM tokens with the registry. If the status is `enabled`, then both ACRs and ARM audience tokens can be used for authentication. If the status is `disabled` it means only ACR's audience tokens can be used for authentication.

    ```azurecli-interactive
    az acr config authentication-as-arm show -r <registry>
    ```

1. Run the command to update the status of the registry's policy.

    ```azurecli-interactive
    az acr config authentication-as-arm update -r <registry> --status [enabled/disabled]
     ```

## Next steps

To learn more about assigning policies

> [!div class="nextstepaction"]
> [Create and manage policies](../tutorials/create-and-manage.md)
> Create a [custom policy definition](../governance/policy/tutorials/create-custom-policy-definition.md).
> Learn more about [governance capabilities](../governance/index.yml).
> The [policy assignment structure](/azure/governance/policy/concepts/assignment-structure#enforcement-mode)
> Enable conditional policy using [Azure portal](../governance/policy/assign-policy-portal.md) 
> Enable conditional policy using [Azure CLI](../governance/policy/assign-policy-azurecli.md)
> Enable or disable [policy enforcement](../governance/policy/concepts/assignment-structure.md#enforcement-mode) at any time.