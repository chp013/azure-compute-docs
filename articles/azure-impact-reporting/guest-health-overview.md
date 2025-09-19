---
title: Azure HPC Guest Health Reporting - Overview 
description: Report Azure supercomputing VM device health status to Microsoft. 
author: rolandnyamo 
ms.author: ronyamo 
ms.service: azure 
ms.topic: overview 
ms.date: 09/18/2025 
ms.custom: template-overview 
---

# What is Guest Health Reporting (preview)?

> [!IMPORTANT]
> Guest Health Reporting is currently in preview. For legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability, see the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

The Guest Health Reporting service allows Azure supercomputing customers to provide virtual machine (VM) device health statuses to Azure. Based on these status updates, Azure high-performance computing (HPC) can make decisions to remove problematic nodes from production and send them for repair.

## Onboarding process

To use Guest Health Reporting to report the health of a node, you need to onboard the subscription that hosts the resources to the Impact service:

1. In the Azure portal, select **Subscriptions** and go to the relevant subscription. Then, on the left menu, select **Resource providers**.

   :::image type="content" source="images/guest-health-onboarding-subscription.png" alt-text="Screenshot that shows subscription settings with Resource Providers link.":::

2. Search for and select the **Microsoft.Impact** resource provider.

3. Select **Register**.

   :::image type="content" source="images/guest-health-registration.png" alt-text="Screenshot that shows the Microsoft.Impact RP selection and registration option.":::

4. On the left pane, select **Settings** > **Preview Features**. Search for and select **Allow Impact Reporting**, and then select **Register**.

   :::image type="content" source="images/guest-health-preview-feature-selection.png" alt-text="Screenshot that shows guest health reporting preview feature registration.":::

5. On to the left pane, go to **Settings** > **Overview**. Retrieve your subscription ID and send it to the Azure team member who's helping you complete the onboarding process.

6. Wait for confirmation that the onboarding process is complete before you proceed with submitting Guest Health Reporting requests.

## Access management and role assignment

To submit Guest Health Reporting requests from a resource within Azure, you must assign the appropriate access management roles:

1. Create a user-assigned or system-assigned managed identity.

2. On the left menu, go to **Access control (IAM)** and select **Add role assignment**.

   :::image type="content" source="images/guest-health-add-role.png" alt-text="Screenshot that shows how to add a role assignment.":::

3. Search for and select the **Impact reporter** role.

   :::image type="content" source="images/guest-health-impact-reporter-role.png" alt-text="Screenshot that shows the impact reporter role.":::

4. Go to the **Members** tab. Search for the user identity, app ID, or  service principal and select it. Then select **Members**. The app ID is the service principal for the app to be used for reporting.

5. Review and assign the app ID or managed identity.

## Related content

* [Report node health](guest-health-impact-report.md)
* [HPC Impact Categories](guest-health-impact-categories.md)
