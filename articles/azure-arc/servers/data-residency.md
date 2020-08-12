---
title: Data residency
description: Data residency and information about Azure Arc enabled servers (preview).
ms.topic: reference
ms.date: 08/12/2020
ms.custom: references_regions
---

# Azure Arc enabled servers (preview): Data residency

This article explains the concept of data residency and how it applies to Azure Arc enabled servers (preview).

Azure Arc enabled servers (preview) is **[available in preview](https://azure.microsoft.com/global-infrastructure/services/?products=azure-arc)** in the **United States, Europe, or Asia Pacific**.

[Data residency](#data-residency) refers to where user data is stored.

## Data residency

Azure Arc enabled servers (preview) stores [Azure VM extension](manage-vm-extensions.md) configuration settings (that is, property values) that the extension requires specifying before attempting to enable on the connected machine. For example, when you enable the Log Analytics VM extension, it asks for the Log Analytics **workspace ID** and **primary key**. 

Metadata information about the connected machine are also collected. Specifically:

* operating system name and version
* computer name
* computer fully qualified domain name (FQDN)
* Connected Machine agent version

Where is the data replicated within the region? Ryan will ask the Devs to weigh-in and determine if we need to be explicit here.

This data is stored in the region where the Azure Arc machine resource is configured. For example, if the machine is registered with Arc in the East US region, this data is stored in the US region.

## Next steps