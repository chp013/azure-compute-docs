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

Start planning a migration to either Standard SSD or Premium SSD disks.

1. Make a list of all affected OS disks and VMs
    
    > [!NOTE]
    > Disks with the **Storage type** set to **Standard HDD LRS** and **OS type** set to **Linux and Windows** on the [Azure portal's Disk Storage Center](https://ms.portal.azure.com/#view/Microsoft_Azure_StorageHub/StorageHub.MenuView/~/DisksBrowse) are all the affected disks within the subscription.
    >     
    > The Owner column indicates the name of the virtual machine that uses the listed Standard HDD OS disks.

1. Once you have a list of Standard HDD OS disks, [convert your Standard HDD OS disks](disks-convert-types.md#change-the-type-of-an-individual-managed-disk) to either Standard SSD or Premium SSD disks.
1. For technical questions and issues, contact support.


## What resources are available for this migration?

[Microsoft Q&A](/answers/topics/azure-virtual-machines-migration.html): Microsoft and community support for migration.