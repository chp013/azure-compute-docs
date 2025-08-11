---
title: Add a data disk to Linux VM using the Azure CLI
description: Learn to add a persistent data disk to your Linux VM with the Azure CLI
author: roygara
ms.service: azure-disk-storage
ms.custom: devx-track-azurecli, linux-related-content
ms.collection: linux
ms.topic: how-to
ms.date: 12/09/2024
ms.author: rogarana
# Customer intent: As a system administrator managing Linux VMs, I want to attach and configure a persistent data disk using the command line, so that I can ensure data retention and improve performance for my applications.
---

# Add a disk to a Linux VM

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets

This article shows you how to attach a persistent disk to your VM so that you can preserve your data - even if your VM is reprovisioned due to maintenance or resizing.

## Attach a new disk to a VM

If you want to add a new, empty data disk on your VM, use the [az vm disk attach](/cli/azure/vm/disk) command with the `--new` parameter. If your VM is in an Availability Zone, the disk is automatically created in the same zone as the VM. For more information, see [Overview of Availability Zones](/azure/reliability/availability-zones-overview). The following example creates a disk named *myDataDisk* that is 50 Gb in size:

```azurecli
az vm disk attach \
   -g myResourceGroup \
   --vm-name myVM \
   --name myDataDisk \
   --new \
   --size-gb 50
```

### Lower latency

In select regions, the disk attach latency has been reduced, so you'll see an improvement of up to 15%. This is useful if you have planned/unplanned failovers between VMs, you're scaling your workload, or are running a high scale stateful workload such as Azure Kubernetes Service. However, this improvement is limited to the explicit disk attach command, `az vm disk attach`. You won't see the performance improvement if you call a command that may implicitly perform an attach, like `az vm update`. You don't need to take any action other than calling the explicit attach command to see this improvement.

[!INCLUDE [virtual-machines-disks-fast-attach-detach-regions](../includes/virtual-machines-disks-fast-attach-detach-regions.md)]

## Attach an existing disk

To attach an existing disk, find the disk ID and pass the ID to the [az vm disk attach](/cli/azure/vm/disk) command. The following example queries for a disk named *myDataDisk* in *myResourceGroup*, then attaches it to the VM named *myVM*:

```azurecli
diskId=$(az disk show -g myResourceGroup -n myDataDisk --query 'id' -o tsv)

az vm disk attach -g myResourceGroup --vm-name myVM --name $diskId
```

## Identifying disks

Azure Linux virtual machines use different disk interfaces depending on the VM SKU and generation.
- **Older VM SKUs**: Use SCSI interface for disk management
- **Newer VM SKUs (v5, v6, and newer)**: May use NVMe interface for improved performance

For more information about SCSI vs NVMe differences, see [SCSI to NVMe conversion](/azure/virtual-machines/nvme-linux#scsi-vs-nvme). 

### Connect to the virtual machine

To identify disks associated with your Linux virtual machine (VM), SSH into the VM. For more information, see [How to use SSH with Linux on Azure](/azure/virtual-machines/linux/mac-create-ssh-keys). The following example connects to a VM with the public IP address of 10.123.123.25 with the username azureuser:

```bash
ssh azureuser@10.123.123.25
```
> [!NOTE]
> Before identifying specific disks, determine whether your VM uses SCSI, NVMe, or a combination of both interfaces.

### Identifying SCSi controlled disks

Once you connect to your VM, find the disk. In this example, we're using `lsblk` to list SCSI disks.

```bash
lsblk -o NAME,HCTL,SIZE,MOUNTPOINT | grep -i "sd"
```

The output is similar to the following example:

```
sda     0:0:0:0      30G
├─sda1             29.9G /
├─sda14               4M
└─sda15             106M /boot/efi
sdb     1:0:1:0      14G
└─sdb1               14G /mnt
sdc     3:0:0:0      50G
```
Here, `sdc` is the data disk that we have added. If you have added multiple disks, and aren't sure which disk it's based on size alone, you can go to the VM page in the portal, select **Disks**, and check the LUN number for the disk under **Data disks**. Compare the LUN number from the portal to the last number of the HCTL portion of the output, which is the LUN. 

Another option is to list the contents of the `/dev/disk/azure/scsi1` directory:

```bash
ls -l /dev/disk/azure/scsi1
```

The output should be similar to the following example:

```
lrwxrwxrwx 1 root root 12 Mar 28 19:41 lun0 -> ../../../sdc
```

### Identifying NVMe controlled disks

```bash
lsblk -o NAME,TYPE,SIZE,MOUNTPOINT | grep nvme
```

The output is similar to the following example:
```
nvme0n1     disk    30G
├─nvme0n1p1 part   29.9G /
├─nvme0n1p14 part   4M
└─nvme0n1p15 part  106M  /boot/efi
nvme1n1     disk   50G
nvme0n2     disk   50G
```

### Identifying disks using azure-vm-utils

The [azure-vm-utils](https://github.com/Azure/azure-vm-utils) package provides essential utilities and udev rules to optimize the Linux experience on Azure virtual machines. This package consolidates device management tools for SCSI, NVMe, MANA, and Mellanox devices, making disk identification and management more reliable and consistent across different VM configurations.

See [azure-vm-utils](azure-virtualmachine-utilities.md) for additonal infromation including instructions for instllating the package incase it is not baked into the image.

Usee the following command to list NVMe controlled disks on the VM:
```bash
sudo azure-nvme-id
```

The output is similar to the following example:
```
/dev/nvme0n1: type=os
/dev/nvme0n2: type=data, lun=0
/dev/nvme1n1: type=local, index=1, name=nvme-50G-1
```

## Next Steps

- Format and mount the disks based on your requirements and use case, review isntructions for fromatting and mounting [temp disks](formatting-mounting-temp-resource-disks-linux.md) and [remote disks](formatting-mounting-remote-disks-linux.md).