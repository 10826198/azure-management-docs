---
title:  Overview of the Connected Machine Windows agent
description: This article provides a detailed overview of the Azure Arc for servers agent available which support monitoring virtual machines hosted in hybrid environments.
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-servers
author: mgoedtel
ms.author: magoedte
ms.date: 04/27/2020
ms.topic: conceptual
---

# Overview of Azure Arc for servers agent

The Azure Arc for servers Connected Machine agent enables you to manage your Windows and Linux machines hosted outside of Azure on your corporate network or other cloud provider. This article provides a detailed overview of the agent, system and network requirements, and the different deployment methods.

## Download agents

You can download the Azure Connected Machine agent package for Windows and Linux from the locations listed below.

- [Windows agent Windows Installer package](https://aka.ms/AzureConnectedMachineAgent) from the Microsoft Download Center.
- Linux agent package is distributed from Microsoft's [package repository](https://packages.microsoft.com/) using the preferred package format for the distribution (.RPM or .DEB).

>[!NOTE]
>During this preview, only one package has been released, which is suitable for Ubuntu 16.04 or 18.04.

The Azure Connected Machine agent for Windows and Linux can be upgraded to the latest release manually or automatically depending on your requirements. For more information, see [here](manage-agent.md).

## Windows agent installation details

The Connected Machine agent for Windows can be installed by using one of the following three methods:

* Double-click the file `AzureConnectedMachineAgent.msi`.
* Manually by running the Windows Installer package `AzureConnectedMachineAgent.msi` from the Command shell.
* From a PowerShell session using the scripted method.

After installing the Connected Machine agent for Windows, the following additional system-wide configuration changes are applied. These artifacts are removed when the agent is uninstalled.

* During installation of the agent, its supporting files are installed in `C:\Program Files\AzureConnectedMachineAgent` by default. Its configuration files are installed in `%ProgramData%\AzureConnectedMachineAgent`.

* The following Windows services are created on the target machine during installation of the agent.

    |Service name | Display name | Process name | Description |
    |-------------|--------------|--------------|-------------|
    | himds | Azure Hybrid Instance Metadata Service | himds.exe | This service implements the Azure Instance Metadata service (IMDS) to track the machine.|
    | DscService | Guest Configuration Service | dsc_service.exe | This is the Desired State Configuration (DSC v2) codebase used inside Azure to implement In-Guest Policy.|

* There are four log files available for troubleshooting. They are described in the following table.

    |Log | Location | Description |
    |----|----------|-------------|
    |himds.log | %ProgramData%\AzureConnectedMachineAgent\Log | Records details of the agents (himds) service and interaction with Azure. |
    |azcmagent.log | %ProgramData%\AzureConnectedMachineAgent\Log | Contains the output of the azcmagent tool commands, when the verbose (-v) argument is used. |
    |gc_agent.log | %ProgramData%\GuestConfig\gc_agent_logs | Records details of the DSC service activity, in particular the connectivity between the himds service and Azure Policy.|
    |gc_agent_telemetry.txt | %ProgramData%\GuestConfig\gc_agent_logs | Records details about DSC service telemetry / verbose logging.|

If the agent fails to start after setup is finished, check the %ProgramData%\AzureConnectedMachineAgentAgent\logs\himds.log for detailed error information.

## Linux agent installation details

## Prerequisites

### Supported operating systems

The following versions of the Windows and Linux operating system are officially supported for the Azure Connected Machine agent: 

- Windows Server 2012 R2 and higher (including Windows Server Core)
- Ubuntu 16.04 and 18.04
- CentOS Linux 7
- SUSE Linux Enterprise Server (SLES) 15
- Red Hat Enterprise Linux (RHEL) 7
- Amazon Linux 2

>[!NOTE]
>This preview release of the Connected Machine agent for Windows only supports Windows Server configured to use the English language.
>

### Required permissions

- To onboard machines, you are a member of the **Azure Connected Machine Onboarding** role.

- To read, modify, re-onboard, and delete a machine, you are a member of the **Azure Connected Machine Resource Administrator** role. 

### Azure subscription and service limits

Before configuring your machines with Azure Arc for servers (preview), you should review the Azure Resource Manager [subscription limits](../../azure-resource-manager/management/azure-subscription-service-limits.md#subscription-limits) and [resource group limits](../../azure-resource-manager/management/azure-subscription-service-limits.md#resource-group-limits) to plan for the number of machines to be connected.

## TLS 1.2 protocol

To ensure the security of data in transit to Azure, we strongly encourage you to configure machine to use Transport Layer Security (TLS) 1.2. Older versions of TLS/Secure Sockets Layer (SSL) have been found to be vulnerable and while they still currently work to allow backwards compatibility, they are **not recommended**. 

|Platform/Language | Support | More Information |
| --- | --- | --- |
|Linux | Linux distributions tend to rely on [OpenSSL](https://www.openssl.org) for TLS 1.2 support. | Check the [OpenSSL Changelog](https://www.openssl.org/news/changelog.html) to confirm your version of OpenSSL is supported.|
| Windows Server 2012 R2 and higher | Supported, and enabled by default. | To confirm that you are still using the [default settings](https://docs.microsoft.com/windows-server/security/tls/tls-registry-settings).|

### Networking Configuration

The Connected Machine agent for Linux and Windows communicates outbound securely to Azure Arc over TCP port 443. If the machine connects through a firewall or proxy server to communicate over the Internet, review requirements below to understand the network configuration requirements.

If outbound connectivity is restricted by your firewall or proxy server, make sure the URLs listed below are not blocked. If you only allow the IP ranges or domain names required for the agent to communicate with the service, you must also allow access to the following Service Tags and URLs.

Service Tags:

- AzureActiveDirectory
- AzureTrafficManager

URLs:

| Agent resource | Description |
|---------|---------|
|management.azure.com|Azure Resource Manager|
|login.windows.net|Azure Active Directory|
|dc.services.visualstudio.com|Application Insights|
|agentserviceapi.azure-automation.net|Guest Configuration|
|*-agentservice-prod-1.azure-automation.net|Guest Configuration|
|*.his.hybridcompute.azure-automation.net|Hybrid Identity Service|

For a list of IP addresses for each service tag/region, see the JSON file - [Azure IP Ranges and Service Tags – Public Cloud](https://www.microsoft.com/download/details.aspx?id=56519). Microsoft publishes weekly updates containing each Azure Service and the IP ranges it uses. For more information, review [Service tags](https://docs.microsoft.com/azure/virtual-network/security-overview#service-tags).

The URLs in the previous table are required in addition to the Service Tag IP address range information because the majority of services do not currently have a Service Tag registration. As such, the IP addresses are subject to change. If IP address ranges are required for your firewall configuration, then the **AzureCloud** Service Tag should be used to allow access to all Azure services. Do not disable security monitoring or inspection of these URLs, allow them as you would other Internet traffic.

### Register Azure resource providers

Azure Arc for servers (preview) depends on the following Azure resource providers in your subscription in order to use this service:

- **Microsoft.HybridCompute**
- **Microsoft.GuestConfiguration**

If they are not registered, you can register them using the following commands:

Azure PowerShell:

```azurepowershell-interactive
Login-AzAccount
Set-AzContext -SubscriptionId [subscription you want to onboard]
Register-AzResourceProvider -ProviderNamespace Microsoft.HybridCompute
Register-AzResourceProvider -ProviderNamespace Microsoft.GuestConfiguration
```

Azure CLI:

```azurecli-interactive
az account set --subscription "{Your Subscription Name}"
az provider register --namespace 'Microsoft.HybridCompute'
az provider register --namespace 'Microsoft.GuestConfiguration'
```

You can also register the resource providers in the Azure portal by following the steps under [Azure portal](../../azure-resource-manager/management/resource-providers-and-types.md#azure-portal).


## Installation and configuration

Connecting machines in your hybrid environment directly with Azure can be accomplished using different methods depending on your requirements. The following table highlights each method to determine which works best for your organization.

| Method | Description |
|--------|-------------|
| Interactively | Manually install the agent on a single or small number of machines following the steps in [Connect machines from Azure portal](onboard-portal.md).<br> From the Azure portal, you can generate a script and execute it on the machine to automate the install and configuration steps of the agent.|
| At scale | Install and configure the agent for multiple machines following the [Connect machines using a Service Principal](onboard-service-principal.md).<br> This method creates a service principal to connect machines non-interactively.|
| At scale | Install and configure the agent for multiple machines following the method [Using Windows PowerShell DSC](onboard-dsc.md).<br> This method uses a service principal to connect machines non-interactively with PowerShell DSC. |

## Next steps

To begin evaluating Azure Arc for servers (preview), follow the article [Connect hybrid machines to Azure from the Azure portal](onboard-portal.md).