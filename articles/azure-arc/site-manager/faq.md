---
title: Frequently asked questions
description: "Frequently asked questions to understand and troubleshoot Azure Arc sites and site manager"
author: kgremban
ms.author: kgremban
ms.service: azure-arc
#ms.subservice: site-manager
ms.topic: faq #Don't change
ms.date: 02/16/2024

#customer intent: As a customer, I want answers to questions so that I can answer my own questions.

---

# Frequently asked questions: Azure Arc site manager (preview)

The following are frequently asked questions and answers for Azure Arc site manager.

**Question:** I have resources in the resource group, which aren't yet supported by site manager. Do I need to move them?

**Answer:** Site manager provides health status aggregation for only the supported resource types. Resources of other types won't be managed via site manager. They continue to function normally as they would without otherwise.

**Question:** Can I update a site after it has been created?

**Answer:** After creation, site properties and details can't be updated aside from site name and site address. The best way to update the site properties is to delete the site and create a new site.

**Question:** Does site manager have a subscription or fee for usage?

**Answer:** Site manager doesn't have any fee, so feel free to create and use as many sites as desired. However, the Azure services that integrated with sites and site manager might have a fee. Additionally, alerts used with site manager via monitor might have fees as well.

**Question:** What regions are currently supported via site manager? What regions of these supported regions aren't fully supported?

**Answer:** Regions supported can be found [here](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/?products=azure-arc&regions=all). Currently, all supported regions have full support for resources except for the following:
| Region | Support limitation |
|--|--|
| Brazil South | Connectivity and updates status aren't supported for Arc-enabled machines or Arc-enabled Kubernetes clusters |
| UAE North | Connectivity and updates status aren't supported for Arc-enabled machines or Arc-enabled Kubernetes clusters |
| South Africa North | Connectivity and updates status aren't supported for Arc-enabled machines or Arc-enabled Kubernetes clusters |

**Question:** Does an Arc site have to represent a physical site or location?

**Answer:** Site manager is tailored for customers who manage on-premises infrastructure. Arc sites can also be created to represent more than just a physical location as well.
