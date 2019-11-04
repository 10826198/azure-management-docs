---
title: Tenants, roles, and users in Azure Lighthouse scenarios
description: Understand the concepts of Azure Active Directory tenants, users, and roles, as well as how they can be used in Azure Lighthouse scenarios.
author: JnHs
ms.service: lighthouse
ms.author: jenhayes
ms.date: 11/05/2019
ms.topic: overview
manager: carmonm
---

# Tenants, roles, and users in Azure Lighthouse scenarios

Before onboarding customers for [Azure delegated resource management](azure-delegated-resource-management.md), it's important to understand the concepts of Azure Active Directory (Azure AD) tenants, users, and roles, as well as how they can be used in Azure Lighthouse scenarios.

A *tenant* is a dedicated and trusted instance of Azure AD. Typically, each tenant represents a single organization. Azure delegated resource management enables logical projection of resources from one tenant to another tenant, allowing resources in the first tenant to be managed by users in the second tenant.

This projection requires a subscription (or one or more resource groups within a subscription) to be *onboarded* for Azure delegated resource management. This onboarding process can be done either [through Azure Resource Manager templates](../how-to/onboard-customer.md) or by [publishing a public or private offer to Azure Marketplace](../how-to/publish-managed-services-offers.md).

Whichever onboarding method you choose, you will need to define *authorizations*. Each authorization specifies a user account in the managing tenant which will have access to the delegated resources, and a built-in role that sets the permissions that each of these users will have for these resources.

## Supported roles for Azure delegated resource management

When defining an authorization, each user account must be assigned one of the [role-based access control (RBAC) built-in roles](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles). Custom roles and [classic subscription administrator roles](https://docs.microsoft.com/azure/role-based-access-control/classic-administrators) are not supported.

All built-in roles are currently supported with Azure delegated resource management, with the following exceptions:
- The Owner role is not supported.
- Any built-in roles with [DataActions](https://docs.microsoft.com/azure/role-based-access-control/role-definitions#dataactions) permission are not supported.
- The User Access Administrator built-in role is supported, but only for the specific purpose of [assigning roles to a managed identity in the customer tenant](../how-to/deploy-policy-remediation.md#create-a-user-who-can-assign-roles-to-a-managed-identity-in-the-customer-tenant). If you define a user with the User Access Administrator, you must also specify the built-in role(s) that this user can assign to managed identities.

## Best practices for defining users and roles

In most cases, you'll want to assign permissions to an Azure AD user group or service principal, rather than to a series of individual user accounts. This lets you add or remove access for individual users without having to update and republish the plan when your access requirements change.

Be sure to follow the principle of least privilege so that users only have the permissions needed to complete their job, helping to reduce the chance of inadvertent errors. For more info, see [Recommended security practices](../concepts/recommended-security-practices.md).

## Next steps

- Learn about [cross-tenant management experiences](cross-tenant-management-experience.md).
- Learn about [Azure delegated resource management](azure-delegated-resource-management.md).