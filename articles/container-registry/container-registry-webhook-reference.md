---
title: Azure Container Registry webhook schema reference
description: Webhook request JSON payload reference for Azure Container Registry.
services: container-registry
author: mmacy
manager: stevelas

ms.service: container-registry
ms.devlang: na
ms.topic: article
ms.date: 12/02/2017
ms.author: marsma
---

# Azure Container Registry webhook reference

Webhook events are triggered when certain actions are performed against your registry, such as container image `push` and `delete` operations. When a webhook is triggered, Azure Container Registry issues a request to an endpoint you specify. Your endpoint can then process the webhook and act accordingly.

The following sections detail the schema of the webhook requests for supported events. Each section contains one or more example commands that would trigger the webhook event, an example JSON body of the resulting request, and tables that detail the schema objects in the request.

For information about configuring webhooks for your Azure container registry, see [Using Azure Container Registry webhooks](container-registry-webhook.md).

## Push image

Example command that triggers this webhook:

```bash
docker push myregistry.azurecr.io/hello-world:v1
```

Request body:

```JSON
{
  "id": "cb8c3971-9adc-488b-bdd8-43cbb4974ff5",
  "timestamp": "2017-11-17T16:52:01.343145347Z",
  "action": "push",
  "target": {
    "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
    "size": 524,
    "digest": "sha256:80f0d5c8786bb9e621a45ece0db56d11cdc624ad20da9fe62e9d25490f331d7d",
    "length": 524,
    "repository": "hello-world",
    "tag": "v1"
  },
  "request": {
    "id": "3cbb6949-7549-4fa1-86cd-a6d5451dffc7",
    "host": "myregistry.azurecr.io",
    "method": "PUT",
    "useragent": "docker/17.09.0-ce go/go1.8.3 git-commit/afdb6d4 kernel/4.10.0-27-generic os/linux arch/amd64 UpstreamClient(Docker-Client/17.09.0-ce \\(linux\\))"
  }
}
```

|Element name|Type|Description|
|-------------|----------|-----------|
|id|String|The ID of the webhook event.|
|timestamp|DateTime|The time at which the webhook event was triggered.|
|action|String|The action that triggered the webhook event.|
|[target](#target)|Complex Type|The target of the event that triggered the webhook event.|
|[request](#request)|Complex Type|The request that generated the webhook event.|

### target

|Element name|Type|Description|
|------------------|----------|-----------|
|mediaType|String|The MIME type of the referenced object.|
|size|Int32|The number of bytes of the content. Same as Length field.|
|digest|String|The digest of the content, as defined by the Registry V2 HTTP API Specificiation.|
|length|Int32|The number of bytes of the content. Same as Size field.|
|repository|String|The repository name.|
|tag|String|The image tag name.|

### request

|Element name|Type|Description|
|------------------|----------|-----------|
|id|String|The ID of the request that initiated the event.|
|host|String|The externally accessible hostname of the registry instance, as specified by the HTTP host header on incoming requests.|
|method|String|The request method that generated the event.|
|useragent|String|The user agent header of the request.|

## Delete repository or manifest

Example commands that trigger this webhook:

```azurecli
# Delete repository
az acr repository delete -n MyRegistry --repository MyRepository

# Delete manifest
az acr repository delete -n MyRegistry --repository MyRepository --tag MyTag --manifest
```

Request body:

```JSON
{
    "id": "afc359ce-df7f-4e32-bdde-1ff8aa80927b",
    "timestamp": "2017-11-17T16:54:53.657764628Z",
    "action": "delete",
    "target": {
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "digest": "sha256:80f0d5c8786bb9e621a45ece0db56d11cdc624ad20da9fe62e9d25490f331d7d",
      "repository": "hello-world"
    },
    "request": {
      "id": "3d78b540-ab61-4f75-807f-7ca9ecf559b3",
      "host": "myregistry.azurecr.io",
      "method": "DELETE",
      "useragent": "python-requests/2.18.4"
    }
  }
```

|Element name|Type|Description|
|-------------|----------|-----------|
|id|String|The ID of the webhook event.|
|timestamp|DateTime|The time at which the webhook event was triggered.|
|action|String|The action that triggered the webhook event.|
|[target](#delete_target)|Complex Type|The target of the event that triggered the webhook event.|
|[request](#delete_request)|Complex Type|The request that generated the webhook event.|

### <a name="delete_target"></a> target

|Element name|Type|Description|
|------------------|----------|-----------|
|mediaType|String|The MIME type of the referenced object.|
|digest|String|The digest of the content, as defined by the Registry V2 HTTP API Specificiation.|
|repository|String|The repository name.|

### <a name="delete_request"></a> request

|Element name|Type|Description|
|------------------|----------|-----------|
|id|String|The ID of the request that initiated the event.|
|host|String|The externally accessible hostname of the registry instance, as specified by the HTTP host header on incoming requests.|
|method|String|The request method that generated the event.|
|useragent|String|The user agent header of the request.|

## Delete tag

Webhook events are not generated when tags are deleted.

## Next steps

[Using Azure Container Registry webhooks](container-registry-webhook.md)