---
title: How to resize disks encrypted using Azure Disk Encryption with Traditional LVM
description: This article provides instructions for resizing ADE encrypted disks by using logical volume management.
author: jofrance
ms.service: azure-virtual-machines
ms.subservice: disks
ms.custom: linux-related-content
ms.topic: how-to
ms.author: jofrance
ms.date: 10/27/2025
# Customer intent: "As a Linux system administrator, I want to resize Azure Disk Encryption-encrypted disks using logical volume management, so that I can efficiently manage storage capacity while maintaining data security."
---

# How to resize logical volume management devices that use Azure Disk Encryption

[!INCLUDE [Azure Disk Encryption retirement notice](~/reusable-content/ce-skilling/azure/includes/security/azure-disk-encryption-retirement.md)]

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

- Traditional LVM encryption
- Data disks only. OS disk resizing is not supported.

### Traditional LVM configurations

Traditional LVM configurations extend a logical volume (LV) when the volume group (VG) has available space.

### Traditional LVM encryption

In traditional LVM encryption, LVs are encrypted. The whole disk isn't encrypted.

By using traditional LVM encryption, you can:

- Extend the LV when the VG has available space
- Extend the LV when you add a new physical volume (PV).
- Extend the LV when you resize an existing PV.

For LVM on-crypt scenarios, please follow [How to resize encrypted LVM on crypt](how-to-resize-encrypted-lvm-on-crypt.md).

> [!NOTE]
> We don't recommend mixing traditional LVM encryption and LVM-on-crypt on the same VM.

The following sections provide examples of how to use LVM and LVM-on-crypt. The examples use preexisting values for disks, PVs, VGs, LVs, file systems, universally unique identifiers (UUIDs), and mount points. Replace these values with your own values to fit your environment.

#### Extend an LV when the VG has available space

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

#### Extend a traditional LVM volume by adding a new PV

When you need to add a new disk to increase the VG size, extend your traditional LVM volume by adding a new PV.

1. Verify the current size of the file system that you want to increase:

    ```bash
    df -h /mountpoint
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/005-resize-lvm-scenariob-check-fs.png" alt-text="Screenshot showing the code that checks the current size of a file system with the command and result highlighted.":::

2. Verify the current PV configuration:

    ```bash
    sudo pvs
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/006-resize-lvm-scenariob-check-pvs.png" alt-text="Screenshot showing the code that checks the current PV configuration with the command and result highlighted.":::

3. Check the current VG information:

    ```bash
    sudo vgs
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/007-resize-lvm-scenariob-check-vgs.png" alt-text="Screenshot showing the code that checks the current volume group information with the command and the result highlighted.":::

4. Check the current disk list. Identify data disks by checking the devices in */dev/disk/azure/scsi1/*.

    ```bash
    sudo ls -l /dev/disk/azure/scsi1/
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/008-resize-lvm-scenariob-check-scs1.png" alt-text="Screenshot showing the code that checks the current disk list with the command and results highlighted.":::

5. Check the output of `lsblk`:

    ```bash
    sudo lsbk
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/008-resize-lvm-scenariob-check-lsblk.png" alt-text="Screenshot showing the code that checks the output of l s b l k with the command and results highlighted.":::

6. Attach the new disk to the VM by following the instructions in [Attach a data disk to a Linux VM](attach-disk-portal.yml).

7. Check the disk list, and notice the new disk.

    ```bash
    sudo ls -l /dev/disk/azure/scsi1/
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/009-resize-lvm-scenariob-check-scsi12.png" alt-text="Screenshot showing the code that checks the disk list with the results highlighted.":::

    ```bash
    sudo lsblk
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/009-resize-lvm-scenariob-check-lsblk1.png" alt-text="Screenshot showing the code that checks the disk list by using l s b l k with the command and result highlighted.":::
   
9. Create a new PV on top of the new data disk:

    ```bash
    sudo pvcreate /dev/newdisk
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/010-resize-lvm-scenariob-pvcreate.png" alt-text="Screenshot showing the code that creates a new PV with the result highlighted.":::

    This method uses the whole disk as a PV without a partition. Alternatively, you can use `fdisk` to create a partition and then use that partition for `pvcreate`.

