---
title: HBv5-series summary include file
description: Include file for HBv5-series summary
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 08/10/2025
ms.author: padmalathas
ms.reviewer: mattmcinnes
ms.custom: include file
---

HBv5-series VMs are optimized for the most memory bandwidth-intensive HPC applications, including:

* Running computational fluid dynamics
* Running automotive simulations
* Running aerospace simulations
* Running weather system models
* Running energy research workloads
* Running molecular dynamics simulations
* Running computer-aided engineering tasks
* Running other demanding HPC workloads

HBv5 VMs deliver 6.7 TB/s of memory bandwidth across 438 GiB (450 GB) of high-bandwidth memory (HBM) and support up to 368 4th Gen AMD EPYC™ processor cores with up to 4 GHz max frequency, without simultaneous multithreading. 

They also feature 800 Gb/s InfiniBand from NVIDIA Networking, enabling supercomputer-scale MPI workloads. In addition, each VM includes 15 TiB of local NVMe SSD storage, offering up to 50 GB/s read and 30 GB/s write throughput for block devices.

All HBv5-series VMs feature 800 Gbps per node of NDR InfiniBand (2 x 400 Gbps CX-7 NIC per node) of InfiniBand connectivity from NVIDIA Networking to enable supercomputer-scale MPI workloads. These VMs are connected in a nonblocking fat tree for optimized and consistent RDMA performance. 

InfiniBand includes features like Adaptive Routing, Dynamically Connected Transport (DCT), MPI collective offload, and congestion-aware latency optimization. These features improve application performance, scalability, and consistency, and are recommended for use.
