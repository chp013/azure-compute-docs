---
title: Azure Guest Health Reporting - Report Node Health #Required; page title is displayed in search results. Include the brand.
description: Share supercomputing VM device health status with Azure. #Required; article description that is displayed in search results. 
author: rolandnyamo #Required; your GitHub user alias, with correct capitalization.
ms.author: ronyamo #Required; microsoft alias of author; optional team alias.
ms.service: azure #Required; use either service or product per approved list. 
ms.topic: overview #Required; leave this attribute/value as-is.
ms.date: 09/18/2025 #Required; mm/dd/yyyy format.
ms.custom: template-overview #Required; leave this attribute/value as-is.
# Customer intent: As a cloud administrator, I want to utilize impact reporting tools to document performance issues in my Azure workloads, so that I can quickly identify and address platform-related problems to maintain service reliability.
---

# Report Guest Health Status (Preview)
> [!IMPORTANT]
> Guest Health Reporting is currently in Preview. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

Review health reporting [prerequisites](guest-health-overview.md#onboarding-process).

## REST Client Reporting
```
PUT https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Impact/workloadImpacts/{workloadImpactName}?api-version=2023-02-01-preview
```
Descriptions of URI parameters are as follows:

| **Field Name**       | **Description**       |
|---------------------|--------------------|
| subscriptionId  | Subscription previously allow-listed. |
| subscriptionId   | A unique name that would identify a specific impact. You can use a GUID as well.  |
| api-version   | API version to be used for this operation. Use `2023-02-01-preview`   |

### [Healthy Node](#tab/healthy/)

```json
{
  "properties": {
      "startDateTime": "2025-09-15T01:06:21.3886467Z",
      "impactCategory": "Resource.Hpc.Healthy",
      "impactDescription": "Missing GPU device",
      "impactedResourceId": "/subscriptions/111111-f1122-2233-11bc-bb00123/resourceGroups/<rg_name>/providers/Microsoft.Compute/virtualMachines/<vm_name>",
      "additionalProperties": {
            "PhysicalHostName": "GGBB90904476",
      }
   }
}

```
--
### [Missing GPU](#tab/missingGPU/)

```json
{
  "properties": {
      "startDateTime": "2025-09-15T01:06:21.3886467Z",
      "impactCategory": "Resource.Hpc.Unhealthy.HpcMissingGpu",
      "impactDescription": "Missing GPU device",
      "impactedResourceId": "/subscriptions/111111-f1122-2233-11bc-bb00123/resourceGroups/<rg_name>/providers/Microsoft.Compute/virtualMachines/<vm_name>",
      "additionalProperties": {
            "LogUrl": "https://someurl.blob.core.windows.net/rma",
            "PhysicalHostName": "GGBB90904476",
            "VmUniqueId": "1111111-22dr-3345-22rf-34454g89j", //GUID
            "Manufacturer": "Nvidia",
            "SerialNumber": "12345679",
            "ModelNumber": "NV3LB225",
            "Location": "0"
      }
   }
}

```
--
### [Investigate Node](#tab/investigate/)

```json
{
  "properties": {
      "startDateTime": "2025-09-15T01:06:21.3886467Z",
      "impactCategory": "Resource.Hpc.Investigate.NVLink",
      "impactDescription": "NvLink may be down",
      "impactedResourceId": "/subscriptions/111111-f1122-2233-11bc-bb00123/resourceGroups/<rg_name>/providers/Microsoft.Compute/virtualMachines/<vm_name>",
      "additionalProperties": {
            "LogUrl": "https://someurl.blob.core.windows.net/rma",
            "VmUniqueId": "1111111-22dr-3345-22rf-34454g89j", //GUID
            "CollectTelemtery": "0"
      }
   }
}

```
--
### [Unhealthy Non GPU](#tab/unhealthynongpu/)

```json
{
  "properties": {
      "startDateTime": "2025-09-15T01:06:21.3886467Z",
      "impactCategory": "Resource.Hpc.Unhealthy.IBPerformance",
      "impactDescription": "IB low bandwidth",
      "impactedResourceId": "/subscriptions/111111-f1122-2233-11bc-bb00123/resourceGroups/<rg_name>/providers/Microsoft.Compute/virtualMachines/<vm_name>",
      "additionalProperties": {
            "LogUrl": "https://someurl.blob.core.windows.net/rma",
            "PhysicalHostName": "GGBB90904476",
            "VmUniqueId": "1111111-22dr-3345-22rf-34454g89j"
      }
   }
}

```
--

| **Field Name**       | **Required** | **Data Type** | **Description**                                                                 |
|-----------------------|--------------|---------------|---------------------------------------------------------------------------------|
| startDateTime         | Y            | datetime      | Time (UTC) when the impact happened.                                           |
| impactCategory        | Y            | string        | Observation type/ Fault Scenario. Only approved string list allowed.           |
| impactDescription     | Y            | string        | Description of the reported impact.                                            |
| impactedResourceId    | Y            | string        | Fully qualified resource URI for the Azure resource.                             |
| physicalHostName      | Y            | string        | Node identifier, available in metadata.                                        |
| VmUniqueId            | Y            | string        | Virtual machine unique ID. Queryable inside VM.                                |
| logUrl                | N*           | string        | URL to saved logs.                                                             |
| manufacturer          | N*           | string        | GPU Manufacturer.                                                              |
| serialNumber          | N*           | string        | GPU serial number.                                                             |
| modelNumber           | N*           | string        | Model number.                                                                  |
| location              | N*           | string        | PCIe Location.                                                                 |

>[!NOTE]
> Providing optional information can speed up the node recovery time.
> PhysicalHostName can be retrieved from within the VM using this script: [Utilities/kvp_client.c at main·jeseszhang1010/Utilities·GitHub](https://github.com/jeseszhang1010/Utilities/blob/main/kvp_client.c)

**Use the following command to get the PhysicalHostName**
```shell
timeout 100 gcc -o /root/scripts/GPU/kvp_client /root/scripts/GPU/kvp_client.c
timeout 60 sudo /root/scripts/GPU/kvp_client | grep "PhysicalHostName;" | awk '{print$4}' | tee PhysicalHostName.txt
```

### HPC Additional Properties

To aid Guest Health Reporting in taking the correct action, you can provide more information about the issue using the additionalProperties field. <br>
`Resource.Hpc.*` fields:
* `LogUrl` (string) – URL to relevant log file
* `PhysicalHostName` (string) – physical host name of the node (alphanumeric)
* `VmUniqueId` (string) – virtual machine unique ID(GUID)

> [!IMPORTANT]
> All HPC impact requests must include either a PhysicalHostName or VmUniqueId (PhysicalHostName is preferred). The VM in question can be from any subscription and isn't limited to the VMs in the subscription that you're reporting from.

`Resource.Hpc.Unhealthy.*` fields that are specific to GPUs only:
* `Manufacturer` (string) – manufacturer of GPU
* `SerialNumber` (string) – serial number of GPU
* `ModelNumber` (string) – model number of GPU
* `Location` (string) – physical location of GPU

`Resource.Hpc.Investigate.*` fields:
* `CollectTelemetry` (Boolean - 0/1) – tell HPC to collect telemetry from the impacted VM

`gpu_row_remap_failure` field:
* SerialNumber – string – serial number of GPU
* Row remapping flag:
    * "`gpu_row_remap_failure`: GPU # (SXM# SN:#): row remap failure. This is an official end of life condition: decommission the GPU”

`gpu_row_remap_*` fields:
* `UCE` (string) - count of uncorrectable errors in histogram data
* `SerialNumber` (string) – serial number of GPU
    * “`gpu_row_remap_*`: GPU # (SXM# SN:#): bank with multiple row remaps: partial 1, low 0, none 0. CE: 0, UCE: #”

> [!IMPORTANT]
> Customers are advised to include detailed row remapping fields with the specified information in their claims to expedite node restoration.


## Next steps
<!-- Add a context sentence for the following links -->
* [What is Guest Health Reporting](guest-health-overview.md)
* [HPC Impact Categories](guest-health-impact-categories.md)
<!-- - [View previous impact reports](links-how-to.md) -->
