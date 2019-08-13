---
title: Automatically remove image resources in Azure Container Registry
description: Use a purge command to delete multiple tags and manifests from an Azure container registry based on age and a tag filter, and optionally schedule purge operations.
services: container-registry
author: dlepow
manager: gwallace

ms.service: container-registry
ms.topic: article
ms.date: 08/13/2019
ms.author: danlep
---

# Automatically purge images from an Azure container registry

When you use an Azure container registry as part of a development workflow, the registry can quickly fill up with images or other artifacts that aren't needed after a short period. You might want to delete all tags that are older than a certain duration or match a specified name filter. To delete multiple artifacts quickly, this article introduces the `acr purge` command you can run as an on-demand or [scheduled](container-registry-tasks-scheduled.md) ACR Task. 

The `acr purge` command is currently distributed in a public container image available from Microsoft Container Registry.

You can use the Azure Cloud Shell or a local installation of the Azure CLI to run the ACR task examples in this article. If you'd like to use it locally, version 2.0.69 or later is required. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI][azure-cli-install]. 

> [!IMPORTANT]
> This feature is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use][terms-of-use]. Some aspects of this feature may change prior to general availability (GA).

> [!WARNING]
> Use the `acr purge` command with caution--deleted image data is UNRECOVERABLE. If you have systems that pull images by manifest digest (as opposed to image name), you should not purge untagged images. Deleting untagged images will prevent those systems from pulling the images from your registry. Instead of pulling by manifest, consider adopting a *unique tagging* scheme, a [recommended best practice](container-registry-image-tag-version.md).

If you want to delete single image tags or manifests using Azure CLI commands, see [Delete container images in Azure Container Registry](container-registry-delete.md).

## Use the purge command

