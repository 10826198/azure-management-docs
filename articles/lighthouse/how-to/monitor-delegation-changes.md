---
title: Monitor delegation changes in your managing tenant
description: Learn how to monitor delegation activity from customer tenants to your managing tenant. 
ms.date: 03/13/2020
ms.topic: conceptual
---

# Monitor delegation changes in your managing tenant

As a service provider, you may want to be aware when customer subscriptions and/or resource groups are delegated to your tenant through [Azure delegated resource management](../concepts/azure-delegated-resource-management.md), or when previously delegated resources are removed.

In the managing tenant, the [Azure activity log](../../azure-monitor/platform/platform-logs-overview.md) tracks delegation activity at the tenant level. This includes any added or removed delegations from all customer tenants.

This topic shows how you can monitor delegation activity from customer tenants to your managing tenant.

> [!IMPORTANT]
> All of these steps must be performed in your managing tenant, rather than in any customer tenants.

## Enable access to tenant-level data

To access tenant-level Activity Log data, an account must be assigned the [Monitoring Reader](../../role-based-access-control/built-in-roles.md#monitoring-reader) built-in role at root scope (/). This assignment must be performed by a user who has the Global Administrator role with additional elevated access.

### Elevate access for a Global Administrator account

To assign a role at root scope (/), you will need to have the Global Administrator role with elevated access. This elevated access should be added only when you need to make the role assignment and then removed when you are done. For detailed instructions on adding and removing elevation, see [Elevate access to manage all Azure subscriptions and management groups](../../role-based-access-control/elevate-access-global-admin.md).

After you elevate your access, your account will have the User Access Administrator role in Azure at root scope. This allows you to view all resources and assign access in any subscription or management group in the directory, as well as to make root-level role assignments. 

### Create a new service principal account to access tenant-level data

Once you have elevated your access, you'll need to assign the [Monitoring Reader](../../role-based-access-control/built-in-roles.md#monitoring-reader) built-in role to a service principal account at the root scope of your managing tenant.

It's important to understand that granting a role assignment at the root scope means that the same permissions will apply to every resource in the tenant. Because of this, we recommend:

- [Create a new service principal account](../../active-directory/develop/howto-create-service-principal-portal.md) to be used only for this function, rather than assigning this role to an existing service principal used for other automation.￼￼
- Be sure that this service principal does not have access to any delegated customer resources.
- [Use a certificate to authenticate](../../active-directory/develop/howto-create-service-principal-portal.md#certificates-and-secrets) and [store it securely in Azure Key Vault](../../key-vault/key-vault-best-practices.md).
- Limit the users who have access to act on behalf of the service principal.

Use one of the following methods to make the root scope assignments.

#### PowerShell

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

New-AzRoleAssignment -SignInName <yourLoginName> -Scope "/" -RoleDefinitionName "Monitoring Reader"  ApplicationId $servicePrincipal.ApplicationId 
```

#### Azure CLI

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az role assignment create --assignee 00000000-0000-0000-0000-000000000000 --role "Monitoring Reader" --scope "/"
```

### Remove elevated access for the Global Administrator account

After you've created your service principal account and assigned the Monitoring Reader role at root scope, be sure to [remove the elevated access](../../role-based-access-control/elevate-access-global-admin.md#remove-elevated-access) for the Global Administrator account, as this level of access will no longer be needed.

## Query the activity log

Once you've created a new service principal user with Monitoring Reader access to the root scope, you can use it to query and report on delegation activity. This must be done from within your managing tenant, using the service principal account that you created as described above.

The sample below uses Azure PowerShell to query the past 10 days of activity and reports on any added or removed delegations (or attempts which were not successful).

For each delegation that is added or removed, the following details are shown:

- **DelegatedResourceId**: The ID of the delegated subscription or resource group
- **CustomerTenantId**: The customer tenant ID
- **CustomerSubscriptionId**: The subscription ID which was delegated or that contains the resource group that was delegated
- **CustomerDelegationStatus**: The status change for the delegated resource (succeeded or failed)
- **EventTimeStamp**: The date and time at which the delegation change was logged

When reviewing this info, keep in mind:

- If multiple resource groups are delegated in a single deployment, separate entries will be returned for each resource group.
- Changes made to a previous delegation (such as updating the permission structure) will be logged as an added delegation.
- As noted above, a user account must have the Monitoring Reader built-in role at root scope (/) in order to access this tenant-level data.

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

$GetDate = (Get-Date).AddDays((-10))

$dateFormatForQuery = $GetDate.ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ssZ")

# Get Azure context for the API call
$currentContext = Get-AzContext

# Fetch new token
$azureRmProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
$profileClient = [Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient]::new($azureRmProfile)
$token = $profileClient.AcquireAccessToken($currentContext.Subscription.TenantId)

$listOperations = @{
    Uri = "https://management.azure.com/providers/microsoft.insights/eventtypes/management/values?api-version=2015-04-01&`$filter=eventTimestamp ge '$($dateFormatForQuery)'"
    Headers = @{
        Authorization = "Bearer $($token.AccessToken)"
        'Content-Type' = 'application/json'
    }
    Method = 'GET'
}
$list = Invoke-RestMethod @listOperations

$showOperations = $list.value

if ($showOperations.operationName.value -eq "Microsoft.Resources/tenants/register/action")
{
    $outputs  = $showOperations | Where-Object -FilterScript {$_.eventName.value -eq "EndRequest" -and $_.resourceType.value -and $_.operationName.value -eq "Microsoft.Resources/tenants/register/action"}
    foreach ($output in $outputs)
    {
    $outputdata = [pscustomobject]@{
        Event = "An Azure customer has delegated resources to your tenant.";
        DelegatedResourceId = $output.description |%{$_.split('"')[11]};
        CustomerTenantId = $output.description |%{$_.split('"')[7]};
        CustomerSubscriptionId = $output.subscriptionId;
        CustomerDelegationStatus = $output.status.value;
        EventTimeStamp = $output.eventTimestamp;
        }
        $outputdata | Format-List
    }
}
elseif ($showOperations.operationName.value -eq "Microsoft.Resources/tenants/unregister/action") 
{
    $outputs  = $showOperations | Where-Object -FilterScript {$_.eventName.value -eq "EndRequest" -and $_.resourceType.value -and $_.operationName.value -eq "Microsoft.Resources/tenants/unregister/action"}
    foreach ($output in $outputs)
    {
    $outputdata = [pscustomobject]@{
        Event = "An Azure customer has removed delegated resources from your tenant.";
        DelegatedResourceId = $output.description |%{$_.split('"')[11]};
        CustomerTenantId = $output.description |%{$_.split('"')[7]};
        CustomerSubscriptionId = $output.subscriptionId;
        CustomerDelegationStatus = $output.status.value;
        EventTimeStamp = $output.eventTimestamp;
        }
        $outputdata | Format-List
    }
}
else 
{
    Write-Output "No delegation changes."
}
```

## Next steps

- Learn how to onboard customers to [Azure delegated resource management](../concepts/azure-delegated-resource-management.md).
- Learn about [Azure Monitor](../../azure-monitor/index.yml) and the [Azure activity log](../../azure-monitor/platform/platform-logs-overview.md).
