---
title: Manage fault domains in Azure Virtual Machine Scale Sets
description: Learn how to choose the right number of FDs while creating a Virtual Machine Scale Set.
author: mimckitt
ms.author: mimckitt
ms.topic: concept-article
ms.service: azure-virtual-machine-scale-sets
ms.subservice: availability
ms.date: 06/14/2024
ms.reviewer: jushiman
ms.custom: mimckitt, devx-track-azurecli

# Customer intent: As an IT administrator managing cloud resources, I want to select the appropriate number of fault domains for my Virtual Machine Scale Set, so that I can ensure high availability and minimize the risk of downtime due to hardware failures.
---
# Choosing the right number of fault domains for Virtual Machine Scale Set

The fault domain (FD) configuration for Virtual Machine Scale Sets varies depending on the orchestration mode:

## Uniform Orchestration Mode
Virtual Machine Scale Sets with Uniform orchestration are created with five fault domains by default in Azure regions with no zones. For regions that support zonal deployment of Virtual Machine Scale Sets and this option is selected, the default value of the fault domain count is `1` for each of the zones. A `platformFaultDomainCount` of `1` in this case implies that the virtual machine (VM) instances belonging to the scale set are spread across many racks on a best effort basis.

You can also consider aligning the number of scale set fault domains with the number of Managed Disks fault domains. This alignment can help prevent loss of quorum if an entire Managed Disks fault domain goes down. The FD count can be set to less than or equal to the number of Managed Disks fault domains available in each of the regions. Refer to the [documentation](../virtual-machines/availability-set-overview.md) to learn about the number of Managed Disks fault domains by region.

## Flexible Orchestration Mode
Virtual Machine Scale Sets with Flexible orchestration support different fault domain configurations depending on the deployment type:

- **Regional deployments**: Support fault domain counts of `1`, `2`, or `3`.
- **Zonal deployments**: Support only fault domain count of `1`.

For zonal deployments, a `platformFaultDomainCount` of `1` implies that the VM instances belonging to the scale set are spread across many racks within the zone on a best effort basis. Fault domain and update domain information isn't exposed in the Instance View REST API response for Flexible scale sets, unlike Uniform orchestration mode.

### Instance View API Behavior
When using the [Virtual Machines - Instance View REST API](/rest/api/compute/virtualmachines/instanceview) with Flexible orchestration mode:
- The response doesn't include `faultDomain` and `updateDomain` properties
- This is by design and differs from Uniform orchestration mode where these properties are returned
- For regional deployments with multiple fault domains, the VM instances are distributed across the configured fault domains, but this information isn't exposed through the API
- For zonal deployments, the VM instances are distributed across multiple racks within the zone

## REST API
For Uniform orchestration mode, you can set the property `properties.platformFaultDomainCount` to `1`, `2`, or `3`. If not set, the property will default to `1`. For Flexible orchestration mode, you can set this property to `1`, `2`, or `3` for regional deployments, and only `1` is supported for zonal deployments. Refer to the REST API documentation for [Virtual Machine Scale Sets](/rest/api/compute/virtualmachinescalesets/createorupdate).

## Azure CLI

> [!IMPORTANT]
>Starting November 2023, VM scale sets created using PowerShell and Azure CLI will default to Flexible Orchestration Mode if no orchestration mode is specified. For more information about this change and what actions you should take, go to [Breaking Change for VMSS PowerShell/CLI Customers - Microsoft Community Hub](https://techcommunity.microsoft.com/t5/azure-compute-blog/breaking-change-for-vmss-powershell-cli-customers/ba-p/3818295)

For Uniform orchestration mode, you can set the parameter `--platform-fault-domain-count` to 1, 2, or 3 (default of 3 if not specified). For Flexible orchestration mode, you can set this parameter to 1, 2, or 3 for regional deployments, but only 1 is supported for zonal deployments. Refer to the Azure CLI documentation for [Virtual Machine Scale Sets](/cli/azure/vmss#az-vmss-create).

### Example for Uniform Orchestration Mode

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --orchestration-mode Uniform \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --platform-fault-domain-count 3\
  --generate-ssh-keys
```

### Example for Flexible Orchestration Mode

#### Regional deployment with multiple fault domains
```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --orchestration-mode Flexible \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --platform-fault-domain-count 3 \
  --generate-ssh-keys
```

#### Zonal deployment 
```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --orchestration-mode Flexible \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --zones 1 \
  --generate-ssh-keys
```

> [!NOTE]
> For zonal Flexible VMSS deployments, the fault domain count is automatically set to 1 and can't be configured to a higher value.

It takes a few minutes to create and configure all the scale set resources and VMs.

## Next steps
- Learn more about [availability and redundancy features](../virtual-machines/availability.md) for Azure environments.
