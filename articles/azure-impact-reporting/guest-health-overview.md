---
title: Azure HPC Guest Health Reporting - Overview #Required; page title is displayed in search results. Include the brand.
description: Report Azure supercomputing VM device health status to Microsoft. #Required; article description that is displayed in search results. 
author: rolandnyamo #Required; your GitHub user alias, with correct capitalization.
ms.author: ronyamo #Required; microsoft alias of author; optional team alias.
ms.service: azure #Required; use either service or product per approved list. 
ms.topic: overview #Required; leave this attribute/value as-is.
ms.date: 09/18/2025 #Required; mm/dd/yyyy format.
ms.custom: template-overview #Required; leave this attribute/value as-is.
# Customer intent: As a cloud administrator, I want to utilize impact reporting tools to document performance issues in my Azure workloads, so that I can quickly identify and address platform-related problems to maintain service reliability.
---

# What is Guest Health Reporting (GHR)? (Preview)
> [!IMPORTANT]
> Guest Health Reporting is currently in Preview. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

The Guest Health Reporting service allows Azure supercomputing customers to provide VM device health statuses to Azure. Based on these status updates, Azure HPC can make decisions to remove problematic nodes out of production and send them for repair.

## Onboarding Process

To use Guest Health Reporting to report the health of a node, the subscription that hosts the resources needs to be onboarded to the Impact service using the following steps:

1. Go to the Azure portal -> Subscriptions (select) -> Resource Providers in the left menu. <br>
[ ![ASubscription settings showing Resource Providers link.](images/guest-health-onboarding-subscription.png) ](images/guest-health-onboarding-subscription.png#lightbox)
2. Search for the `Microsoft.Impact` resource provider
3. Select and Register it.<br>
[ ![Microsoft.Impact RP selection and registration.](images/guest-health-registration.png) ](images/guest-health-registration.png#lightbox)
4. Once registered, in the left pane select Settings -> Preview Features, search for "Allow Impact Reporting", select it and select "Register". <br>
[ ![GHR preview feature registration.](images/guest-health-preview-feature-selection.png) ](images/guest-health-preview-feature-selection.png#lightbox)
5. Go to the left pane -> Settings -> Overview and retrieve your Subscription ID and send it to the Azure team member assisting you to complete the onboarding process.
6. **Wait for confirmation that the onboarding process is complete before proceeding with using GHR requests submission.**

## Access Management and Role assignment

To submit GHR requests from a resource within Azure, the appropriate access management roles must be assigned.
1. Create a User or System Assigned Managed Identity 
2. Go to Access Control (IAM) in the left menu -> select Add Role Assignment. <br>
[ ![GHR add a role assignment.](images/guest-health-add-role.png) ](images/guest-health-add-role.png#lightbox)
3. Search for `Impact Reporter` role in the search box and select it. <br>
[ ![GHR impact reporter role.](images/guest-health-impact-reporter-role.png) ](images/guest-health-impact-reporter-role.png#lightbox)
4. Go to the Members tab and search for the user-identity/ app-id/ service-principal in the search box and select it -> Select Members. The app-ID is the service-principal for the app to be used for reporting. 
5. Once the app-id/ managed-identity is selected, review-assign it.


## Next steps
<!-- Add a context sentence for the following links -->
* [Report node health](guest-health-impact-report.md)
* [HPC Impact Categories](guest-health-impact-categories.md)
<!-- - [View previous impact reports](links-how-to.md) -->
