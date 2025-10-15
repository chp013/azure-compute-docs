---
title: Azure N-series AMD GPU Driver Setup for Linux
description: How to set up AMD Radeon PRO V710 NVv5 Linux Installation Guide.
services: virtual-machines
author: nmagatala-MSFT
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.collection: linux
ms.topic: how-to
ms.custom: linux-related-content
ms.date: 10/15/2025
ms.author: padmalathas
# Customer intent: As a cloud engineer managing Linux VMs with AMD GPUs, I want to install the necessary AMD GPU drivers and configure my NVv5-V710 instances, so that I can optimize performance for AI and graphics workloads.
---

# Install AMD GPU drivers on NVads V710-series Linux VMs

**Applies to:** :heavy_check_mark: Linux VMs

> [!IMPORTANT]
> To align with inclusive language practices, we have replaced the term "blacklist" with "blocklist" throughout this documentation. This change reflects our commitment to avoiding terminology that may carry unintended negative connotations or perceived racial bias.
> However, in code snippets and technical references where "blacklist" is part of established syntax or tooling (for example, configuration files, command-line parameters), the original term is retained to preserve functional accuracy. This usage is strictly technical and does not imply any discriminatory intent.

## NVads V710-series

To leverage the GPU capabilities of Azure’s new NVads V710-series virtual machines running Linux, you’ll need to install AMD GPU drivers. The AMD GPU driver extension streamlines this process by automating driver installation for NVv710-series VMs. You can manage the extension via the Azure portal, Azure PowerShell, or Azure Resource Manager (ARM) templates. For details on supported operating systems and deployment steps, see [AMD GPU Driver Extension](../extensions/hpccompute-amd-gpu-linux.md) documentation.

The [marketplace image](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/amdinc1746636494855.nvv5_v710_linux_rocm_image?tab=Overview) comes preloaded with the AMD GPU driver, helping accelerate VM setup. This guide explains how to install AMD GPU drivers on Azure NVads V710-series Linux virtual machines (VMs). It covers both automated and manual installation methods specifically for **Ubuntu**. 

## ROCm

