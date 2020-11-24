---
title: Zone-redundant registry for high availability
description: Learn about enabling zone redundancy in Azure Container Registry by creating a container registry or geo-replica in an Azure availability zone. Zone redundancy is a feature of the Premium service tier.
ms.topic: article
ms.date: 11/24/2020
---

# Enable zone redundancy in Azure Container Registry for resiliency and high availability

In addition to [geo-replication](container-registry-geo-replication.md), which replicates registry data across one or more Azure regions to provide availability and reduce latency for regional operations, Azure Container Registry supports optional *zone redundancy*. [Zone redundancy](../availability-zones/az-overview.md#availability-zones) provides resiliency and high availability to a registry or replication resource (replica) in a specific region.

This article shows how to set up a zone-redundant container registry or zone-redundant replica by using an Azure Resource Manager template. 

Zone redundancy is a **preview** feature of the Premium container registry service tier. For information about registry service tiers and limits, see [Azure Container Registry service tiers](container-registry-skus.md).

## Limitations

The following are current limitations of zone redundancy in Azure Container Registry:

* Supported only in the following regions: East US, East US 2, and West US 2.
* Can only be enabled in a registry or a regional replica when you create the resource.
* Can't be disabled after it's enabled. You can't migrate a zone-redundant registry or replica to a non-zone-redundant state, nor the other way around. You also can't downgrade the registry to a different service tier.
* [ACR Tasks](container-registry-tasks-overview.md) configured for this registry aren't zone resilient.
* Can only be enabled by using an Azure Resource Manager template. The feature roadmap includes additional tooling.

## About zone redundancy

Use Azure [availability zones](../availability-zones/az-overview.md) to create a resilient and high availability Azure container registry (or geo-replication) in a specific Azure region. For example, organizations can set up a zone-redundant Azure container registry with other [supported Azure resources](../availability-zones/az-region.md) to meet data residency or other compliance requirements while providing high availability.

Availability zones are unique physical locations within an Azure region. To ensure resiliency, there's a minimum of three separate zones in all enabled regions. Each zone has one or more datacenters equipped with independent power, cooling, and networking. When configured for zone redundancy, a registry (or a registry replica in a different region) is replicated across all availability zones in the region, keeping it available in the event of datacenter failures.

## Create a zone-redundant registry - template

### Create a resource group

If needed, run the [az group create](/cli/az/group#az_group_create) command to create a resource group for the registry in a region that [supports availability zones](../availability-zones/az-region.md) for Azure Container Registry, such as *eastus2*.

```azurecli
az group create --name <resource-group-name> --location <location>
```

### Deploy the template 

You can use the following Resource Manager template to create a zone-redundant, geo-replicated registry. The template by default enables zone redundancy in the registry and an additional regional replica. 

Copy the following contents to a new file and save it using a filename such as `registryZone.json`.

```JSON
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "acrName": {
        "type": "string",
        "defaultValue": "[concat('acr', uniqueString(resourceGroup().id))]",
        "minLength": 5,
        "maxLength": 50,
        "metadata": {
          "description": "Globally unique name of your Azure Container Registry"
        }
      },
      "acrAdminUserEnabled": {
        "type": "bool",
        "defaultValue": false,
        "metadata": {
          "description": "Enable admin user that has push / pull permission to the registry."
        }
      },
      "location": {
        "type": "string",
        "defaultValue": "[resourceGroup().location]",
        "metadata": {
          "description": "Location for registry home replica."
        }
      },
      "acrSku": {
        "type": "string",
        "defaultValue": "Premium",
        "allowedValues": [
          "Premium"
        ],
        "metadata": {
          "description": "Tier of your Azure Container Registry. Geo-replication and zone redundancy require Premium SKU."
        }
      },
      "acrZoneRedundancy": {
        "type": "string",
        "defaultValue": "Enabled",
        "metadata": {
          "description": "Enable zone redundancy of registry's home replica. Requires registry location to support availability zones."
        }
      },
      "acrReplicaLocation": {
        "type": "string",
        "metadata": {
          "description": "Short name for registry replica location."
        }
      },
      "acrReplicaZoneRedundancy": {
        "type": "string",
        "defaultValue": "Enabled",
        "metadata": {
          "description": "Enable zone redundancy of registry replica. Requires replica location to support availability zones."
        }
      }
    },
    "resources": [
      {
        "comments": "Container registry for storing docker images",
        "type": "Microsoft.ContainerRegistry/registries",
        "apiVersion": "2020-11-01-preview",
        "name": "[parameters('acrName')]",
        "location": "[parameters('location')]",
        "sku": {
          "name": "[parameters('acrSku')]",
          "tier": "[parameters('acrSku')]"
        },
        "tags": {
          "displayName": "Container Registry",
          "container.registry": "[parameters('acrName')]"
        },
        "properties": {
          "adminUserEnabled": "[parameters('acrAdminUserEnabled')]",
          "zoneRedundancy": "[parameters('acrZoneRedundancy')]"
        }
      },
      {
        "type": "Microsoft.ContainerRegistry/registries/replications",
        "apiVersion": "2020-11-01-preview",
        "name": "[concat(parameters('acrName'), '/', parameters('acrReplicaLocation'))]",
        "location": "[parameters('acrReplicaLocation')]",
          "dependsOn": [
          "[resourceId('Microsoft.ContainerRegistry/registries/', parameters('acrName'))]"
        ],
        "properties": {
          "zoneRedundancy": "[parameters('acrReplicaZoneRedundancy')]"
        }
      }
    ],
    "outputs": {
      "acrLoginServer": {
        "value": "[reference(resourceId('Microsoft.ContainerRegistry/registries',parameters('acrName')),'2019-12-01-preview').loginServer]",
        "type": "string"
      }
    }
  }
```

Run the following [az deployment group create](/cli/az/deployment#az_group_deployment_create) command to create the registry using the preceding template file. Where indicated, provide:

* a unique registry name, or deploy the template without parameters and it will create a unique name for you
* a location for the replica that supports availability zones, such as *westus2*

```azurecli
az deployment group create \
  --resource-group <resource-group-name> \
  --template-file registryZone.json \
  --parameters registryName=<registry-name> acrReplicaLocation=<replica-location>
```

In the command output, note the `zoneRedundancy` property for the registry and the replica. When enabled, each resource is zone redundant:

```JSON
{
 [...]
"zoneRedundancy": "Enabled",
}
```

## Next steps

* Learn more about [regions that support availability zones](../availability-zones/az-region.md).
* Learn more about building for [reliability](/azure/architecture/framework/resiliency/overview) in Azure.