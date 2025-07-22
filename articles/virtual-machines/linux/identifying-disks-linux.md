---
title: Identifying disks on Linux virtual machines in Azure
description: Learn how to identify OS, data, and temporary disks on Azure Linux VMs, including SCSI and NVMe interfaces
author: vamckms
ms.service: azure-disk-storage
ms.custom: devx-track-azurecli, linux-related-content
ms.collection: linux
ms.topic: how-to
ms.date: 22/07/2025
ms.author: vakavuru
---
# Identifying disks

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

Azure Linux virtual machines use different disk interfaces depending on the VM SKU and generation. This article covers how to identify OS, data, and temporary disks on both SCSI and NVMe-based VMs. Understanding disk identification is crucial for proper disk management, especially when working with multiple disks or automating disk operations.

## VM generations and disk interfaces

- **Older VM SKUs**: Use SCSI interface for disk management
- **Newer VM SKUs (v5, v6, and newer)**: May use NVMe interface for improved performance
- **Mixed configurations**: Some VMs combine NVMe and SCSI disks

For more information about SCSI vs NVMe differences, see [SCSI to NVMe conversion](/azure/virtual-machines/nvme-linux#scsi-vs-nvme). 

## Connect to the virtual machine

To identify disks associated with your Linux virtual machine (VM), SSH into the VM. For more information, see [How to use SSH with Linux on Azure](/azure/virtual-machines/linux/mac-create-ssh-keys). The following example connects to a VM with the public IP address of 10.123.123.25 with the username azureuser:

```bash
ssh azureuser@10.123.123.25
```

## Identify disk interface type

Before identifying specific disks, determine whether your VM uses SCSI, NVMe, or a combination of both interfaces.

### Check for NVMe devices

```bash
lsblk -o NAME,TYPE,SIZE,MOUNTPOINT | grep nvme
```

If NVMe devices are present, you'll see output similar to:
```
nvme0n1     disk    30G
├─nvme0n1p1 part   29.9G /
├─nvme0n1p14 part   4M
└─nvme0n1p15 part  106M  /boot/efi
nvme1n1     disk   50G
```

### Check for SCSI devices

```bash
lsblk -o NAME,TYPE,SIZE,MOUNTPOINT | grep sd
```

For SCSI devices, you'll see output similar to:
```
sda     disk    30G
├─sda1  part   29.9G /
├─sda14 part    4M
└─sda15 part   106M  /boot/efi
sdb     disk   50G
```

## Traditional SCSI disk identification

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

Here, `sdc` is the disk that we want, because it's 50G. If you add multiple disks, and aren't sure which disk it's based on size alone, you can go to the VM page in the portal, select **Disks**, and check the LUN number for the disk under **Data disks**. Compare the LUN number from the portal to the last number of the HCTL portion of the output, which is the LUN. 

Another option is to list the contents of the `/dev/disk/azure/scsi1` directory:

```bash
ls -l /dev/disk/azure/scsi1
```

The output should be similar to the following example:

```
lrwxrwxrwx 1 root root 12 Mar 28 19:41 lun0 -> ../../../sdc
```

## Using azure-vm-utils for disk identification

If your VM has the [azure-vm-utils](./az-vm-utils.md) package installed, you can use predictable symlinks to identify disks more reliably.

### Check if azure-vm-utils is installed

```bash
# Check for azure-vm-utils package
rpm -qa | grep azure-vm-utils    # For RPM-based systems
dpkg -l | grep azure-vm-utils    # For DEB-based systems

# Check for Azure disk symlinks
ls -la /dev/disk/azure/
```

### Using Azure disk symlinks

If azure-vm-utils is installed, you can use predictable symlinks:

#### For SCSI disks:
```bash
# List all Azure disk symlinks
ls -la /dev/disk/azure/

# Check OS disk
ls -la /dev/disk/azure/os

# Check resource/temp disk
ls -la /dev/disk/azure/resource

# Check data disks by SCSI controller and LUN
ls -la /dev/disk/azure/scsi1/
```

#### For NVMe disks:
```bash
# Check OS disk
ls -la /dev/disk/azure/os

# Check data disks by LUN
ls -la /dev/disk/azure/data/by-lun/

# Check local NVMe disks by serial number
ls -la /dev/disk/azure/local/by-serial/

# For newer VM SKUs with namespace support
ls -la /dev/disk/azure/local/by-index/
ls -la /dev/disk/azure/local/by-name/
```

## NVMe disk identification

For VMs with NVMe interfaces, use these methods to identify disks:

### List NVMe devices
```bash
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT | grep nvme
```

### Use azure-nvme-id utility
If azure-vm-utils is installed, you can use the `azure-nvme-id` utility for detailed NVMe device information:

```bash
# Identify Azure NVMe devices
sudo azure-nvme-id

# For specific device information
DEVNAME=/dev/nvme0n1 azure-nvme-id --udev
```

### NVMe device naming convention
- `nvme0n1`, `nvme1n1`, etc.: NVMe devices
- `nvme0n1p1`, `nvme1n1p1`, etc.: Partitions on NVMe devices

## Practical examples

### Identify all disks and their purposes
```bash
# List all block devices with mount points
lsblk -f

# Show disk usage
df -h

# Get detailed disk information
sudo fdisk -l
```

### Cross-reference with Azure portal
1. Go to your VM in the Azure portal
2. Select **Disks** from the left menu
3. Note the LUN numbers for data disks
4. Compare with the HCTL output or azure-vm-utils symlinks

### Troubleshooting disk identification
If you're having trouble identifying disks:

1. **Check VM generation and type** in the Azure portal
2. **Verify azure-vm-utils installation** if using newer VM SKUs
3. **Use multiple identification methods** (lsblk, azure-vm-utils symlinks, portal)
4. **Check for both SCSI and NVMe devices** on mixed configurations
