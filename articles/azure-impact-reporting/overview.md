---
title: 'Azure Impact Reporting: Overview' #Required; page title is displayed in search results. Include the brand.
description: Azure Impact Reporting is a service that you can use to report observed performance and availability regressions with your Azure workloads. #Required; article description that is displayed in search results. 
author: rolandnyamo #Required; your GitHub user alias, with correct capitalization.
ms.author: ronyamo #Required; microsoft alias of author; optional team alias.
ms.service: azure #Required; use either service or product per approved list. 
ms.topic: overview #Required; leave this attribute/value as-is.
ms.date: 09/17/2025 #Required; mm/dd/yyyy format.
ms.custom: template-overview #Required; leave this attribute/value as-is.
# Customer intent: As a cloud administrator, I want to use impact reporting tools to document performance issues in my Azure workloads so that I can quickly identify and address platform-related problems to maintain service reliability.
---

# What is Azure Impact Reporting Preview?

> [!IMPORTANT]
> Azure Impact Reporting is currently in preview. For legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

To enhance your investigations into observed workload regressions, Azure Impact Reporting provides an Azure platform relevant to the resource and the time of the regression. When you report an issue, we investigate changes, outages, and other platform events in the context of provided resources and provide you with findings.

[![Diagram that shows the architecture of impact connectors for Azure Monitor.](images/impact-reporting-end-to-end.png)](images/impact-reporting-end-to-end.png#lightbox)

## What is an impact?

In this context, an *impact* is any observed regression, unexpected behavior, or issue that negatively affects your workloads, and you suspect the platform is the cause.

Examples of impacts include:

* Unexpected virtual machine reboots.
* Disk IO failures.
* High data-path latency.

## Related content
<!-- Add a context sentence for the following links -->
* [File an impact report](report-impact.md)
<!-- - [View previous impact reports](links-how-to.md) -->
