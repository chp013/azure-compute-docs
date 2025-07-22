---
title: Managing remote data disks on Linux VMs using Azure CLI
description: Learn to format, mount, and persist remote data disks on Linux VMs with both SCSI and NVMe interfaces
author: vamckms
ms.service: azure-disk-storage
ms.custom: devx-track-azurecli, linux-related-content
ms.collection: linux
ms.topic: how-to
ms.date: 22/07/2025
ms.author: vakavuru
---

# Managing remote data disks

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

This article covers how to format, mount, and persist remote data disks on Azure Linux VMs. Remote data disks are persistent storage attached to your VM that can use either SCSI or NVMe interfaces depending on your VM SKU.

## Prerequisites

Before formatting and mounting a data disk, ensure you have:

1. [Identified the correct disk](./identifying-disks-linux.md) to avoid data loss
2. SSH access to your VM
3. Root or sudo privileges

> [!WARNING]
> Always verify you're working with the correct disk before formatting. Formatting the wrong disk will result in data loss.

## Identify the disk to format

First, identify the disk you want to format. The method varies depending on whether your VM uses SCSI or NVMe interface.

### Using traditional SCSI interface
```bash
# List SCSI disks
lsblk -o NAME,HCTL,SIZE,MOUNTPOINT | grep sd

# Use azure-vm-utils symlinks if available
ls -la /dev/disk/azure/scsi1/
```

### Using NVMe interface
```bash
# List NVMe disks
lsblk -o NAME,SIZE,MOUNTPOINT | grep nvme

# Use azure-vm-utils symlinks if available
ls -la /dev/disk/azure/data/by-lun/
```

### Using azure-vm-utils (recommended)
If [azure-vm-utils](./az-vm-utils.md) is installed, use predictable symlinks:

```bash
# List all Azure disks
ls -la /dev/disk/azure/

# For data disks by LUN (works for both SCSI and NVMe)
ls -la /dev/disk/azure/data/by-lun/

# Example: Access data disk at LUN 0
ls -la /dev/disk/azure/data/by-lun/0
```

# Format the disk

Format the disk with `parted`. If the disk size is two tebibytes (TiB) or larger then you must use GPT partitioning, if it is under 2TiB, then you can use either MBR or GPT partitioning.

> [!NOTE]
> It is recommended that you use the latest version `parted` that is available for your distro. If the disk size is 2 tebibytes (TiB) or larger, you must use GPT partitioning. If disk size is under 2 TiB, then you can use either MBR or GPT partitioning.

## Traditional device naming approach

The following example uses `parted` on `/dev/sdc`, which is where the first data disk will typically be on most VMs. Replace `sdc` with the correct option for your disk. We're also formatting it using the [XFS](https://xfs.wiki.kernel.org/) filesystem.

```bash
sudo parted /dev/sdc --script mklabel gpt mkpart xfspart xfs 0% 100%
sudo partprobe /dev/sdc
sudo mkfs.xfs /dev/sdc1
```

## Using azure-vm-utils symlinks (recommended)

For more reliable disk identification, use azure-vm-utils symlinks when available:

```bash
# Example: Format data disk at LUN 0
sudo parted /dev/disk/azure/data/by-lun/0 --script mklabel gpt mkpart xfspart xfs 0% 100%
sudo partprobe /dev/disk/azure/data/by-lun/0
sudo mkfs.xfs /dev/disk/azure/data/by-lun/0p1
```

## NVMe disk formatting

For NVMe disks, the process is similar but device names differ:

```bash
# Example: Format NVMe disk (replace nvme1n1 with your disk)
sudo parted /dev/nvme1n1 --script mklabel gpt mkpart xfspart xfs 0% 100%
sudo partprobe /dev/nvme1n1
sudo mkfs.xfs /dev/nvme1n1p1
```

