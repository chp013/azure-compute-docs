---
title: 'Azure Impact Reporting: Azure Monitor Alert Connector' #Required; page title is displayed in search results. Include the brand.
description: Learn how to seamlessly report impact through Azure Monitor alerts. #Required; article description that is displayed in search results. 
author: rolandnyamo #Required; your GitHub user alias, with correct capitalization.
ms.author: ronyamo #Required; microsoft alias of author; optional team alias.
ms.service: azure #Required; use either service or product per approved list. 
ms.topic: overview #Required; leave this attribute/value as-is.
ms.date: 09/17/2025 #Required; mm/dd/yyyy format.
ms.custom: template-overview #Required; leave this attribute/value as-is.
# Customer intent: "As a systems administrator, I want to utilize the Impact Connector for Azure Monitor alerts, so that I can efficiently report impact events and enhance change event correlation within my organization's AIOps workflows."
---

# What is the impact connector for Azure Monitor alerts (preview)?

> [!IMPORTANT]
> Azure Impact Reporting is currently in preview. For legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

When you use the impact connector for Azure Monitor alerts, you can seamlessly report the impact from an alert into Microsoft AI operations for change event correlation.

## How connectors work

[![Diagram that shows the architecture of impact connectors for Azure Monitor.](images/azure-monitor-connector.png)](images/azure-monitor-connector.png#lightbox)

When you create a connector, it gets associated with a subscription. When alerts whose target resources reside in the specified subscription are fired, an impact report is created through Azure Impact Reporting and sent to Microsoft intelligence systems.

## Related content

* [Create a connector for Azure Monitor alerts](create-azure-monitor-connector.md)
* [Impact Reporting connectors: Troubleshooting guide](connectors-troubleshooting-guide.md)
* [View reported impacts and insights](view-impact-insights.md)
