---
title: Managing temporary and resource disks on Linux VMs
description: Learn to format and mount temporary and resource disks on Azure Linux VMs with both SCSI and NVMe interfaces
author: vamckms
ms.service: azure-disk-storage
ms.custom: devx-track-azurecli, linux-related-content
ms.collection: linux
ms.topic: how-to
ms.date: 22/07/2025
ms.author: vakavuru
---

# Managing temporary and resource disks

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

This article covers how to format and mount temporary (local) and resource disks on Azure Linux VMs. These disks provide local storage that is not persistent and may use either SCSI or NVMe interfaces depending on your VM SKU.

## Understanding temporary and resource disks

- **Temporary/Local disks**: Fast local storage for temporary data, not persistent across VM deallocations
- **Resource disks**: Traditional temporary storage, usually SCSI-based on older VMs
- **NVMe local disks**: High-performance local storage on newer VM SKUs

> [!WARNING]  
> Temporary and resource disks are not persistent. Data stored on these disks will be lost when the VM is deallocated, redeployed, or stopped for maintenance.

## Prerequisites

Before formatting temporary or resource disks:

1. [Identify the correct disk](./add-disk.md#identifying-disks) to avoid data loss
2. Understand that data is not persistent across VM stops/deallocations
3. Have SSH access to your VM with root or sudo privileges

# Format temporary and resource disks

> [!WARNING]
> Formatting will permanently erase all data on the disk. Ensure you're working with the correct disk and that no important data exists on it.

## Format SCSI resource disks

> [!NOTE]
> It is recommended that you use the latest version `parted` that is available for your distro. If the disk size is 2 tebibytes (TiB) or larger, you must use GPT partitioning. If disk size is under 2 TiB, then you can use either MBR or GPT partitioning.

The following example uses `parted` on `/dev/sdb`, which is typically where the resource disk appears. Replace `sdb` with the correct device for your disk. We're using the [XFS](https://xfs.wiki.kernel.org/) filesystem for better performance.

```bash
sudo parted /dev/sdb --script mklabel gpt mkpart xfspart xfs 0% 100%
sudo partprobe /dev/sdb
sudo mkfs.xfs /dev/sdb1
```

## Format NVMe local disks

### Traditional Approach
```bash
# Example: Format local NVMe disk (replace nvme1n1 with your disk)
sudo parted /dev/nvme1n1 --script mklabel gpt mkpart xfspart xfs 0% 100%
sudo partprobe /dev/nvme1n1
sudo mkfs.xfs /dev/nvme1n1p1
```

### Using azure-vm-utils
**Content to be added**

## Multiple local disk scenarios (RAID)

Some VM SKUs provide multiple local NVMe disks. You can set up RAID for better performance or redundancy:

```bash
# Example: Create RAID 0 with two local NVMe disks for performance
sudo mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/nvme1n1 /dev/nvme2n1
sudo mkfs.xfs /dev/md0
```

Use the [partprobe](https://linux.die.net/man/8/partprobe) utility to make sure the kernel is aware of the new partition and filesystem. Failure to use `partprobe` can cause the blkid or lsblk commands to not return the UUID for the new filesystem immediately.

# Mount temporary and resource disks

Now, create a directory to mount the file system using `mkdir`. For temporary storage, common mount points include `/mnt`, `/tmp`, or application-specific directories.

```bash
sudo mkdir /mnt/temp
```

## Mount SCSI resource disks

Use `mount` to mount the filesystem. The following example mounts the `/dev/sdb1` partition to the `/mnt/temp` mount point:

```bash
sudo mount /dev/sdb1 /mnt/temp
```

## Mount NVMe local disks

For NVMe local disks:

```bash
# Example: Mount NVMe partition
sudo mount /dev/nvme1n1p1 /mnt/temp
```

## Mount using azure-vm-utils symlinks

**Content to be added**


## TRIM/UNMAP support for temporary disks

Local temporary disks support TRIM/UNMAP operations. For optimal performance:

### Use the `discard` mount option in `/etc/fstab`:

```
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /mnt/temp   xfs   defaults,discard,nobootwait   0   0
```

### Alternatively, run `fstrim` periodically:

## Ubuntu

```bash
sudo apt install util-linux
sudo fstrim /mnt/temp
```

## RHEL/CentOS/Fedora

```bash
sudo yum install util-linux
sudo fstrim /mnt/temp
```

## SLES

```bash
sudo zypper in util-linux
sudo fstrim /mnt/temp
```

# Important reminders

> [!WARNING]
> - Temporary and resource disk data is lost during VM deallocations, stops for maintenance, or redeployments
> - Always use persistent Azure disks for important data
> - Test your application's behavior with temporary disk data loss scenarios

> [!TIP]  
> - Use temporary disks for swap, cache, temporary files, and application working directories
> - NVMe local disks offer the highest performance for temporary storage
> - Monitor disk usage to avoid running out of temporary storage space
> - Consider automated backup strategies if temporary disk content has any value

# Troubleshooting

[!INCLUDE [virtual-machines-linux-lunzero](~/articles/includes/virtual-machines-linux-lunzero.md)]
