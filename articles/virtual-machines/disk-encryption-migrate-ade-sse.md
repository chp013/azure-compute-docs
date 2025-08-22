---
title: Migrate from Azure Disk Encryption to encryption at host
description: Learn how to migrate your virtual machines from Azure Disk Encryption (ADE) to encryption at  host before the ADE retirement date.
author: msmbaldwin
ms.date: 08/21/2025
ms.topic: how-to
ms.author: mbaldwin
ms.service: azure-virtual-machines
ms.subservice: security
ms.custom: references_regions
# Customer intent: As a security administrator, I need to migrate my VMs from Azure Disk Encryption to encryption at host before the ADE retirement date to avoid service disruption.
---

# Migrate from Azure Disk Encryption to encryption at host

This article provides step-by-step guidance for migrating your virtual machines from Azure Disk Encryption (ADE) to Encryption at Host. The migration process requires creating new disks and VMs, as in-place conversion is not supported.

## Migration overview

Azure Disk Encryption (ADE) encrypts data within the VM using BitLocker (Windows) or dm-crypt (Linux), while Encryption at Host encrypts data at the storage layer without consuming VM CPU resources. Encryption at Host provides end-to-end encryption for all VM data, including temp disks, caches, and data flows between compute and storage.

## Migration limitations and considerations

### Important limitations

- **No in-place migration**: You cannot directly convert ADE-encrypted disks to SSE. Migration requires creating new disks and VMs.

- **Linux OS disk limitation**: Disabling ADE on Linux OS disks is not supported. For Linux VMs with ADE-encrypted OS disks, you must create a new VM with a new OS disk.

- **Windows ADE disablement**: Azure Disk Encryption can be disabled on Windows VMs regardless of whether only data disks are encrypted or both OS and data disks are encrypted. This allows for a more straightforward migration path compared to Linux VMs.

- **Downtime required**: The migration process requires VM downtime for disk operations and VM recreation.

### Prerequisites

Before starting the migration:

1. **Backup your data**: Create backups of all critical data before beginning the migration process.

1. **Test the process**: If possible, test the migration process on a non-production VM first.

1. **Prepare encryption resources**: Ensure your VM size supports encryption at host. Most current VM sizes support this feature.

1. **Document configuration**: Record your current VM configuration, including network settings, extensions, and attached resources.

## Migration scenarios

The migration approach depends on your VM's operating system and which disks are encrypted:

| Scenario | Supported | Migration Method |
|----------|-----------|------------------|
| Windows VM - Data disks only encrypted | ✅ Yes | Disable ADE, copy disks, create new VM with Encryption at Host |
| Windows VM - OS and data disks encrypted | ✅ Yes | Disable ADE, copy disks, create new VM with Encryption at Host |
| Linux VM - Data disks only encrypted | ✅ Yes | Disable ADE, copy disks, attach to new/existing VM with Encryption at Host |
| Linux VM - OS disk encrypted | ❌ Limited | Create new VM with Encryption at Host, migrate data disks |

## Migration procedures

### Windows VM migration

This procedure works for Windows VMs where you can disable Azure Disk Encryption.

## Step 1: Disable Azure Disk Encryption

First, disable ADE and remove the encryption extension:

# [Azure PowerShell - Disable ADE](#tab/azure-powershell-disable)

```azurepowershell
# Disable encryption on the VM
Disable-AzVMDiskEncryption -ResourceGroupName "MyResourceGroup" -VMName "MyVM" -VolumeType "All"

# Check encryption status
Get-AzVmDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "MyVM"

# Remove the encryption extension after disks are decrypted
Remove-AzVMExtension -ResourceGroupName "MyResourceGroup" -VMName "MyVM" -Name "AzureDiskEncryption"
```

# [Azure CLI - Disable ADE](#tab/azure-cli-disable)

```azurecli
# Disable encryption on the VM
az vm encryption disable --resource-group "MyResourceGroup" --name "MyVM" --volume-type "all"

# Check encryption status
az vm encryption show --resource-group "MyResourceGroup" --name "MyVM"

# Remove the encryption extension after disks are decrypted
az vm extension delete --resource-group "MyResourceGroup" --vm-name "MyVM" --name "AzureDiskEncryption"
```

---

## Step 2: Create snapshots of the decrypted disks

# [Azure PowerShell - Snapshots](#tab/azure-powershell-snapshots)

