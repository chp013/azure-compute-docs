---
title: Azure HPC Guest Health Reporting - FAQ #Required; page title is displayed in search results. Include the brand.
description: Frequently asked questions for Azure Guest Health Reporting. #Required; article description that is displayed in search results. 
author: rolandnyamo #Required; your GitHub user alias, with correct capitalization.
ms.author: ronyamo #Required; microsoft alias of author; optional team alias.
ms.topic: faq #Required; leave this attribute/value as-is.
ms.service: azure #Required; use either service or product per approved list. 
ms.date: 09/18/2025 #Required; mm/dd/yyyy format
ms.custom: template-overview #Required; leave this attribute/value as-is.
# Customer intent: As an Azure administrator, I want to troubleshoot and configure Azure Impact Reporting Connectors, so that I can ensure successful integration with Azure Monitor and manage permissions effectively.
---

# Azure Guest Health Reporting FAQ (Preview)
> [!IMPORTANT]
> Azure Guest Health Reporting is currently in Preview. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

Here are answers to common questions about Azure Guest Health Reporting.

## What happens if I don’t deallocate the node after sending the request to GHR?

For regular GHR request to UA/OFR the node, if customer doesn't deallocate VMs in 30 days after the node is UA, the node will automatically get into HI (HumanInvestigate). For reset request, there's no timeout as it doesn't require customers to deallocate VMs. For reboot request, if customer doesn't deallocate VMs in 30 days after the node is UA, the node will be set to Available, means the customer's request to reboot the node will get ignored

## How do I upload logs?

1. Get access token to customers storage account/container via
`/subscriptions/[subscriotionId]/providers/Microsft.Impact/getUploadtoken?api-version=2025-01-01preview`

2. Upload logs using the upload URL/token
    ```bash
    az storage blob upload –file “path/to/local/file.zip” –blob-url
    https://[storageAccount].blob.core.windows.net/[container]/[datetime]_[randomHash].zip?[SasToken]
    ```
3. Trim off SAS token and send report with `LogUrl` filed
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

## Next steps
* [What is Guest Health Reporting](ghr-overview.md)
* [Report node health](ghr-impact-report.md)