10. Verify that the PV was added to the PV list:

    ```bash
    sudo pvs
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/011-resize-lvm-scenariob-check-pvs1.png" alt-text="Screenshot showing the code that shows the physical volume list with the result highlighted.":::

11. Extend the VG by adding the new PV to it:

    ```bash
    sudo vgextend vgname /dev/newdisk
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/012-resize-lvm-scenariob-vgextend.png" alt-text="Screenshot showing the code that extends the volume group with the result highlighted.":::

12. Check the new VG size:

    ```bash
    sudo vgs
    ```

   :::image type="content" source="./media/disk-encryption/resize-lvm/013-resize-lvm-scenariob-check-vgs1.png" alt-text="Screenshot showing the code that checks the volume group size with the results highlighted.":::

13. Use `lsblk` to identify the LV that needs to be resized:

    ```bash
    sudo lsblk
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/013-resize-lvm-scenariob-check-lsblk1.png" alt-text="Screenshot showing the code that identifies the local volume that needs to be resized with the results highlighted.":::

14. Extend the LV size by using `-r` to increase the file system online:

    ```bash
    sudo lvextend -r -L +2G /dev/vgname/lvname
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/013-resize-lvm-scenariob-lvextend.png" alt-text="Screenshot showing code that increases the size of the file system online with the results highlighted.":::

15. Verify the new sizes of the LV and file system:

    ```bash
    df -h /mountpoint
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/014-resize-lvm-scenariob-check-fs1.png" alt-text="Screenshot showing the code that checks the sizes of the local volume and the file system with the command and result highlighted.":::

    >[!IMPORTANT]
    >When Azure Data Encryption is used on traditional LVM configurations, the encrypted layer is created at the LV level, not at the disk level.
    >
    >At this point, the encrypted layer is expanded to the new disk. The actual data disk has no encryption settings at the platform level, so its encryption status isn't updated.
    >
    >These are some of the reasons why LVM-on-crypt is the recommended approach.

16. Check the encryption information from the portal:

    :::image type="content" source="./media/disk-encryption/resize-lvm/014-resize-lvm-scenariob-check-portal1.png" alt-text="Screenshot showing encryption information in the portal with the disk name and encryption highlighted.":::

    To update the encryption settings on the disk, add a new LV and enable the extension on the VM.

17. Add a new LV, create a file system on it, and add it to `/etc/fstab`.

18. Set the encryption extension again. This time you'll stamp the encryption settings on the new data disk at the platform level. Here's a CLI example:

    ```azurecliazurecli-interactive
    az vm encryption enable -g ${RGNAME} --name ${VMNAME} --disk-encryption-keyvault "<your-unique-keyvault-name>"
    ```

19. Check the encryption information from the portal:

    :::image type="content" source="./media/disk-encryption/resize-lvm/014-resize-lvm-scenariob-check-portal2.png" alt-text="Screenshot showing encryption information in the portal with the disk name and the encryption information highlighted.":::

After the encryption settings are updated, you can delete the new LV. Also delete the entry from the `/etc/fstab` and `/etc/crypttab` that you created.

:::image type="content" source="./media/disk-encryption/resize-lvm/014-resize-lvm-scenariob-delete-fstab-crypttab.png" alt-text="Screenshot showing the code that deletes the new logical volume with the deleted F S tab and crypt tab highlighted.":::

Follow these steps to finish cleaning up:

1. Unmount the LV:

    ```bash
    sudo umount /mountpoint
    ```

1. Close the encrypted layer of the volume:

    ```bash
    sudo cryptsetup luksClose /dev/vgname/lvname
    ```

1. Delete the LV:

    ```bash
    sudo lvremove /dev/vgname/lvname
    ```

#### Extend a traditional LVM volume by resizing an existing PV

Im some scenarios, your limitations might require you to resize an existing disk. Here's how:

1. Identify your encrypted disks:

    ```bash
    sudo ls -l /dev/disk/azure/scsi1/
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/015-resize-lvm-scenarioc-check-scsi1.png" alt-text="Screenshot showing the code that identifies encrypted disks with the results highlighted.":::

    ```bash
    sudo lsblk -fs
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/015-resize-lvm-scenarioc-check-lsblk.png" alt-text="Screenshot showing alternative code that identifies encrypted disks with the results highlighted.":::

2. Check the PV information:

    ```bash
    sudo pvs
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/016-resize-lvm-scenarioc-check-pvs.png" alt-text="Screenshot showing the code that checks information about the physical volume with the results highlighted.":::

    The results in the image show that all of the space on all of the PVs is currently used.