```azurepowershell
# Get the VM
$vm = Get-AzVM -ResourceGroupName "MyResourceGroup" -Name "MyVM"

# Create snapshot of OS disk
$osDisk = Get-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName $vm.StorageProfile.OsDisk.Name
$osSnapshotConfig = New-AzSnapshotConfig -SourceUri $osDisk.Id -Location $vm.Location -CreateOption Copy
$osSnapshot = New-AzSnapshot -Snapshot $osSnapshotConfig -SnapshotName "MyVM-OS-Snapshot" -ResourceGroupName "MyResourceGroup"

# Create snapshots of data disks
foreach ($dataDisk in $vm.StorageProfile.DataDisks) {
    $disk = Get-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName $dataDisk.Name
    $snapshotConfig = New-AzSnapshotConfig -SourceUri $disk.Id -Location $vm.Location -CreateOption Copy
    $snapshot = New-AzSnapshot -Snapshot $snapshotConfig -SnapshotName "$($dataDisk.Name)-Snapshot" -ResourceGroupName "MyResourceGroup"
}
```

# [Azure CLI - Snapshots](#tab/azure-cli-snapshots)

```azurecli
# Get VM information
vm_info=$(az vm show --resource-group "MyResourceGroup" --name "MyVM")
location=$(echo $vm_info | jq -r '.location')

# Create snapshot of OS disk
os_disk_name=$(echo $vm_info | jq -r '.storageProfile.osDisk.name')
az snapshot create \
    --resource-group "MyResourceGroup" \
    --name "MyVM-OS-Snapshot" \
    --source $os_disk_name

# Create snapshots of data disks
data_disks=$(echo $vm_info | jq -r '.storageProfile.dataDisks[].name')
for disk in $data_disks; do
    az snapshot create \
        --resource-group "MyResourceGroup" \
        --name "${disk}-Snapshot" \
        --source $disk
done
```

---

## Step 3: Create new managed disks

# [Azure PowerShell - New Disks](#tab/azure-powershell-disks)

```azurepowershell
# Create new OS disk from snapshot (will be encrypted with Encryption at Host)
$osSnapshotId = $osSnapshot.Id
$osDiskConfig = New-AzDiskConfig -Location $vm.Location -DiskSizeGB $osDisk.DiskSizeGB -SourceResourceId $osSnapshotId -CreateOption Copy
$newOsDisk = New-AzDisk -Disk $osDiskConfig -ResourceGroupName "MyResourceGroup" -DiskName "MyVM-OS-New"

# Create new data disks from snapshots (will be encrypted with Encryption at Host)
foreach ($dataDisk in $vm.StorageProfile.DataDisks) {
    $snapshotName = "$($dataDisk.Name)-Snapshot"
    $snapshot = Get-AzSnapshot -ResourceGroupName "MyResourceGroup" -SnapshotName $snapshotName
    $diskConfig = New-AzDiskConfig -Location $vm.Location -DiskSizeGB $dataDisk.DiskSizeGB -SourceResourceId $snapshot.Id -CreateOption Copy
    $newDisk = New-AzDisk -Disk $diskConfig -ResourceGroupName "MyResourceGroup" -DiskName "$($dataDisk.Name)-New"
}
```

# [Azure CLI - New Disks](#tab/azure-cli-disks)

```azurecli
# Create new OS disk from snapshot (will be encrypted with Encryption at Host)
os_snapshot_id=$(az snapshot show --resource-group "MyResourceGroup" --name "MyVM-OS-Snapshot" --query "id" -o tsv)
az disk create \
    --resource-group "MyResourceGroup" \
    --name "MyVM-OS-New" \
    --source $os_snapshot_id

# Create new data disks from snapshots (will be encrypted with Encryption at Host)
for disk in $data_disks; do
    snapshot_id=$(az snapshot show --resource-group "MyResourceGroup" --name "${disk}-Snapshot" --query "id" -o tsv)
    az disk create \
        --resource-group "MyResourceGroup" \
        --name "${disk}-New" \
        --source $snapshot_id
done
```

---

## Step 4: Create new VM with encrypted disks

# [Azure PowerShell - New VM](#tab/azure-powershell-vm)

```azurepowershell
# Create new VM configuration
$vmConfig = New-AzVMConfig -VMName "MyVM-New" -VMSize $vm.HardwareProfile.VmSize

# Set OS disk
$vmConfig = Set-AzVMOSDisk -VM $vmConfig -ManagedDiskId $newOsDisk.Id -CreateOption Attach -Windows

# Add data disks
foreach ($dataDisk in $vm.StorageProfile.DataDisks) {
    $newDiskName = "$($dataDisk.Name)-New"
    $newDisk = Get-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName $newDiskName
    $vmConfig = Add-AzVMDataDisk -VM $vmConfig -ManagedDiskId $newDisk.Id -Lun $dataDisk.Lun -CreateOption Attach
}

# Enable encryption at host (required for migration from ADE)
$vmConfig = Set-AzVMSecurityProfile -VM $vmConfig -EncryptionAtHost $true

# Set network configuration
$nic = Get-AzNetworkInterface -ResourceId $vm.NetworkProfile.NetworkInterfaces[0].Id
$vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id

# Create the new VM
New-AzVM -ResourceGroupName "MyResourceGroup" -Location $vm.Location -VM $vmConfig
```

