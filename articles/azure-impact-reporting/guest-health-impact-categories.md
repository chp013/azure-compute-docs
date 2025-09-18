---
title: Azure HPC Guest Health Reporting - Impact Categories #Required; page title is displayed in search results. Include the brand.
description: View GHR impact categories #Required; article description that is displayed in search results. 
author: rolandnyamo #Required; your GitHub user alias, with correct capitalization.
ms.author: ronyamo #Required; microsoft alias of author; optional team alias.
ms.service: azure #Required; use either service or product per approved list. 
ms.topic: overview #Required; leave this attribute/value as-is.
ms.date: 09/18/2025 #Required; mm/dd/yyyy format.
ms.custom: template-overview #Required; leave this attribute/value as-is.
# Customer intent: As a cloud administrator, I want to utilize impact reporting tools to document performance issues in my Azure workloads, so that I can quickly identify and address platform-related problems to maintain service reliability.
---

# Guest Health Reporting Impact Categories (Preview)
> [!IMPORTANT]
> Guest Health Reporting is currently in Preview. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

To properly report issues to Guest Health Reporting, you must use an impact category that starts with `Resource.HPC`.

There are three main types of HPC impact categories:
1. `Reset`: Request a refresh of the node health state.
2. `Reboot`: Request a node reboot.
3. `Unhealthy`: Issues are observed on the node. Node should be taken out of production for further diagnostics and repair.

## Detailed HPC Impact Categories

| Category                                          | Description                                   | Mark OFR |
|--------------------------------------------------|-----------------------------------------------|----------|
| Resource.Hpc.Reset                               | Reset node health status                      | No       |
| Resource.Hpc.Reboot                              | Reboot node                                   | No       |
| Resource.Hpc.Unhealthy.HpcMissingGpu             | Missing GPU                                   | Yes      |
| Resource.Hpc.Unhealthy.MissingIB                 | Missing InfiniBand port                       | Yes      |
| Resource.Hpc.Unhealthy.IBPerformance             | Degraded InfiniBand performance               | Yes      |
| Resource.Hpc.Unhealthy.IBPortDown                | InfiniBand port is in DOWN state              | Yes      |
| Resource.Hpc.Unhealthy.IBPortFlapping            | InfiniBand port flapping                      | Yes      |
| Resource.Hpc.Unhealthy.HpcGpuDcgmDiagFailure     | GPU DCGMI diagnostic failure                  | Yes      |
| Resource.Hpc.Unhealthy.HpcRowRemapFailure        | GPU row remap failure                         | Yes      |
| Resource.Hpc.Unhealthy.HpcInforomCorruption      | GPU infoROM corruption                        | Yes      |
| Resource.Hpc.Unhealthy.HpcGenericFailure         | Issue doesn't fall into any other category   | Yes      |
| Resource.Hpc.Unhealthy.ManualInvestigation       | Request further manual investigation by HPC team | Yes   |
| Resource.Hpc.Unhealthy.XID95UncontainedECCError  | GPU uncontained ECC error (Xid 95)            | Yes      |
| Resource.Hpc.Unhealthy.XID94ContainedECCError    | GPU contained ECC error (Xid 94)              | Yes      |
| Resource.Hpc.Unhealthy.XID79FallenOffBus         | GPU fallen off PCIe bus (Xid 79)              | Yes      |
| Resource.Hpc.Unhealthy.XID48DoubleBitECC         | GPU reports double bit ECC error (Xid 48)     | Yes      |
| Resource.Hpc.Unhealthy.UnhealthyGPUNvidiasmi     | nvidia-smi hangs and might not recover        | Yes      |
| Resource.Hpc.Unhealthy.NvLink                    | NvLink is down                                | Yes      |
| Resource.Hpc.Unhealthy.HpcDcgmiThermalReport     | DCGMI reports thermal violations              | Yes      |
| Resource.Hpc.Unhealthy.ECCPageRetirementTableFull| Double-bit ECC error page retirements over threshold | Yes |
| Resource.Hpc.Unhealthy.DBEOverLimit              | GPU has more than 10 double-bit ECC retired pages in seven days | Yes |
| Resource.Hpc.Unhealthy.GpuXIDError               | GPU reports Xid error other than 48,79,94,95  | Yes      |
| Resource.Hpc.Unhealthy.AmdGpuResetFailed         | AMD GPU unrecoverable reset failure error     | Yes      |
| Resource.Hpc.Unhealthy.EROTFailure               | GPU memory EROT failure                       | Yes      |
| Resource.Hpc.Unhealthy.GPUMemoryBWFailure        | GPU memory bandwidth failure                  | Yes      |
| Resource.Hpc.Unhealthy.CPUPerformance            | CPU performance issue                         | Yes      |

## Next steps
<!-- Add a context sentence for the following links -->
* [What is Guest Health Reporting](guest-health-overview.md)
* [Report Node Health](guest-health-impact-report.md)
<!-- - [View previous impact reports](links-how-to.md) -->