The `acr purge` container command deletes images by tag in a repository that match a name filter and that are older than a specified duration. By default, only tag references are deleted, not the underlying [manifests](container-registry-concepts.md#manifest) and layer data. The command has an option to also delete manifests. 

> [!NOTE]
> `acr purge` does not delete an image tag or repository where the `write-enabled` attribute is set to `false`. For information, see [Lock a container image in an Azure container registry](container-registry-image-lock.md).

`acr purge` is designed to run as a container command in an [ACR Task](container-registry-tasks-overview.md), so that it authenticates automatically with the registry where the task runs. 

At a minimum, specify the following when you run `acr purge`:

* `--registry` - The Azure container registry where you run the command. 
* `--filter` - A repository and a *regular expression* to filter tags in the repository. Examples: `--filter "hello-world:.*"` matches all tags in the `hello-world` repository, and `--filter "hello-world:^1.*"` matches tags beginning with `1`. Pass multiple `--filter` parameters to purge multiple repositories.
* `--ago` - An expression in Go duration format to indicate a duration beyond which images are deleted. For example, `--ago 2d3h6m` selects all images last modified more than 2 days, 3 hours, and 6 minutes ago. 

`acr purge` supports several optional parameters. The following two are used in examples in this article:

* `--untagged` - Specifies that manifests that don't have associated tags (*untagged manifests*) are deleted.
* `--dry-run` - Specifies that no data is deleted, but the output is the same as if the command is run without this flag. This parameter is useful for testing a purge command to make sure it does not inadvertently delete data you intend to preserve.

For additional parameters, run `acr purge --help`. 

`acr purge` supports other features of ACR Tasks commands including [run variables](container-registry-tasks-reference-yaml.md#run-variables) and [task run logs](container-registry-tasks-overview.md#view-task-logs) that are streamed and also saved for later retrieval.

### Run in an on-demand task

The following example uses the [az acr run][az-acr-run] command to run the `purge` command on-demand. This example deletes all image tags and manifests in the `hello-world` repository in *myregistry* that were modified more than 1 day ago. The task runs without a source context.

```azurecli
az acr run \
  --cmd "mcr.microsoft.com/acr/acr-cli:0.1 purge --registry {{.Run.Registry}} --filter "hello-world:.*" --untagged --ago 1d" \
  --registry myregistry \
  /dev/null
```

### Run in a scheduled task

The following example uses the [az acr task create][az-acr-task-create] command to create a daily [scheduled ACR task](container-registry-tasks-scheduled.md). The task purges tags modified more than 7 days ago in the `hello-world` repository. The task runs without a source context.

```azurecli
az acr task create --name purgeTask \
  --cmd "mcr.microsoft.com/acr/acr-cli:0.1 purge --registry {{.Run.Registry}} --filter "hello-world:.*"  --ago 7d" \
  --context /dev/null \
  --schedule "0 0 * * *" \
  --registry myregistry
```

Run the [az acr task show][az-acr-task-show] command to see that the timer trigger is configured.

### Purge large numbers of tags and manifests

Purging a large number of tags and manifests could take several minutes or longer. To purge thousands of tags and manifests, the on-demand or scheduled task command might need to run longer than the default [timeout time](container-registry-tasks-reference-yaml.md#cmd) of 600 seconds (10 minutes). In this case, pass the `--timeout` parameter to set a larger value. 

For example, the following on-demand task times out after 3600 seconds (1 hour):

```azurecli
az acr run \
  --cmd "mcr.microsoft.com/acr/acr-cli:0.1 purge --registry {{.Run.Registry}} --filter "hello-world:.*" --untagged --ago 1d" \
  --registry myregistry \
  --timeout 3600 \
  /dev/null
```

## Example: Scheduled purge of multiple repositories in a registry

This example walks through using `acr purge` to periodically clean up multiple repositories in a registry. For example, you might have a development pipeline that pushes images to the `devimage1` and `devimage2` repositories. You periodically import development images into a production repository for your deployments, so you no longer need the development images. On a weekly basis, you purge the `devimage1` and `devimage2` repositories, in preparation for the coming week's work.

### Preview the purge

Before deleting data, we recommend running an on-demand purge task using the `--dry-run` parameter. This option allows you to see the tags and manifests that the command will purge, without removing any data. 

In the following example, the filter in each repository selects all tags. The `--ago 0d` parameter matches images of all ages in the repositories that match the filters. Modify the selection criteria as needed for your scenario. The `--untagged` parameter indicates to delete manifests in addition to tags. 

```azurecli
az acr run \
  --cmd "mcr.microsoft.com/acr/acr-cli:0.1 purge --registry {{.Run.Registry}} --filter "devimage1:.*" --filter "devimage2:.*" --ago 0d --untagged --dry-run" \
  --registry myregistry \
  /dev/null
```

Review the command output to see the tags and manifests that match the selection parameters. Because the command is run with `--dry-run`, no data is deleted.

Sample output:

```console
[...]
Deleting tags for repository: devimage1
myregistry.azurecr.io/devimage1:232889b
myregistry.azurecr.io/devimage1:a21776a
Deleting manifests for repository: devimage1
myregistry.azurecr.io/devimage1@sha256:81b6f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e788b
myregistry.azurecr.io/devimage1@sha256:3ded859790e68bd02791a972ab0bae727231dc8746f233a7949e40f8ea90c8b3
Deleting tags for repository: devimage2
myregistry.azurecr.io/devimage2:5e788ba
myregistry.azurecr.io/devimage2:f336b7c
Deleting manifests for repository: devimage2
myregistry.azurecr.io/devimage2@sha256:8d2527cde610e1715ad095cb12bc7ed169b60c495e5428eefdf336b7cb7c0371
myregistry.azurecr.io/devimage2@sha256:ca86b078f89607bc03ded859790e68bd02791a972ab0bae727231dc8746f233a

Number of deleted tags: 4
Number of deleted manifests: 4
[...]
```

### Schedule the purge

After you've verified the dry run, create a scheduled task to automate the purge. The following example schedules a weekly task on Sunday at 1:00 UTC to run the previous purge command:

```azurecli
az acr task create --name weeklyPurgeTask \
  --cmd "mcr.microsoft.com/acr/acr-cli:0.1 purge --registry {{.Run.Registry}} --filter "devimage1:.*" --filter "devimage2:.*" --ago 0d --untagged" \
  --context /dev/null \
  --schedule "0 1 * * Sun" \
  --registry myregistry
```

Run the [az acr task show][az-acr-task-show] command to see that the timer trigger is configured.

## Next steps

Learn about other options to [delete image data](container-registry-delete.md) in Azure Container Registry.

For more information about image storage, see [Container image storage in Azure Container Registry](container-registry-storage.md).

<!-- LINKS - External -->

[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
[az-acr-run]: /cli/azure/acr#az-acr-run
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-show]: /cli/azure/acr/task#az-acr-task-show

