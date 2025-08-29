---
title: Managing temporary and resource disks on Linux VMs
description: Learn to format and mount temporary and resource disks on Azure Linux VMs with both SCSI and NVMe interfaces
author: vamckms
ms.service: azure-disk-storage
ms.custom: devx-track-azurecli, linux-related-content
ms.collection: linux
ms.topic: how-to
ms.date: 07/22/2025
ms.author: vakavuru
---

# Managing temporary and resource disks

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

This article covers how to format and mount temporary (local) and resource disks on Azure Linux VMs. These disks provide local storage that isn't persistent and might use either SCSI or NVMe interfaces depending on your VM SKU.

## Understanding temporary and resource disks

- **Temporary/Local disks**: Fast local storage for temporary data, not persistent across VM deallocations
- **Resource disks**: Traditional temporary storage, usually SCSI-based on older VMs
- **NVMe local disks**: High-performance local storage on newer VM SKUs

> [!WARNING]  
> Temporary and resource disks aren't persistent. Data stored on these disks will be lost when the VM is deallocated, redeployed, or stopped for maintenance.

## Prerequisites

Before formatting temporary or resource disks:

1. [Identify the correct disk](./add-disk.md#identifying-disks) to avoid data loss
2. Understand that data isn't persistent across VM stops/deallocations
3. Have SSH access to your VM with root or sudo privileges

## Format temporary and resource disks

> [!WARNING]
> Formatting will permanently erase all data on the disk. Ensure you're working with the correct disk and that no important data exists on it.

> [!NOTE]
> We recommend that you use the latest version of `parted` that's available for your distribution. If the disk size is 2 tebibytes (TiB) or larger, you must use GPT partitioning. If the disk size is under 2 TiB, then you can use either MBR or GPT partitioning.

### [SCSI Resource Disk](#tab/scsi)

The following example uses `parted` on `/dev/sdb`, which is typically where the resource disk appears. Replace `sdb` with the correct device for your disk. We're using the [XFS](https://xfs.wiki.kernel.org/) file system for better performance.

```bash
sudo parted /dev/disk/azure/resource --script mklabel gpt mkpart xfspart xfs 0% 100%  
sudo partprobe /dev/sdb
sudo mkfs.xfs /dev/sdb1
```

### [NVMe](#tab/nvme)

The following examples assume you have identified your disk as shown in the [identifying disks](./add-disk.md#identifying-disks) section. If you have azure-vm-utils installed, you can use it to identify local disks.

```bash
# Example: Format local NVMe disk (replace nvme1n1 with your identified disk)
sudo parted /dev/disk/azure/local/by-index/0 --script mklabel gpt mkpart xfspart xfs 0% 100%
sudo partprobe /dev/nvme1n1
sudo mkfs.xfs /dev/nvme1n1p1
```

### [RAID](#tab/raid)

Some VM SKUs provide multiple local NVMe disks. You can set up RAID for better performance or redundancy:

```bash
# Example: Create RAID 0 with two local NVMe disks for performance
sudo mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/nvme1n1 /dev/nvme2n1
sudo mkfs.xfs /dev/md0
```

---

Use the [partprobe](https://linux.die.net/man/8/partprobe) utility to make sure the kernel is aware of the new partition and file system. Failure to use `partprobe` can cause the blkid or lsblk commands to not return the UUID for the new file system immediately.


## Mount temporary and resource disks

Now, create a directory to mount the file system by using `mkdir`. For temporary storage, common mount points include `/mnt`, `/tmp`, or application-specific directories.

```bash
sudo mkdir /mnt/temp
```

### [SCSI](#tab/scsi-mount)

Use `mount` to mount the file system. The following example mounts the `/dev/sdb1` partition to the `/mnt/temp` mount point:

```bash
sudo mount /dev/sdb1 /mnt/temp
```

You can also use the Azure device path:

```bash
sudo mount /dev/disk/azure/resource-part1 /mnt/temp
```

### [NVMe](#tab/nvme-mount)

The following examples assume you have identified your disk as shown in the [identifying disks](./add-disk.md#identifying-disks) section. If you have azure-vm-utils installed, you can use it to identify local disks.

```bash
# Using direct device path (replace nvme1n1p1 with your identified disk's partition)
sudo mount /dev/nvme1n1p1 /mnt/temp

# Or using Azure device path
sudo mount /dev/disk/azure/local/by-index/0-part1 /mnt/temp  
```

### [RAID](#tab/raid-mount)

For RAID arrays created as shown in the formatting section:

```bash
sudo mkdir /mnt/raid
sudo mount /dev/md0 /mnt/raid
```
---



## TRIM/UNMAP support for temporary disks

Local temporary disks support TRIM/UNMAP operations. For optimal performance:

### Use the `discard` mount option in `/etc/fstab`:

```
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /mnt/temp   xfs   defaults,discard,nobootwait   0   0
```

### Alternatively, run `fstrim` periodically:

### [Ubuntu](#tab/ubuntu)

```bash
sudo apt install util-linux
sudo fstrim /mnt/temp
```

### [RHEL](#tab/rhel)

```bash
sudo yum install util-linux
sudo fstrim /mnt/temp
```

### [SLES](#tab/suse)

```bash
sudo zypper in util-linux
sudo fstrim /mnt/temp
```
---

## Important reminders

> [!WARNING]
> - Temporary and resource disk data is lost during VM deallocations, stops for maintenance, or redeployments
> - Always use persistent Azure disks for important data
> - Test your application's behavior with temporary disk data loss scenarios

> [!TIP]  
> - Use temporary disks for swap, cache, temporary files, and application working directories
> - NVMe local disks offer the highest performance for temporary storage
> - Monitor disk usage to avoid running out of temporary storage space
> - Consider automated backup strategies if temporary disk content has any value

## Troubleshooting

[!INCLUDE [virtual-machines-linux-lunzero](../includes/virtual-machines-linux-lunzero.md)]
