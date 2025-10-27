
---
title: How to resize disks encrypted using Azure Disk Encryption (LVM-on Crypt)
description: This article provides instructions for resizing ADE encrypted disks by using logical volume management.
author: jofrance
ms.service: azure-virtual-machines
ms.subservice: disks
ms.custom: linux-related-content
ms.topic: how-to
ms.author: jofrance
ms.date: 10/27/2025
---
# How to resize logical volume management devices that use Azure Disk Encryption (LVM-on Crypt)
**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets
In this article, you'll learn how to resize data disks that use Azure Disk Encryption. To resize the disks, you'll use logical volume management (LVM) on Linux. The steps apply to multiple scenarios.
You can use this resizing process in the following environments:
- Linux distributions:
    - Red Hat Enterprise Linux (RHEL) 7 or later
    - Ubuntu 18.04 or later
    - SUSE 12 or later
- Azure Disk Encryption versions:
    - Single-pass extension
    - Dual-pass extension
## Prerequisites
This article assumes that you have:
- An existing LVM configuration. For more information, see [Configure LVM on a Linux VM](/previous-versions/azure/virtual-machines/linux/configure-lvm).
- Disks that are already encrypted by Azure Disk Encryption. For more information, see [Configure LVM and RAID on encrypted devices](how-to-configure-lvm-raid-on-crypt.md).
- Experience using Linux and LVM.
- Experience using */dev/disk/scsi1/* paths for data disks on Azure. For more information, see [Troubleshoot Linux VM device name problems](/troubleshoot/azure/virtual-machines/troubleshoot-device-names-problems).
## Scenarios
The procedures in this article apply to the following scenarios:
- LVM-on-crypt configurations
- Data disks only. OS disk resizing is not supported.
## Overview
The recommended method for disk encryption is LVM-on-encrypt. This method encrypts the entire disk, not just the LV.
By using LVM-on-crypt, you can:
- Extend an LV when the VG has available space
- Extend the LV when you add a new PV.
- Extend the LV when you resize an existing PV.
> [!NOTE]
> We don't recommend mixing traditional LVM encryption and LVM-on-crypt on the same VM.
The following sections provide examples of how to use LVM-on-crypt. The examples use preexisting values for disks, PVs, VGs, LVs, file systems, universally unique identifiers (UUIDs), and mount points. Replace these values with your own values to fit your environment.
For Traditional LVM steps, please follow documentation: [How to resize encrypted LVM](how-to-encrypted-lvm.md).

### Extend an LV on-crypt volume when the VG has available space
The traditional way to resize LVs is to extend an LV when the VG has space available. You can use this method for nonencrypted disks, traditional LVM-encrypted volumes, and LVM-on-crypt configurations.
1. Verify the current size of the file system that you want to increase:
    ```bash
    df -h /mountpoint
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/001-resize-lvm-scenarioa-check-fs.png" alt-text="Screenshot showing code that checks the size of the file system with the command and the result highlighted.":::
2. Verify that the VG has enough space to increase the LV:
    ```bash
    sudo vgs
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/002-resize-lvm-scenarioa-check-vgs.png" alt-text="Screenshot showing the code that checks for space on the VG with the command and the result highlighted.":::
    You can also use `vgdisplay`:
    ```bash
    sudo vgdisplay vgname
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/002-resize-lvm-scenarioa-check-vgdisplay.png" alt-text="Screenshot showing the V G display code that checks for space on the VG with the command and result highlighted.":::
3. Identify which LV needs to be resized:
    ```bash
    sudo lsblk
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/002-resize-lvm-scenarioa-check-lsblk1.png" alt-text="Screenshot showing the result of the l s b l k command with the command and result highlighted.":::
    For LVM-on-crypt, the difference is that this output shows that the encrypted layer is at the disk level.
    :::image type="content" source="./media/disk-encryption/resize-lvm/002-resize-lvm-scenarioa-check-lsblk2.png" alt-text="Screenshot showing the result of the l s b l k command with the output highlighted and showing the encrypted layer.":::
4. Check the LV size:
    ```bash
    sudo lvdisplay lvname
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/002-resize-lvm-scenarioa-check-lvdisplay01.png" alt-text="Screenshot showing the code that checks the logical volume size with the command and result highlighted.":::
5. Increase the LV size by using `-r` to resize the file system online:
    ```bash
    sudo lvextend -r -L +2G /dev/vgname/lvname
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/003-resize-lvm-scenarioa-resize-lv.png" alt-text="Screenshot showing the code that increases the size of the logical volume with the command and results highlighted.":::
6. Verify the new sizes for the LV and the file system:
    ```bash
    df -h /mountpoint
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/004-resize-lvm-scenarioa-check-fs.png" alt-text="Screenshot showing the code that verifies the size of the LV and the file system with the command and result highlighted.":::
    The size output indicates that the LV and file system were successfully resized.
You can check the LV information again to confirm the changes at the level of the LV:
```bash
sudo lvdisplay lvname
```
:::image type="content" source="./media/disk-encryption/resize-lvm/004-resize-lvm-scenarioa-check-lvdisplay2.png" alt-text="Screenshot showing the code that confirms the new sizes with the sizes highlighted.":::

### Extend an LVM-on-crypt volume by adding a new PV
You can also extend an LVM-on-crypt volume by adding a new PV. This method closely follows the steps in [Configure LVM and RAID on encrypted devices](how-to-configure-lvm-raid-on-crypt.md#general-steps). See the sections that explain how to add a new disk and set it up in an LVM-on-crypt configuration.
You can use this method to add space to an existing LV. Or you can create new VGs or LVs.
1. Verify the current size of your VG:
    ```bash
    sudo vgdisplay vgname
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/033-resize-lvm-scenarioe-check-vg01.png" alt-text="Screenshot showing the code that checks the volume group size with results highlighted.":::
2. Verify the size of the file system and LV that you want to expand:
    ```bash
    sudo lvdisplay /dev/vgname/lvname
    ```
    ![Screenshot showing the code that checks the size of the local volume. Results are highlighted.](./media/disk-encryption/resize-lvm/034-resize-lvm-scenarioe-check-lv01.png)
    ```bash
    df -h mountpoint
    ```
   :::image type="content" source="./media/disk-encryption/resize-lvm/034-resize-lvm-scenarioe-check-fs01.png" alt-text="Screenshot showing the code that checks the file system's size with the result highlighted.":::
3. Add a new data disk to the VM and identify it.
    Before you add the new disk, check the disks:
    ```bash
    sudo fdisk -l | egrep ^"Disk /"
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/035-resize-lvm-scenarioe-check-newdisk01.png" alt-text="Screenshot showing the code that checks the size of the disks with the result highlighted.":::
    Here's another way to check the disks before you add the new disk:
    ```bash
    sudo lsblk
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/035-resize-lvm-scenarioe-check-newdisk02.png" alt-text="Screenshot showing an alternative code that checks the size of the disks with the results highlighted.":::
    To add the new disk, you can use PowerShell, the Azure CLI, or the Azure portal. For more information, see [Attach a data disk to a Linux VM](attach-disk-portal.yml).
    The kernel name scheme applies to the newly added device. A new drive is normally assigned the next available letter. In this case, the added disk is `sdd`.
4. Check the disks to make sure the new disk has been added:
    ```bash
    sudo fdisk -l | egrep ^"Disk /"
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/036-resize-lvm-scenarioe-check-newdisk02.png" alt-text="Screenshot showing the code that lists the disks with the results highlighted.":::
    ```bash
    sudo lsblk
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/036-resize-lvm-scenarioe-check-newdisk03.png" alt-text="Screenshot showing the newly added disk in the output.":::
5. Create a file system on top of the recently added disk. Match the disk to the linked devices on `/dev/disk/azure/scsi1/`.
    ```bash
    sudo ls -la /dev/disk/azure/scsi1/
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/037-resize-lvm-scenarioe-check-newdisk03.png" alt-text="Screenshot showing the code that creates a file system with the results highlighted.":::
    ```bash
    sudo mkfs.ext4 /dev/disk/azure/scsi1/${disk}
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-mkfs01.png" alt-text="Screenshot showing additional code that creates a file system and matches the disk to the linked devices with the results highlighted.":::
6. Create a temporary mount point for the new added disk:
    ```bash
    newmount=/data4
    sudo mkdir ${newmount}
    ```
7. Add the recently created file system to `/etc/fstab`.
    ```bash
    sudo blkid /dev/disk/azure/scsi1/lun4| awk -F\" '{print "UUID="$2" '${newmount}' "$4" defaults,nofail 0 0"}' >> /etc/fstab
    ```
8. Mount the newly created file system:
    ```bash
    sudo mount -a
    ```
9. Verify that the new file system is mounted:
    ```bash
    df -h
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-df.png" alt-text="Screenshot showing the code that verifies that the file system is mounted with the result highlighted.":::
    ```bash
    sudo lsblk
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-lsblk.png" alt-text="Screenshot showing additional code that verifies that the file system is mounted with the result highlighted.":::
10. Restart the encryption that you previously started for data drives.
    >[!TIP]
    >For LVM-on-crypt, we recommend that you use `EncryptFormatAll`. Otherwise, you might see a double encryption while you set additional disks.
    >
    >For more information, see [Configure LVM and RAID on encrypted devices](how-to-configure-lvm-raid-on-crypt.md).
    Here's an example:
    ```azurecli-interactive
    az vm encryption enable \
    --resource-group ${RGNAME} \
    --name ${VMNAME} \
    --disk-encryption-keyvault ${KEYVAULTNAME} \
    --key-encryption-key ${KEYNAME} \
    --key-encryption-keyvault ${KEYVAULTNAME} \
    --volume-type "DATA" \
    --encrypt-format-all \
    -o table
    ```
    When the encryption finishes, you see a crypt layer on the newly added disk:
    ```bash
    sudo lsblk
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-lsblk2.png" alt-text="Screenshot showing the code that checks the crypt layer with the result highlighted.":::
11. Unmount the encrypted layer of the new disk:
    ```bash
    sudo umount ${newmount}
    ```
12. Check the current PV information:
    ```bash
    sudo pvs
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-currentpvs.png" alt-text="Screenshot showing the code that checks information about the physical volume with the result highlighted.":::
13. Create a PV on top of the encrypted layer of the disk. Take the device name from the previous `lsblk` command. Add a `/dev/` mapper in front of the device name to create the PV:
    ```bash
    sudo pvcreate /dev/mapper/mapperdevicename
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-pvcreate.png" alt-text="Screenshot showing the code that creates a physical volume on the encrypted layer with the results highlighted.":::
    You see a warning about wiping the current `ext4 fs` signature. This warning is expected. Answer this question with `y`.
14. Verify that the new PV was added to the LVM configuration:
    ```bash
    sudo pvs
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-newpv.png" alt-text="Screenshot showing the code that verifies that the physical volume was added to the LVM configuration with the result highlighted.":::
15. Add the new PV to the VG that you need to increase.
    ```bash
    sudo vgextend vgname /dev/mapper/nameofhenewpv
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-vgextent.png" alt-text="Screenshot showing the code that adds a physical volume to a volume group with the results highlighted.":::
16. Verify the new size and free space of the VG:
    ```bash
    sudo vgdisplay vgname
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-vgdisplay.png" alt-text="Screenshot showing the code that verifies the size and free space of the volume group with the results highlighted.":::
    Note the increase of the `Total PE` count and the `Free PE / Size`.
17. Increase the size of the LV and the file system. Use the `-r` option on `lvextend`. In this example, we're adding the total available space in the VG to the given LV.
    ```bash
    sudo lvextend -r -l +100%FREE /dev/vgname/lvname
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-lvextend.png" alt-text="Screenshot showing the code that increases the size of the local volume and the file system with the results highlighted.":::
Follow the next steps to verify your changes.
1. Verify the size of the LV:
    ```bash
    sudo lvdisplay /dev/vgname/lvname
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-lvdisplay.png" alt-text="Screenshot showing the code that verifies the new size of the local volume with the results highlighted.":::
2. Verify the new size of the file system:
    ```bash
    df -h /mountpoint
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-df1.png" alt-text="Screenshot showing the code that verifies the new size of the file system with the result highlighted.":::
3. Verify that the LVM layer is on top of the encrypted layer:
    ```bash
    sudo lsblk
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-lsblk3.png" alt-text="Screenshot showing the code that verifies that the LVM layer is on top of the encrypted layer with the result highlighted.":::
    If you use `lsblk` without options, then you see the mount points multiple times. The command sorts by device and LVs.
    You might want to use `lsblk -fs`. In this command, `-fs` reverses the sort order so that the mount points are shown once. The disks are shown multiple times.
    ```bash
    sudo lsblk -fs
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/038-resize-lvm-scenarioe-lsblk4.png" alt-text="Screenshot showing alternative code that verifies that the LVM layer is on top of the encrypted layer with the result highlighted.":::
### Extend an LVM on a crypt volume by resizing an existing PV
1. Identify your encrypted disks:
    ```bash
    sudo lsblk
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/039-resize-lvm-scenariof-lsblk01.png" alt-text="Screenshot showing the code that identifies the encrypted disks with the results highlighted.":::
    ```bash
    sudo lsblk -s
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/040-resize-lvm-scenariof-lsblk012.png" alt-text="Screenshot showing alternative code that identifies the encrypted disks with the results highlighted.":::
2. Check your PV information:
    ```bash
    sudo pvs
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/041-resize-lvm-scenariof-pvs.png" alt-text="Screenshot showing the code that checks information for physical volumes with the results highlighted.":::
3. Check your VG information:
    ```bash
    sudo vgs
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/042-resize-lvm-scenariof-vgs.png" alt-text="Screenshot showing the code that checks information for volume groups with the results highlighted.":::
4. Check your LV information:
    ```bash
    sudo lvs
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/043-resize-lvm-scenariof-lvs.png" alt-text="Screenshot showing the code that checks information for the local volume with the result highlighted.":::
5. Check the file system utilization:
    ```bash
    df -h /mountpoint(s)
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/044-resize-lvm-scenariof-fs.png" alt-text="Screenshot showing the code that checks how much of the file system is being used with the results highlighted.":::
6. Check the sizes of your disks:
    ```bash
    sudo fdisk
    sudo fdisk -l | egrep ^"Disk /"
    sudo lsblk
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/045-resize-lvm-scenariof-fdisk01.png" alt-text="Screenshot showing the code that checks the size of disks with the results highlighted.":::
7. Resize the data disk. You can use the portal, CLI, or PowerShell. For more information, see the disk-resize section in [Expand virtual hard disks on a Linux VM](expand-disks.md#expand-an-azure-managed-disk).
    >[!IMPORTANT]
    >You can't resize virtual disks while the VM is running. Deallocate your VM for this step.
8. Check your disks sizes:
    ```bash
    sudo fdisk
    sudo fdisk -l | egrep ^"Disk /"
    sudo lsblk
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/046-resize-lvm-scenariof-fdisk02.png" alt-text="Screenshot showing code that checks disk sizes with the results highlighted.":::
    In this case, both disks were resized from 2 GB to 4 GB. But the size of the file system, LV, and PV remain the same.
9. Check the current PV size. Remember that on LVM-on-crypt, the PV is the `/dev/mapper/` device, not the `/dev/sd*` device.
    ```bash
    sudo pvdisplay /dev/mapper/devicemappername
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/047-resize-lvm-scenariof-pvs.png" alt-text="Screenshot showing the code that checks the size of the current physical volume with the results highlighted.":::
10. Resize the PV:
    ```bash
    sudo pvresize /dev/mapper/devicemappername
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/048-resize-lvm-scenariof-resize-pv.png" alt-text="Screenshot showing the code that resizes the physical volume with the results highlighted.":::
11. Check the new PV size:
    ```bash
    sudo pvdisplay /dev/mapper/devicemappername
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/049-resize-lvm-scenariof-pv.png" alt-text="Screenshot showing the code that checks the size of the physical volume with the results highlighted.":::
12. Resize the encrypted layer on the PV:
    ```bash
    sudo cryptsetup resize /dev/mapper/devicemappername
    ```
    Apply the same procedure for all of the disks that you want to resize.
13. Check your VG information:
    ```bash
    sudo vgdisplay vgname
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/050-resize-lvm-scenariof-vg.png" alt-text="Screenshot showing the code that checks information for the volume group with the results highlighted.":::
    The VG now has enough space to be allocated to the LVs.
14. Check the LV information:
    ```bash
    sudo lvdisplay vgname/lvname
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/051-resize-lvm-scenariof-lv.png" alt-text="Screenshot showing the code that checks information for the local volume with the results highlighted.":::
15. Check the file system utilization:
    ```bash
    df -h /mountpoint
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/052-resize-lvm-scenariof-fs.png" alt-text="Screenshot showing the code that checks utilization of the file system with the results highlighted.":::
16. Resize the LV:
    ```bash
    sudo lvresize -r -L +2G /dev/vgname/lvname
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/053-resize-lvm-scenariof-lvresize.png" alt-text="Screenshot showing the code that resizes the local volume with the results highlighted.":::
    Here we use the `-r` option to also resize the file system.
17. Check the LV information:
    ```bash
    sudo lvdisplay vgname/lvname
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/054-resize-lvm-scenariof-lvsize.png" alt-text="Screenshot showing the code that gets information about the local volume with the results highlighted.":::
18. Check the file system utilization:
    ```bash
    df -h /mountpoint
    ```
    :::image type="content" source="./media/disk-encryption/resize-lvm/055-resize-lvm-scenariof-fs.png" alt-text="Screenshot showing the code that checks the file system utilization with the results highlighted.":::
Apply the same resizing procedure to any other LV that requires it.
## Next steps
[Troubleshoot Azure Disk Encryption](disk-encryption-troubleshooting.md)
