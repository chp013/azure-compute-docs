---
title: Migrate Standard HDD OS disks by September 08, 2028
description: Learn about the retirement of the capability to use Standard HDD as OS disks for Azure Virtual Machines.
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 08/19/2025
ms.author: rogarana

---

# Migrate your Standard HDD OS disks by September 08, 2028

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs

On September 8, 2028, Standard HDD disks can no longer be used as OS disks. At that time, all Standard HDD disks being used as OS disks will be automatically converted to Standard SSD disks. To avoid potential service disruption, you should convert your Standard HDD disks being used as OS disks to either Standard SSD or Premium SSD disks.

If you're using Standard HDD as OS disks, begin planning a migration now. Generally, Standard SSD provides the closest price to performance ratio as Standard HDD disks. If you need higher performance, migrate to Premium.

## How does this affect me?

- After September 30, 2026, you won't be able to configure or deploy new Standard HDD disks as OS disks
- After September 8, 2028, any existing virtual machines using Standard HDDs as their OS disks will automatically be converted to a Standard SSD of an equivalent size. Workloads on these disks may experience a service disruption if they're not migrated before September 8, 2028.

## What is being retired?

This retirement is only for the ability to use Standard HDD disks as OS disks. All other managed disk types and ephemeral OS disks, aren't affected by this retirement.

## What actions should I take?

To start planning a migration to either Standard SSD or Premium SSD disks, first, make a list of all affected OS disks and VMs.

If you have multiple subscriptions, the easiest way to do this is to use the Disk Storage Center. Add two filters, one for **Storage type** which should be equal to **Standard HDD LRS** and one for **OS type** which should equal to **Linux and Windows**. This'll produce a list of all Standard HDD OS disks across all your subscriptions. The **Owner** column is the name of the virtual machine that uses the listed Standard HDD OS disks.

The Disk Storage Center will still work if you have individual subscriptions. But, if you prefer, you can use the Azure CLI or the Azure PowerShell module to get the same list for an individual subscription. For the Azure PowerShell module, use `Get-AzDisk | Where-Object { $_.Sku.Name -eq "Standard_LRS" -and $_.OsType }` and for the Azure CLI, use `az disk list --query "[?sku.name=='Standard_LRS' && osType!=null]" --output table`.

Once you have a list of Standard HDD OS disks, [convert your Standard HDD OS disks](disks-convert-types.md#change-the-type-of-an-individual-managed-disk) to either Standard SSD or Premium SSD disks.

For technical questions and issues, contact support.


## What resources are available for this migration?

[Microsoft Q&A](/answers/topics/azure-virtual-machines-migration.html): Microsoft and community support for migration.