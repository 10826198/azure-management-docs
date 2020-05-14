---
title: VM extension management with Azure Arc for servers
description: Azure Arc for servers (preview) can manage deployment of virtual machine extensions that provide post-deployment configuration and automation tasks on non-Azure VMs.
ms.date: 05/13/2020
ms.topic: conceptual
ms.service: azure-arc
ms.subservice: azure-arc-servers
author: mgoedtel
ms.author: magoedte
---

# Virtual machine extension management with Azure Arc for servers (preview)

Virtual machine (VM) extensions are small applications that provide post-deployment configuration and automation tasks on Azure VMs. For example, if a virtual machine requires software installation, anti-virus protection, or to run a script inside of it, a VM extension can be used. Azure Arc for servers (preview) enables you to deploy Azure VM extensions to non-Azure Windows and Linux VMs. 

VM extensions can be run with the Azure REST API, PowerShell, Azure Resource Manager templates, the Azure portal, or Azure policies on hybrid servers managed by Arc for servers (preview).

In this preview, we are supporting the following VM extensions on Windows and Linux machines.

|Extension |Publisher |Additional information |
|----------|----------|-----------------------|
|CustomScriptExtension |Microsoft.Compute |[Windows Custom Script Extension](../../virtual-machines/extensions/custom-script-windows.md)<br> [Linux Custom Script Extension Version 2](../../virtual-machines/extensions/custom-script-linux.md) |
|DSC |Microsoft.PowerShell|[Windows PowerShell DSC Extension](../../virtual-machines/extensions/dsc-windows.md)<br> [PowerShell DSC Extension for Linux](../../virtual-machines/extensions/dsc-linux.md) |
|MicrosoftMonitoringAgent |Microsoft.EnterpriseCloud.Monitoring |[Log Analytics VM extension for Windows](../../virtual-machines/extensions/oms-windows.md)<br> [Log Analytics VM extension for Linux](../../virtual-machines/extensions/oms-linux.md) |

>[!NOTE]
> VM extension functionality is available only in the following regions:
> * EastUS
> * WestUS2
> * WestEurope 
>
> Ensure you onboard your machine in one of these regions.

VM extensions can be run with the Azure PowerShell, Azure Resource Manager templates, and the Azure portal.

## Prerequisite

This feature depends on the following Azure resource providers in your subscription:

* **Microsoft.HybridCompute**
* **Microsoft.GuestConfiguration**

If they are not already registered, follow the steps under [Register Azure resource providers] (overview.md#register-azure-resource-providers). 

### Connected Machine agent

Verify your machine matches the [supported versions](overview.md#supported-operating-systems) of Windows and Linux operating system for the Azure Connected Machine agent. 

The minimum version of the Connected Machine agent that is supported with this feature is:

* Windows - 0.7.*.*
* Linux - 0.8.*.*

To upgrade your machine to the version of the agent required, see [Upgrade agent](manage-agent.md#upgrading-agent).

## Enable extensions from the portal

VM extensions can be applied your Arc for server (preview) managed machine through the Azure portal.

1. From your browser, go to the [Azure portal](https://aka.ms/arcserver-preview). 

2. In the portal, browse to **Machines - Azure Arc** and select your hybrid machine from the list.

3. Choose **Extensions**, then select **Add**. Choose the extension you want from the list of available extensions and follow the instructions in the wizard.

 
