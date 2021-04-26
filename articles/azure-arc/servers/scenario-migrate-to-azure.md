---
title: Migrate Azure Arc enabled server to Azure
description: Learn how to migrate your Azure Arc enabled servers running on-premises or other cloud environment to Azure.
ms.date: 04/26/201
ms.topic: conceptual
---

# Migrate your on-premises or other cloud Arc enabled server to Azure

This article is intended to help you plan and successfully migrate your on-premises server or virtual machine managed by Azure Arc enabled servers to Azure. By following these steps, you are able to transition management from Arc enabled servers based on the supported VM extensions installed and Azure services based on it's Arc server resource identity.

Before performing these steps, review the Azure Migrate [Prepare on-premises machines for migration to Azure](../../migrate/prepare-for-migration.md) article to understand requirements how to prepare for using Azure Migrate.

In this article you:

* Inventory Azure Arc enabled servers supported VM extensions installed
* Uninstall all VM extensions from the Arc enabled server
* Identify Azure services the resource ID of the Arc enabled server is registered with, and prepare to update those Azure services to use the new Azure VM identity after migration.
* Document the Azure role-based access control (Azure RBAC) access rights granted to the Arc enabled server resource to maintain who has access to the resource after it has been migrated to an Azure VM.
* Delete the Arc enabled server resource identity from Azure and remove the Arc enabled server agent.
* Install the Azure guest agent
* Migrate the server or VM to Azure

## Inventory and remove VM extensions installed and remove extensions

To inventory the VM extensions installed on your Arc enabled server, you can list them using the Azure CLI or with Azure PowerShell.

With Azure PowerShell, use the [Get-AzConnectedMachineExtension](/powershell/module/az.connectedmachine/get-azconnectedmachineextension) command with the `-MachineName` and `-ResourceGroupName` parameters.

With the Azure CLI, use the [az connectedmachine extension list](/cli/azure/ext/connectedmachine/connectedmachine/extension#ext_connectedmachine_az_connectedmachine_extension_list) command with the `--machine-name` and `--resource-group` parameters. By default, the output of Azure CLI commands is in JSON (JavaScript Object Notation). To change the default output to a list or table, for example, use [az configure --output](/cli/azure/reference-index). You can also add `--output` to any command for a one time change in output format.

After identifying which VM extensions are deployed, if the Log Analytics VM extension or Dependency agent VM extension were deployed using Azure Policy and the [VM insights initiative](../../azure-monitor/vm/vminsights-enable-policy.md), it is necessary to [create an exclusion](../../governance/policy/tutorials/create-and-manage.md#remove-a-non-compliant-or-denied-resource-from-the-scope-with-an-exclusion) to prevent re-evaluation and deployment of the extensions on the Arc enabled server.

## Inventory Azure services 

To inventory the Azure services that 

## Review access rights granted to the Arc enabled server resource

To list role assignments for the Arc enabled servers resource, you can use [Azure PowerShell](../../role-based-access-control/role-assignments-list-powershell.md#list-role-assignments-for-a-resource) 


## Disconnect from Azure Arc and uninstall agent

Disconnect the machine from Azure Arc using one of the following methods:

    * Running `azcmagent disconnect` command on the machine or server.

    * From the selected registered Arc enabled server in the Azure portal by selecting **Delete** from the top bar.

    * Using the [Azure CLI](../../azure-resource-manager/management/delete-resource-group.md?tabs=azure-cli#delete-resource) or [Azure PowerShell](../../azure-resource-manager/management/delete-resource-group.md?tabs=azure-powershell#delete-resource). For the`ResourceType` parameter use `Microsoft.HybridCompute/machines`.

Remove the Azure Arc enabled servers Windows or Linux agent following the [Remove agent](manage-agent.md#remove-the-agent) steps.

## Install the Azure Guest Agent

The VM that is migrated to Azure from on-premises doesn't have the Linux or Windows Azure Guest Agent installed. In these scenarios, you have to manually install the VM agent. For more information about how to install the VM Agent, see [Azure Virtual Machine Windows Agent Overview](../../virtual-machines/extensions/agent-windows.md) or [Azure Virtual Machine Linux Agent Overview](../../virtual-machines/extensions/agent-linux.md).

## Migrate server or machine to Azure

Before proceeding with the migration with Azure Migration, review the [Prepare on-premises machines for migration to Azure](../../migrate/prepare-for-migration.md) article to learn about requirements necessary to use Azure Migrate. To complete the migration to Azure, review the Azure Migrate [migration options](../../migrate/prepare-for-migration.md#next-steps) based on your environment.