3. Check the VG information:

    ```bash
    sudo vgs
    sudo vgdisplay -v vgname
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/017-resize-lvm-scenarioc-check-vgs.png" alt-text="Screenshot showing the code that checks information about the volume group with the results highlighted.":::

4. Check the disk sizes. You can use `fdisk` or `lsblk` to list the drive sizes.

    ```bash
    for disk in `sudo ls -l /dev/disk/azure/scsi1/* | awk -F/ '{print $NF}'` ; do echo "sudo fdisk -l /dev/${disk} | grep ^Disk "; done | bash

    sudo lsblk -o "NAME,SIZE"
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/018-resize-lvm-scenarioc-check-fdisk.png" alt-text="Screenshot showing the code that checks disk sizes with the results highlighted.":::

    Here we identified which PVs are associated with which LVs by using `lsblk -fs`. You can identify the associations by running `lvdisplay`.

    ```bash
    sudo lvdisplay --maps VG/LV
    sudo lvdisplay --maps datavg/datalv1
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/019-resize-lvm-scenarioc-check-lvdisplay.png" alt-text="Screenshot showing an alternative way to identify physical volume associations with local volumes with the results highlighted.":::

    In this case, all four data drives are part of the same VG and a single LV. Your configuration might differ.

5. Check the current file system utilization:

    ```bash
    df -h /datalvm*
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/020-resize-lvm-scenarioc-check-df.png" alt-text="Screenshot showing the code that checks file system utilization with the command and results highlighted.":::

6. Resize the data disks by following the instructions in [Expand an Azure managed disk](expand-disks.md#expand-an-azure-managed-disk). You can use the portal, the CLI, or PowerShell.

    >[!IMPORTANT]
    >Some data disks on Linux VMs can be resized without Deallocating the VM, please check [Expand virtual hard disks on a Linux VM](/azure/virtual-machines/linux/expand-disks? tabs=ubuntu#expand-an-azure-managed-disk) in order to verify your disks meet the requirements.

7. Start the VM and check the new sizes by using `fdisk`.

    ```bash
    for disk in `sudo ls -l /dev/disk/azure/scsi1/* | awk -F/ '{print $NF}'` ; do echo "sudo fdisk -l /dev/${disk} | grep ^Disk "; done | bash

    sudo lsblk -o "NAME,SIZE"
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/021-resize-lvm-scenarioc-check-fdisk1.png" alt-text="Screenshot showing the code that checks disk size with the result highlighted.":::

    In this case, `/dev/sdd` was resized from 5 G to 20 G.

8. Check the current PV size:

    ```bash
    sudo pvdisplay /dev/resizeddisk
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/022-resize-lvm-scenarioc-check-pvdisplay.png" alt-text="Screenshot showing the code that checks the size of the P V with the result highlighted.":::

    Even though the disk was resized, the PV still has the previous size.

9. Resize the PV:

    ```bash
    sudo pvresize /dev/resizeddisk
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/023-resize-lvm-scenarioc-check-pvresize.png" alt-text="Screenshot showing the code that resizes the physical volume with the result highlighted.":::


10. Check the PV size:

    ```bash
    sudo pvdisplay /dev/resizeddisk
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/024-resize-lvm-scenarioc-check-pvdisplay1.png" alt-text="Screenshot showing the code that checks the physical volume's size with the result highlighted.":::

    Apply the same procedure for all of the disks that you want to resize.

11. Check the VG information.

    ```bash
    sudo vgdisplay vgname
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/025-resize-lvm-scenarioc-check-vgdisplay1.png" alt-text="Screenshot showing the code that checks information for the volume group with the result highlighted.":::

    The VG now has enough space to be allocated to the LVs.

12. Resize the LV:

    ```bash
    sudo lvresize -r -L +5G vgname/lvname
    sudo lvresize -r -l +100%FREE /dev/datavg/datalv01
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/031-resize-lvm-scenarioc-check-lvresize1.png" alt-text="Screenshot showing the code that resizes the L V with the results highlighted.":::

13. Check the size of the file system:

    ```bash
    df -h /datalvm2
    ```

    :::image type="content" source="./media/disk-encryption/resize-lvm/032-resize-lvm-scenarioc-check-df3.png" alt-text="Screenshot showing the code that checks the size of the file system with the result highlighted.":::

## Next steps

[Troubleshoot Azure Disk Encryption](disk-encryption-troubleshooting.md)
