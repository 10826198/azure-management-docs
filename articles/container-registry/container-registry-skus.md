---
title: Azure Container Registry SKUs
description: Comparisons between the different service tiers available in Azure Container Registry
services: container-registry
author: stevelas
manager: timlt

ms.service: container-registry
ms.topic: article
ms.date: 12/15/2017
ms.author: stevelas
---

# Azure Container Registry SKUs

Azure Container Registry (ACR) is available in multiple service tiers, known as SKUs. These SKUs provide predictable pricing and several options for how you wish to use your private Docker registry in Azure. Choosing a higher-level SKU provides more performance and scale. However, all SKUs provide the same programmatic capabilities, enabling a developer to get started with Basic, and convert to Standard and Premium as registry usage increases.

| SKU | Managed | Description |
| --- | :-------: | ----------- |
| **Basic** | Yes | A cost-optimized entry point for developers learning about Azure Container Registry. Basic registries have the same programmatic capabilities as Standard and Premium (Azure Active Directory authentication integration, image deletion, and web hooks), however, there are size and usage constraints. |
| **Standard** | Yes | Standard registries offer the same capabilities as Basic, with increased storage limits and image throughput. Standard registries should satisfy the needs of most production scenarios. |
| **Premium** | Yes | Premium registries provide higher limits on constraints such as storage and concurrent operations, enabling high-volume scenarios. In addition to higher image throughput capacity, Premium adds features like [geo-replication][container-registry-geo-replication] for managing a single registry across multiple regions, maintaining a network-close registry to each deployment. |
| Classic | No | The Classic registry SKU enabled the initial release of the Azure Container Registry service in Azure. Classic registries are backed by a storage account that Azure creates in your subscription, which limits the ability for ACR to provide higher-level capabilities such as increased throughput and geo-replication. Because of its limited capabilities, we plan to deprecate the Classic SKU in the future. |

> [!NOTE]
> Because of the planned deprecation of the Classic registry SKU, we recommend you use Basic, Standard, or Premium for all new registries. For information about converting your existing Classic registry, see [Upgrade a Classic registry][container-registry-upgrade].
>

## Managed vs. unmanaged

The Basic, Standard, and Premium SKUs are collectively known as *managed* registries, and Classic registries as *unmanaged*. The primary difference between the two is how your container images are stored.

### Managed (Basic, Standard, Premium)

Managed registries are backed by an Azure Storage account managed by Azure. That is, the storage account that stores your images does not appear within your Azure subscription. There are several benefits gained by using one of the managed registry SKUs, discussed in-depth in [Upgrade a Classic registry][container-registry-upgrade]. This article focuses on the managed registry SKUs and their capabilities.

### Unmanaged (Classic)

Classic registries are "unmanaged" in the sense that the storage account that backs a Classic registry resides within *your* Azure subscription. As such, you are responsible for the management of the storage account in which your container images are stored. With unmanaged registries, you can't switch between SKUs as your needs change (other than [upgrading][container-registry-upgrade] to a managed registry), and several features of managed registries are unavailable (for example, [geo-replication][container-registry-geo-replication] and [webhooks][container-registry-webhook]).

For more information about upgrading a Classic registry to one of the managed SKUs, see [Upgrade a Classic registry][container-registry-upgrade].

## Registry SKU feature matrix

The following table details the features and limits of the Basic, Standard, and Premium service tiers.

[!INCLUDE [container-instances-limits](../../includes/container-registry-limits.md)]

## Changing SKUs

You can change a registry's SKU with the Azure CLI or in the Azure portal.

### Azure CLI

To move between SKUs in the Azure CLI 2.0, use the [az acr update][az-acr-update] command. For example, to switch from Basic or Standard to Premium:

```azurecli
az acr update --name myregistry --resource-group myresourcegroup --sku Premium
```

### Azure portal

In the registry **Overview** in the Azure portal, select **Update**, then select a new **SKU** from the SKU drop-down.

![Update container registry SKU in Azure portal][update-registry-sku]

## Changing from Classic

There are additional considerations to take into account when migrating an unmanaged Classic registry to one of the managed Basic, Standard, or Premium SKUs. If your Classic registry contains a large number of images and is many gigabytes in size, the migration process can take some time. Additionally, `docker push` operations are disabled until the migration is complete.

For details on upgrading your Classic registry to one of the managed SKUs, see [Upgrade a Classic container registry][container-registry-upgrade].

## Pricing

For pricing information on each of the Azure Container Registry SKUs, see [Container Registry pricing][container-registry-pricing].

## Next steps

**Azure Container Registry Roadmap**

Visit the [ACR Roadmap][acr-roadmap] on GitHub to find information about upcoming features in the service.

**Azure Container Registry UserVoice**

Submit and vote on new feature suggestions in [ACR UserVoice][container-registry-uservoice].

<!-- IMAGES -->
[update-registry-sku]: ./media/container-registry-skus/update-registry-sku.png

<!-- LINKS - External -->
[acr-roadmap]: https://aka.ms/acr/roadmap
[container-registry-pricing]: https://azure.microsoft.com/pricing/details/container-registry/
[container-registry-uservoice]: https://feedback.azure.com/forums/903958-azure-container-registry

<!-- LINKS - Internal -->
[az-acr-update]: /cli/azure/acr#az_acr_update
[container-registry-geo-replication]: container-registry-geo-replication.md
[container-registry-upgrade]: container-registry-upgrade.md
[container-registry-webhook]: container-registry-webhook.md