# [Azure CLI - New VM](#tab/azure-cli-vm)

```azurecli
# Get original VM size and NIC
vm_size=$(echo $vm_info | jq -r '.hardwareProfile.vmSize')
nic_id=$(echo $vm_info | jq -r '.networkProfile.networkInterfaces[0].id')

# Create new VM with Encryption at Host enabled
os_disk_id=$(az disk show --resource-group "MyResourceGroup" --name "MyVM-OS-New" --query "id" -o tsv)

az vm create \
    --resource-group "MyResourceGroup" \
    --name "MyVM-New" \
    --location $location \
    --size $vm_size \
    --nics $nic_id \
    --attach-os-disk $os_disk_id \
    --os-type Windows \
    --encryption-at-host true

# Attach new data disks
for disk in $data_disks; do
    disk_id=$(az disk show --resource-group "MyResourceGroup" --name "${disk}-New" --query "id" -o tsv)
    # Get original LUN number
    lun=$(echo $vm_info | jq -r ".storageProfile.dataDisks[] | select(.name==\"$disk\") | .lun")
    az vm disk attach \
        --resource-group "MyResourceGroup" \
        --vm-name "MyVM-New" \
        --disk $disk_id \
        --lun $lun
done
```

---

## Linux VM migration

For Linux VMs, the approach depends on whether the OS disk is encrypted:

### Data disks only encrypted

If only data disks are encrypted, follow the same procedure as Windows VMs:

1. [Disable ADE on data disks](linux/disk-encryption-linux.md#disable-encryption-and-remove-the-encryption-extension)
2. Create snapshots of decrypted data disks
3. Create new managed disks
4. Create a new VM with Encryption at Host enabled and attach the new disks

### OS disk encrypted (Linux)

Since disabling ADE on Linux OS disks is not supported, you must create a new VM:

1. Create a new VM with a fresh OS disk and Encryption at Host enabled
2. [Disable ADE on the data disks](linux/disk-encryption-linux.md#disable-encryption-and-remove-the-encryption-extension) of the original VM
3. Create snapshots and new data disks as described above
4. Attach the new data disks to the new VM with Encryption at Host
5. Migrate your data and configuration from the old VM to the new VM

## Post-migration verification

After completing the migration, verify that Encryption at Host is working correctly:

1. **Check Encryption at Host status**: Verify that Encryption at Host is enabled:

   ```azurecli
   az vm show --resource-group "MyResourceGroup" --name "MyVM-New" --query "securityProfile.encryptionAtHost"
   ```

2. **Test VM functionality**: Ensure your applications and services are working correctly.

3. **Verify disk encryption**: Confirm that disks are properly encrypted:

   ```azurepowershell
   Get-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName "MyVM-OS-New" | Select-Object Name, DiskState
   ```

4. **Monitor performance**: Compare performance before and after migration to confirm the expected improvements.

## Cleanup

After successful migration and verification:

1. **Delete old VM**: Remove the original ADE-encrypted VM
2. **Delete old disks**: Remove the original encrypted disks
3. **Delete snapshots**: Clean up temporary snapshots (optional, but recommended for cost savings)
4. **Update documentation**: Update your infrastructure documentation to reflect the migration to Encryption at Host

## Common issues and solutions

### VM size doesn't support Encryption at Host

**Solution**: Check the [list of supported VM sizes](disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data) and resize your VM if necessary

### VM fails to start after migration

**Solution**: Check that all disks are properly attached and that the OS disk is set as the boot disk

### Encryption at Host not enabled

**Solution**: Verify that the VM was created with the `--encryption-at-host true` parameter and that your subscription supports this feature

### Performance issues persist

**Solution**: Verify that encryption at host is properly enabled and that the VM size supports the expected performance.

## Next steps

- [Encryption at host - End-to-end encryption for your VM data](disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data)
- [Overview of managed disk encryption options](disk-encryption-overview.md)
- [Server-side encryption of Azure Disk Storage](disk-encryption.md)
- [Azure Disk Encryption for Windows VMs](windows/disk-encryption-overview.md)
- [Azure Disk Encryption for Linux VMs](linux/disk-encryption-overview.md)