Use the [partprobe](https://linux.die.net/man/8/partprobe) utility to make sure the kernel is aware of the new partition and filesystem. Failure to use `partprobe` can cause the blkid or lsblk commands to not return the UUID for the new filesystem immediately.

# Mount the disk

Now, create a directory to mount the file system using `mkdir`. The following example creates a directory at `/datadrive`:

```bash
sudo mkdir /datadrive
```

## Mounting SCSI disks

Use `mount` to then mount the filesystem. The following example mounts the `/dev/sdc1` partition to the `/datadrive` mount point:

```bash
sudo mount /dev/sdc1 /datadrive
```

## Mounting NVMe disks

For NVMe disks, use the appropriate NVMe device name:

```bash
# Example: Mount NVMe partition
sudo mount /dev/nvme1n1p1 /datadrive
```

## Mounting using azure-vm-utils symlinks

For more reliable mounting, use azure-vm-utils symlinks when available:

```bash
# Example: Mount using LUN-based symlink
sudo mount /dev/disk/azure/data/by-lun/0p1 /datadrive

# Or resolve the symlink and mount
DEVICE=$(readlink -f /dev/disk/azure/data/by-lun/0)
sudo mount ${DEVICE}1 /datadrive
```

# Persist the mount

To ensure that the drive is remounted automatically after a reboot, it must be added to the `/etc/fstab` file. It's also highly recommended that the UUID (Universally Unique Identifier) is used in `/etc/fstab` to refer to the drive rather than just the device name (such as, /dev/sdc1). If the OS detects a disk error during boot, using the UUID avoids the incorrect disk being mounted to a given location. Remaining data disks would then be assigned those same device IDs. To find the UUID of the new drive, use the `blkid` utility:

```bash
sudo blkid
```

The output looks similar to the following example:

```
/dev/sda1: LABEL="cloudimg-rootfs" UUID="11111111-1b1b-1c1c-1d1d-1e1e1e1e1e1e" TYPE="ext4" PARTUUID="1a1b1c1d-11aa-1234-1a1a1a1a1a1a"
/dev/sda15: LABEL="UEFI" UUID="BCD7-96A6" TYPE="vfat" PARTUUID="1e1g1cg1h-11aa-1234-1u1u1a1a1u1u"
/dev/sdb1: UUID="22222222-2b2b-2c2c-2d2d-2e2e2e2e2e2e" TYPE="ext4" TYPE="ext4" PARTUUID="1a2b3c4d-01"
/dev/sda14: PARTUUID="2e2g2cg2h-11aa-1234-1u1u1a1a1u1u"
/dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="xfs" PARTLABEL="xfspart" PARTUUID="c1c2c3c4-1234-cdef-asdf3456ghjk"
```

> [!WARNING]
> Improperly editing the `/etc/fstab` file could result in an unbootable system. If unsure, refer to the distribution's documentation for information on how to properly edit this file. It is also recommended that a backup of the `/etc/fstab` file is created before editing.

Next, open the `/etc/fstab` file in a text editor. Add a line to the end of the file, using the UUID value for the `/dev/sdc1` device that was created in the previous steps, and the mountpoint of `/datadrive`. Using the example from this article, the new line would look like the following:

```
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   xfs   defaults,nofail   1   2
```

When you're done editing the file, save and close the editor.

Alternatively, you can run the following command to add the disk to the `/etc/fstab` file:

```bash
# Get UUID of the new filesystem
UUID=$(sudo blkid -s UUID -o value /dev/sdc1)
echo "UUID=$UUID   /datadrive   xfs   defaults,nofail   1   2" | sudo tee -a /etc/fstab
```

## NVMe disk persistence considerations

For NVMe disks, the fstab entry process is similar, but device names differ:

```bash
# Example for NVMe disk
UUID=$(sudo blkid -s UUID -o value /dev/nvme1n1p1)
echo "UUID=$UUID   /datadrive   xfs   defaults,nofail   1   2" | sudo tee -a /etc/fstab
```

## Using azure-vm-utils for persistent mounting

When using azure-vm-utils symlinks, you can reference them in fstab, but UUIDs are still recommended:

```bash
# Get UUID using symlink
UUID=$(sudo blkid -s UUID -o value /dev/disk/azure/data/by-lun/0p1)
echo "UUID=$UUID   /datadrive   xfs   defaults,nofail   1   2" | sudo tee -a /etc/fstab
```

> [!NOTE]
> Later removing a data disk without editing fstab could cause the VM to fail to boot. Most distributions provide either the `nofail` and/or `nobootwait` fstab options. These options allow a system to boot even if the disk fails to mount at boot time. Consult your distribution's documentation for more information on these parameters.

The `nofail` option ensures that the VM starts even if the filesystem is corrupt or the disk does not exist at boot time. Without this option, you may encounter behavior as described in [Cannot SSH to Linux VM due to FSTAB errors](/troubleshoot/azure/virtual-machines/linux-virtual-machine-cannot-start-fstab-errors)

The Azure VM Serial Console can be used for console access to your VM if modifying fstab has resulted in a boot failure. More details are available in the [Serial Console documentation](/troubleshoot/azure/virtual-machines/serial-console-linux).

# TRIM/UNMAP support for remote data disks

Remote data disks in Azure support TRIM/UNMAP operations to discard unused blocks. This feature is important for cost optimization on disks that are billed based on consumed storage.

## NVMe TRIM support

NVMe disks on Azure have enhanced TRIM/UNMAP support. For optimal performance:

- Use the `discard` mount option in `/etc/fstab`
- NVMe devices generally handle TRIM operations more efficiently than SCSI devices
- Consider periodic `fstrim` operations for workloads with high disk churn

## TRIM configuration options

There are two ways to enable TRIM support in your Linux VM. As usual, consult your distribution for the recommended approach:

### Option 1: Mount-time discard
Use the `discard` mount option in `/etc/fstab`, for example:

```
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   xfs   defaults,discard,nofail   1   2
```

### Option 2: Periodic trim
In some cases, the `discard` option may have performance implications. Alternatively, you can run the `fstrim` command manually from the command line, or add it to your crontab to run regularly:

```bash
echo "UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   xfs   defaults,nofail   1   2" >> /etc/fstab
```

> [!NOTE]
> Later removing a data disk without editing fstab could cause the VM to fail to boot. Most distributions provide either the nofail and/or nobootwait fstab options. These options allow a system to boot even if the disk fails to mount at boot time. Consult your distribution's documentation for more information on these parameters.

The nofail option ensures that the VM starts even if the filesystem is corrupt or the disk does not exist at boot time. Without this option, you may encounter behavior as described in [Cannot SSH to Linux VM due to FSTAB errors](/troubleshoot/azure/virtual-machines/linux-virtual-machine-cannot-start-fstab-errors)

The Azure VM Serial Console can be used for console access to your VM if modifying fstab has resulted in a boot failure. More details are available in the [Serial Console documentation](/troubleshoot/azure/virtual-machines/serial-console-linux).

# TRIM/UNMAP support for Linux in Azure

Some Linux kernels support TRIM/UNMAP operations to discard unused blocks on the disk. This feature is primarily useful to inform Azure that deleted pages are no longer valid and can be discarded. This feature can save money on disks that are billed based on the amount of consumed storage, such as unmanaged standard disks and disk snapshots.

There are two ways to enable TRIM support in your Linux VM. As usual, consult your distribution for the recommended approach:

• Use the `discard` mount option in `/etc/fstab`, for example:

```
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   xfs   defaults,discard   1   2
```

• In some cases, the `discard` option may have performance implications. Alternatively, you can run the `fstrim` command manually from the command line, or add it to your crontab to run regularly:

## Ubuntu

```bash
sudo apt install util-linux
sudo fstrim /datadrive
```

## RHEL

```bash
sudo yum install util-linux
sudo fstrim /datadrive
```

## SLES

```bash
sudo zypper in util-linux
sudo fstrim /datadrive
```

# Troubleshooting

[!INCLUDE [virtual-machines-linux-lunzero](~/articles/includes/virtual-machines-linux-lunzero.md)]