> [!NOTE]
> Currently, Azure provides installation instructions for:
> - Ubuntu 22.04
> - Ubuntu 24.04
>   
> For other Linux distributions, see:
> - [Quick start installation guide - ROCm installation(Linux)](https://rocm.docs.amd.com/projects/install-on-linux/en/docs-6.3.3/install/quick-start.html)
> - [ROCm release history - ROCm Documentation](https://rocm.docs.amd.com/en/latest/release/versions.html#rocm-release-history)

Install the AMD Linux Driver to leverage the full capabilities of the AMD Radeon PRO V710 GPU on an NVv5-V710 GPU Linux instance in Microsoft Azure. The sections that follow provide detailed instructions for installing the Linux driver and running inference workloads using ROCm on this instance type.

## Quick Start Options

### Option 1: Use the AMD GPU Driver Extension

The simplest method is using the AMD GPU Driver Extension, which automates driver installation for NVv710-series VMs. You can deploy this extension through:

- Azure Portal
- Azure PowerShell
- Azure Resource Manager templates

### Option 2: Use Pre-configured Marketplace Image

A marketplace image is available with pre-installed AMD GPU drivers, allowing for faster VM deployment.

### Option 3: Manual Installation

Follow the instructions below for manual driver installation and configuration.

---

## ROCm Driver Installation

### Prerequisites

**System Requirements:**

- Disk size must exceed 64GB for optimal performance
- Supported distributions: Ubuntu 22.04 or Ubuntu 24.04
- Virtual Function Device ID: 7461 (AMD Radeon PRO V710 GPU)

### Step 1: Verify Your System

Check your Linux distribution:

```bash
cat /etc/*release
```

Check your kernel version:

```bash
uname -srmv
```

Verify your GPU card is detected:

```bash
sudo lspci -d 1002:7461
```

You should see output similar to:

```
c3:00.0 Display controller: Advanced Micro Devices, Inc. [AMD/ATI] Device 7461
```

### Step 2: Install the Driver

#### For Ubuntu 22.04

```bash
sudo apt update
sudo apt install "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
sudo apt install python3-setuptools python3-wheel
sudo usermod -a -G render,video $LOGNAME
wget https://repo.radeon.com/amdgpu-install/6.4.3/ubuntu/jammy/amdgpu-install_6.4.60403-1_all.deb
sudo apt install ./amdgpu-install_6.4.60403-1_all.deb
sudo apt update
sudo apt install amdgpu-dkms rocm
```

#### For Ubuntu 24.04





```bash
sudo apt update
sudo apt install "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
sudo apt install python3-setuptools python3-wheel
sudo usermod -a -G render,video $LOGNAME # Add the current user to the render and video groups
wget https://repo.radeon.com/amdgpu-install/7.0.1/ubuntu/noble/amdgpu-install_7.0.1.70001-1_all.deb
sudo apt install ./amdgpu-install_7.0.1.70001-1_all.deb
sudo apt update
sudo apt install amdgpu-dkms rocm
```

### Step 3: Load and Verify the Driver

Load the driver:

```bash
sudo modprobe amdgpu
```

Check that the driver loaded successfully:

```bash
sudo dmesg | grep amdgpu
```

Verify driver status with AMD-SMI:

```bash
amd-smi monitor
```

### Step 4: Enable Automatic Loading on Reboot

Search for blocklist entries:

```bash
grep amdgpu /etc/modprobe.d/* -rn
```

If the driver is blocklisted, remove the blocklist:

```bash
sudo nano /etc/modprobe.d/blacklist.conf
```

Delete the line containing `blacklist amdgpu`, then update initramfs:

```bash
sudo update-initramfs -uk all
```

Reboot to apply changes:

```bash
sudo reboot
```

---

## Graphics and ROCm Installation

This section covers installing the AMD driver for graphics workloads with ROCm libraries and development tools.

### Prerequisites

**System Requirements:**

- Ubuntu 22.04 with kernel 6.5
- Disk size greater than 64GB
- Desktop environment (for graphics workloads, use Ubuntu Desktop ISO)

### Pre-Installation Steps

Update package list:

```bash
sudo apt update
```

Install Python packages:

```bash
sudo apt install python3-setuptools python3-wheel
```

Add user to required groups:

```bash
sudo usermod -a -G render,video $LOGNAME
```

Install kernel headers:

```bash
sudo apt install "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
```

### Blocklist Default Driver

Check if the driver is already blocklisted:

```bash
grep amdgpu /etc/modprobe.d/* -rn
```

If not blocklisted, add it:

```bash
sudo vim /etc/modprobe.d/blacklist.conf
```

Add this line:

```
blacklist amdgpu
```

Apply changes:

```bash
sudo update-initramfs -uk all
```

Reboot the system:

```bash
sudo reboot
```

### Install AMD Driver with Graphics Support

Upgrade the system:

```bash
sudo apt upgrade
```

Download the installer:

```bash
wget -N -P /tmp/ https://repo.radeon.com/amdgpu-install/6.1.4/ubuntu/jammy/amdgpu-install_6.1.60104-1_all.deb
```

If a previous driver exists, remove it:

```bash
sudo amdgpu-uninstall
sudo apt remove amdgpu-install --purge
```

Install the new driver:

```bash
sudo apt-get install /tmp/amdgpu-install_6.1.60104-1_all.deb
sudo amdgpu-install --usecase=workstation,rocm,amf --opencl=rocr --vulkan=pro --no-32 --accept-eula
```

Load the driver:

```bash
sudo modprobe amdgpu
```

Verify installation:

```bash
sudo dmesg | grep amdgpu
```

Remove blocklist and enable automatic loading (follow steps from previous section).

---

## X11 Remote Server Configuration

After installing the graphics driver, you'll need to configure a virtual display with hardware acceleration for remote access.

### Install Required Packages

```bash
sudo apt install net-tools
sudo apt install x11vnc
```

### Configure GDM3

Edit the GDM3 configuration:

```bash
sudo vim /etc/gdm3/custom.conf
```

Modify to include:

```ini
[daemon]
AutomaticLoginEnable=true
AutomaticLogin=your_username
WaylandEnable=false
```

Restart GDM3:

```bash
sudo systemctl restart gdm3
```

### Configure X11

Get your GPU's Bus ID:

```bash
lspci -d 1002: | awk '{print $1}'
```

Convert the hex Bus ID to decimal format. For example, `3a9e:00:00.0` becomes `3841536`.

Edit the X configuration file `/usr/share/X11/xorg.conf.d/00-amdgpu.conf`:

```
Section "Device"
    Identifier "Card0"
    Driver "amdgpu"
    BusID "PCI:3841536:0:0"
EndSection

Section "Screen"
    Identifier "Screen0"
    Device "Card0"
    Monitor "Monitor0"
EndSection
```

Edit `/usr/share/X11/xorg.conf.d/10-amdgpu.conf`:

```
Section "OutputClass"
    Identifier "Card0"
    MatchDriver "amdgpu"
    Driver "amdgpu"
    Option "PrimaryGPU" "yes"
EndSection
```

Reboot and load the driver:

```bash
sudo reboot
```

After reboot:

```bash
sudo systemctl stop gdm
sudo modprobe amdgpu
sudo systemctl start gdm
```

### Start VNC Server

```bash
x11vnc --forever -find
```

> [!NOTE] 
> X11 configuration only works with Ubuntu Desktop images, not Server images.

---

## Troubleshooting

### Downgrade to Kernel 6.5

If you're running kernel 6.8, downgrade to 6.5 for compatibility:

Check loaded kernels:

```bash
dpkg --list | egrep -i --color 'linux-image|linux-headers|linux-modules' | awk '{ print $2 }'
```

Install kernel 6.5:

```bash
sudo apt install linux-image-6.5.0-1025-azure
```

Remove kernel 6.8:

```bash
sudo apt purge linux-headers-6.8.0-1025-azure linux-image-6.8.0-1025-azure linux-modules-6.8.0-1025-azure
```

Set kernel 6.5 as default by editing GRUB:

```bash
sudo vim /etc/default/grub
```

Change:

```
GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 6.5.0-1025-azure"
```

Update GRUB and reboot:

```bash
sudo update-grub
sudo reboot
```

Verify kernel version:

```bash
uname -a
```

---

## Uninstalling the Driver

To completely remove the AMD GPU driver:

```bash
dkms status
sudo amdgpu-install --uninstall
sudo amdgpu-uninstall
sudo apt autoremove --purge amdgpu-install
sudo reboot
```

Verify removal:

```bash
dkms status
```

---
