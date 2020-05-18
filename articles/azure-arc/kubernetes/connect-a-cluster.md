---
title: "Connect a Kubernetes cluster"
services: azure-arc
ms.service: azure-arc
#ms.subservice: azure-arc-kubernetes coming soon
ms.date: 05/19/2020
ms.topic: article
author: mlearned
ms.author: mlearned
description: "Connect a Kubernetes cluster with Azure Arc"
keywords: "Kubernetes, Arc, Azure, K8s, containers"
---

# Connect a Kubernetes cluster

## Pre-requisites

1. Before onboarding a cluster to Azure Arc enabled Kubernetes, make sure that your Kubernetes cluster is up and running, and that you have a kubeconfig, and cluster-admin access.

1. The user or service principal used with `az login` and `az connectedk8s connect` commands must have the 'Read' and 'Write' permissions on 'Microsoft.Kubernetes/connectedClusters' resource type.

    A user or a service principal having 'Contributor' or a more permissive role will be able to connect their Kubernetes clusters as these roles already the include 'Read' and 'Write' permissions on Microsoft.Kubernetes/connectedClusters resource type.

    A more finely scoped built-in role 'Kubernetes Cluster - Azure Arc Onboarding' is also available for [role assignment](https://docs.microsoft.com/cli/azure/role/assignment?view=azure-cli-latest#az-role-assignment-create) so that the assignee has only limited permissions to connect their clusters to Azure, but not to update, delete, or modify any other clusters or resources.

## Network requirements

Azure Arc agents require the following protocols/ports/outbound URLs to function.

* TCP on port 443 --> `https://:443`
* TCP on port 9418 --> `git://:9418`

|     | Endpoint (DNS)                                                                                               | Description                                                                                                                 |
| --- | ------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| 1.  | https://management.azure.com                                                                                 | Required for the agent to connect to Azure and register the cluster.                                                        |
| 2.  | https://eastus.dp.kubernetesconfiguration.azure.com, https://westeurope.dp.kubernetesconfiguration.azure.com | Data plane endpoint for the agent to push status and fetch configuration information.                                       |
| 3.  | https://docker.io                                                                                            | Required to pull container images.                                                                                          |
| 4.  | https://github.com, git://github.com                                                                         | Example GitOps repos are hosted on GitHub. Configuration agent requires connectivity to whichever git endpoint you specify. |
| 5.  | https://login.microsoftonline.com                                                                            | Required to fetch and update ARM tokens.                                                                                    |
| 6.  | https://azurearcfork8s.azurecr.io                                                                            | Required to pull container images for Azure Arc agentry.                                                                    |

## Create a Resource Group

First, create a resource group to hold the connected cluster resource. The referenced region stores metadata for your cluster. Currently supported regions:

* East US
* West Europe

```console
az group create --name AzureArcTest -l EastUS -o table
```

**Output:**

```console
Location    Name
----------  ------------
eastus      AzureArcTest
```

## Connect a cluster

Next, we will connect our Kubernetes cluster to Azure. The workflow for `az connectedk8s connect` is as follows:

1. Verify connectivity to your Kubernetes cluster: via `KUBECONFIG`, `~/.kube/config`, or `--kube-config`
1. Deploy Azure Arc Agents for Kubernetes using Helm 3, into the `azure-arc` namespace

```console
az connectedk8s connect --name AzureArcTest1 --resource-group AzureArcTest
```

__Note: If you receive errors regarding missing provider, or provider not found, double check that your subscriptions have been [onboarded to the private preview, and the required providers have been enabled](./enable-providers.md).__

**Output:**

```console
Command group 'connectedk8s' is in preview. It may be changed/removed in a future release.
Helm release deployment succeeded

{
  "aadProfile": {
    "clientAppId": "",
    "serverAppId": "",
    "tenantId": ""
  },
  "agentPublicKeyCertificate": "...",
  "agentVersion": "0.1.0",
  "id": "/subscriptions/57ac26cf-a9f0-4908-b300-9a4e9a0fb205/resourceGroups/AzureArcTest/providers/Microsoft.Kubernetes/connectedClusters/AzureArcTest1",
  "identity": {
    "principalId": null,
    "tenantId": null,
    "type": "None"
  },
  "kubernetesVersion": "v1.15.0",
  "location": "eastus",
  "name": "AzureArcTest1",
  "resourceGroup": "AzureArcTest",
  "tags": {},
  "totalNodeCount": 1,
  "type": "Microsoft.Kubernetes/connectedClusters"
}
```

## Verify connected cluster

List your connected clusters:

```console
az connectedk8s list -g AzureArcTest -c AzureArcTest1 --cluster-type connectedClusters -o table
```

**Output:**

```console
Command group 'connectedk8s' is in preview. It may be changed/removed in a future release.
Name           Location    ResourceGroup
-------------  ----------  ---------------
AzureArcTest1  eastus      AzureArcTest
```

Azure Arc enabled Kubernetes deploys a few operators into the `azure-arc` namespace. You can view these deployments and pods here:

```console
kubectl -n azure-arc get deploy,po
```

**Output:**

```console
NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/config-agent         1/1     1            1           5h43m
deployment.extensions/controller-manager   1/1     1            1           5h43m

NAME                                      READY   STATUS    RESTARTS   AGE
pod/config-agent-c74f6695f-89hp8          1/1     Running   0          5h43m
pod/controller-manager-7cf48dc76b-m9g74   2/2     Running   0          5h43m
```

## About the Azure Arc agents for Kubernetes

Azure Arc enabled Kubernetes consists of a few agents (operators) that run in your cluster deployed to the `azure-arc` namespace.

* `deploy/config-agent`: watches the connected cluster for source control configuration resources applied on the cluster and updates compliance state
* `deploy/controller-manager`: is an operator of operators and orchestrates interactions between Azure Arc components

## Delete a connected cluster

You can delete a `Microsoft.Kubernetes/connectedCluster` using the CLI or Azure portal.

* CLI
  * az connectedk8s delete
  * Deletes the `Microsoft.Kubernetes/connectedCluster` resource in Azure
  * Deletes any associated `sourceControlConfiguration` resources in Azure
  * Does helm uninstall to remove the agents in the cluster
* Portal
  * Deletes the `Microsoft.Kubernetes/connectedCluster` resource in Azure
  * Deletes any associated `sourceControlConfiguration` resources in Azure
  * Note: To remove the agents in the cluster you need to run az connectedk8s delete or helm uninstall azurearcfork8s.

## Next steps

* [Use GitOps in a connected cluster](./use-gitops-in-connected-cluster.md)
* [Use Azure Policy to govern cluster configuration](./use-azure-policy.md)
