---
title: Introduction to Azure-VM-Utils
description: Learn about azure-vm-utils package that provides utilities and udev rules for optimal Linux experience on Azure VMs
author: vamckms
ms.service: azure-virtual-machines
ms.custom: devx-track-azurecli, linux-related-content
ms.collection: linux
ms.topic: concept-article
ms.date: 07/22/2025
ms.author: vakavuru
---

# Learn about Azure-VM-Utils

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

The [azure-vm-utils](https://github.com/Azure/azure-vm-utils) package provides essential utilities and udev rules to optimize the Linux experience on Azure Virtual Machines. This package consolidates device management tools for SCSI, NVMe, MANA, and Mellanox devices, making disk identification and management more reliable and consistent across different VM configurations.

## NVMe Udev Rules

Newer VM SKUs on Azure have adopted NVMe interface for disk management. VMs with NVMe interface interpret and present disks slightly differently than with the SCSI interface. See [SCSI to NVMe conversion](/azure/virtual-machines/nvme-linux#scsi-vs-nvme) for details. 

NVMe udev rules in this package consolidate critical tools and udev rules to create stable, predictable symlinks for disks in Azure. It provides an easy and reliable way to identify disks, making automation, troubleshooting, and management significantly simpler.

### Symlinks

WALinuxAgent currently includes udev rules to provide several symlinks for SCSI disks:

- `/dev/disk/azure/resource`
- `/dev/disk/azure/root`
- `/dev/disk/azure/scsi0/lun<lun>`
- `/dev/disk/azure/scsi1/lun<lun>`

The rules found in WALinuxAgent are being extended with azure-vm-utils to add identification support for NVMe devices.

New symlinks that will be provided for all instances with NVMe disks include:

- `/dev/disk/azure/data/by-lun/<lun>`
- `/dev/disk/azure/local/by-serial/<serial>`
- `/dev/disk/azure/os`

For v6 and newer VM sizes with local NVMe disks supporting namespace identifiers, additional links will be available:

- `/dev/disk/azure/local/by-index/<index>`
- `/dev/disk/azure/local/by-name/<name>`

For future VM sizes with remote NVMe disks supporting namespace identifiers, additional links will be available:

- `/dev/disk/azure/data/by-name/<name>`

### SCSI Compatibility

To ensure backward compatibility for disks using SCSI controllers, azure-vm-utils supports the following links:

- `/dev/disk/azure/os`
- `/dev/disk/azure/resource`

> [!NOTE]
> Some VM sizes come with both NVMe local disks in addition to a SCSI temp resource disk. These temp resource disks are separate from local disks, which are dedicated to local NVMe disks to avoid confusion.

### Linux Distro Support

We are working with all endorsed distro partners to include az-vm-utils in their default images. The following distros and versions currently include it:

| Distro | Version |
|--------|---------|
| Fedora | 41 |
| Flatcar | 4152.2.3 |
| Azure Linux | 2.0 |

### Installation

If the package isn't present in the default platform image, install it via package managers or from the [GitHub repository](https://github.com/Azure/azure-vm-utils).

### Manual Installation

For distributions where azure-vm-utils isn't pre-installed, build and install it manually:

```bash
# Clone the repository
git clone https://github.com/Azure/azure-vm-utils.git
cd azure-vm-utils

# Build the package
cmake .
make

# Install (requires root privileges)
sudo make install
```

## Utilities

### azure-nvme-id

The `azure-nvme-id` utility helps identify Azure NVMe devices and their properties. This is particularly useful for troubleshooting and scripting.

To run the utility:

```bash
sudo azure-nvme-id
```

To run in udev mode (typically used by udev rules):

```bash
DEVNAME=/dev/nvme0n1 azure-nvme-id --udev
```

## Using the Symlinks

Once azure-vm-utils is installed, you can use the predictable symlinks for disk operations instead of relying on device names that might change between reboots.

### Examples

List all Azure disk symlinks:

```bash
ls -la /dev/disk/azure/
```

Access the OS disk:

```bash
ls -la /dev/disk/azure/os
```

Access data disks by LUN:

```bash
ls -la /dev/disk/azure/data/by-lun/
```

Access local NVMe disks by serial number:

```bash
ls -la /dev/disk/azure/local/by-serial/
```

## Verification

To verify that azure-vm-utils is working correctly on your VM:

1. Check if the package is installed:
   
   ```bash
   # For RPM-based systems
   rpm -qa | grep azure-vm-utils
   
   # For DEB-based systems
   dpkg -l | grep azure-vm-utils
   ```

1. Verify udev rules are in place:
   
   ```bash
   ls -la /usr/lib/udev/rules.d/*azure*
   ```

1. Check for Azure disk symlinks:
   
   ```bash
   ls -la /dev/disk/azure/
   ```