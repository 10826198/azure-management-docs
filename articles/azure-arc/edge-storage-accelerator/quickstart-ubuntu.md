---
title: Quickstart - create a single-node Edge Storage Accelerator (ESA) cluster using K3s or AKS Edge Essentials
description: Learn how to create a single-node Edge Storage Accelerator (ESA) using a K3s cluster.
author: sethmanheim
ms.author: sethm
ms.topic: how-to
ms.date: 03/19/2024

---

# Quickstart: create Edge Storage Accelerator single-node cluster

This quickstart guide shows how to set up a single-node Edge Storage Accelerator (ESA) using a [K3s](https://docs.k3s.io/) or [AKS Edge Essentials](/azure/aks/hybrid/aks-edge-overview), based on the instructions provided in the Edge Storage Accelerator documentation.

## Prerequisites

Before you begin, ensure you have the following:

- A machine capable of running K3s, meeting the minimum system requirements.
- Basic understanding of Kubernetes concepts.

## Create a K3s cluster

Follow these steps to create a single-node Edge Storage Accelerator cluster using K3s.

### Step 1: Create and configure a K3s cluster on Ubuntu

Follow the [Azure IoT Operations K3s installation instructions](/azure/iot-operations/get-started/quickstart-deploy?tabs=linux#connect-a-kubernetes-cluster-to-azure-arc) to install K3s on your machine.

### Step 2: Prepare Linux using a single-node cluster

See [Prepare Linux using a single-node cluster](single-node-cluster.md) to set up a single-node K3s cluster.

### Step 3: Install Edge Storage Accelerator

Follow the instructions in [Install Edge Storage Accelerator](install-edge-storage-accelerator.md) to install Edge Storage Accelerator on your single-node Ubuntu K3s cluster.

### Step 4: Create Persistent Volume (PV)

Create a Persistent Volume (PV) by following the steps in [Create a PV](create-pv.md).

### Step 5: Create Persistent Volume Claim (PVC)

Create a Persistent Volume Claim (PVC) to bind with the PV created in the previous step. See [Create a PVC](create-pvc.md) for guidance.

### Step 6: Attach application to Edge Storage

Follow the instructions in [Edge Storage Accelerator: Attach your app](attach-app.md) to attach your application.

## Next steps

- [K3s Documentation](https://k3s.io/)
- [Azure IoT Operations K3s installation instructions](/azure/iot-operations/get-started/quickstart-deploy?tabs=linux#connect-a-kubernetes-cluster-to-azure-arc)
- [Azure Arc documentation](/azure/azure-arc/)
