---
title: Increase Azure Storage account quotas
description: Learn how to request an increase in the quota limit for Azure Storage accounts within a subscription from 250 to 500 for a given region. Quota increases apply to both standard and premium account types.
ms.date: 05/02/2023
ms.topic: how-to
---

# Increase Azure Storage account quotas

This article shows how to request increases for networking quotas from [Azure Home](https://portal.azure.com) or from **My quotas**, a centralized location where you can view your quota usage and request quota increases.

For quick access to request an increase, select **Quotas** on the Azure Home page.

:::image type="content" source="media/networking-quota-request/quotas-icon.png" alt-text="Screenshot of the Quotas icon in the Azure portal.":::

If you don't see **Quotas** on Azure Home, type *quotas* in the search box, then select **Quotas**. The **Quotas** icon will then appear on your Home page the next time you visit.

You can also use the following tools or APIs to view your network quota usage and limits:

- [Azure CLI](/cli/azure/storage/account#az-storage-account-show-usage)
- [Azure PowerShell](/powershell/module/az.storage/get-azstorageusage)
- [REST API](/rest/api/storagerp/usages/list-by-location)
- **Usage + quotas** (in the left pane when viewing your subscription in the Azure portal)

You can request an increase from 250 to up to 500 storage accounts per region for your subscription. This quota increase applies to storage accounts with standard endpoints.

## Request storage account quota increases

Follow these steps to request a storage account quota increase from Azure Home.

1. From [Azure Home](https://portal.azure.com), select **Quotas** and then select **Storage**.

1. Select the subscription for which you want to increase your storage account quota.

1. Locate the region where you want to increase your storage account quota you want to increase, then select the **Request increase** icon.

   :::image type="content" source="media/networking-quota-request/quota-support-icon.png" alt-text="Screenshot showing the support icon for a storage account quota.":::

1. In the **Request quota increase** dialog, enter a number up to 500.

    :::image type="content" source="media/storage-quota-requests/request-quota-increase-portal.png" alt-text="Screenshot showing how to increase your storage account quota":::

1. Select **Submit**. It may take a few minutes to process your request.

## See also

- [Scalability and performance targets for standard storage accounts](../storage/common/scalability-targets-standard-account.md)
- [Scalability targets for premium block blob storage accounts](../storage/blobs/scalability-targets-premium-block-blobs.md)
- [Scalability and performance targets for premium page blob storage accounts](../storage/blobs/scalability-targets-premium-page-blobs.md)
- [Azure subscription and service limits, quotas, and constraints](../azure-resource-manager/management/azure-subscription-service-limits.md)
