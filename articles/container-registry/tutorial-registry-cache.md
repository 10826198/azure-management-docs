---
title: Enable Registry Cache for ACR
description: Learn about the Registry Cache feature, its preview limitations and benefits of enabling the feature in your Registry.
ms.topic: tutorial
ms.date: 04/19/2022
ms.author: tejaswikolli
---
# Caching for Azure Container Registry

Azure Container Registry (ACR) introduces caching for ACR. This feature will allow users to cache container images in a private container registry. The Registry Cache, is a preview feature available in *Basic*, *Standard*, and *Premium* [service tiers](container-registry-skus.md).

This article is part one in a three-part tutorial series. The tutorial covers:

> [!div class="checklist"]
> * Enabling Registry Cache
> * How to configure a Cache
> * Troubleshooting guide for Caching for ACR

## Caching for ACR

By enabling Caching for ACR, you can access, fetch, and cache images. A registry cache enables you to access content from public and private registries. 

The cached artifacts are accessible from private VNETs using ACR [Private Link Support](/azure/container-registry/container-registry-private-link). All the cached artifacts will be delivered through ACR.

Implementing Caching for ACR provides the following benefits:

***High-speed pull operations:*** Faster pulls of the container images are achievable by caching the container images in the registry. Since Microsoft manages the Azure network, pull operations are faster by providing Geo-Replication and Availability zone support to the customers.

***Private networks:*** Cached registries are available on private networks. Therefore, users can configure their firewall to increase security and meet compliance standards. 

***Docker Rate Limit:***  Docker has updated the terms of services to limit anonymous users to 100 pull operations every six hours, free Docker account users to 200 pull operations every six hours, and paid Docker subscription to 5000 pull operations per 24 hours. ACR's Registry caching allows users to pull images from the private cache upon requirment. The feature benefits the customers to avoid hitting the Docker rate limit.

## Preview Limitations

- Quarantine functions like signing, scanning, and manual compliance approval are on the roadmap for cached registries but aren't included in this release.

- Caching for ACR currently only supports caching for pulled images. Caching for ACR doesn't automatically pull new version of images when a new version is available. This is on the roadmap but isn't supported in this release. 

-  Caching for ACR only supports Docker Hub and Microsoft Container Registry. Multiple other registries are on the roadmap but aren't included in this release.

- Caching for ACR is only available via the Azure Portal. The Azure CLI will be released in the coming weeks.   

## Next steps

* To enable Registry Cache using the Azure portal advance to the next article: [Enable Registry Cache](tutorial-enable-registry-cache.md).
