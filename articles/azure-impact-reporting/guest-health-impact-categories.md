---
title: Azure HPC Guest Health Reporting - Impact Categories 
description: View Guest Health Reporting impact categories.
author: rolandnyamo 
ms.author: ronyamo 
ms.service: azure 
ms.topic: overview 
ms.date: 09/18/2025 
ms.custom: template-overview 
---

# Guest Health Reporting impact categories (preview)

> [!IMPORTANT]
> Guest Health Reporting is currently in preview. For legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability, see the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

To properly report issues to Guest Health Reporting, you must use an impact category that starts with `Resource.HPC`.

There are three main types of HPC impact categories:

* `Reset`: Request a refresh of the node health state.
* `Reboot`: Request a node reboot.
* `Unhealthy`: Issues are observed on the node. Node should be taken out of production for further diagnostics and repair.

## Detailed HPC impact categories

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

## Related content

* [What is Guest Health Reporting](guest-health-overview.md)
* [Report Node Health](guest-health-impact-report.md)
