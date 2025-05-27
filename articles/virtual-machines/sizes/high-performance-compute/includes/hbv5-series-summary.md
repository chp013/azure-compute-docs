---
title: HBv5-series summary include file
description: Include file for HBv5-series summary
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 05/27/2025
ms.author: padmalathas
ms.reviewer: mattmcinnes
ms.custom: include file
---

HBv5-series VMs are optimized for the most memory bandwidth-intensive HPC applications. These include workloads such as:
* Running computational fluid dynamics
* Performing automotive simulations
* Conducting aerospace simulations
* Modeling weather systems
* Advancing energy research
* Simulating molecular dynamics
* Supporting computer-aided engineering
* And handling other demanding HPC tasks

HBv5 VMs feature 6.7 TB/s of memory bandwidth across 438 GiB (450 GB) of memory (HBM), and up to 368 4th Generation AMD EPYC™ processor cores with 4 GHz max frequencies and no simultaneous multithreading. HBv5-series VMs also provide 800 Gb/s of InfiniBand connectivity from NVIDIA Networking. This enables supercomputer-scale MPI workloads. In addition, they offer 15 TiB of local NVMe SSD storage, delivering up to 50 GB/s read and 30 GB/s write performance for block devices.


All HBv5-series VMs feature 800 Gbps per node of NDR InfiniBand (2 x 400 Gbps NDR links per node)  from NVIDIA Networking to enable supercomputer-scale MPI workloads. These VMs are connected in a nonblocking fat tree for optimized and consistent RDMA performance. NDR supports features like Adaptive Routing and the Dynamically Connected Transport (DCT), offload of MPI collectives, optimized real-world latencies due to congestion control intelligence, and enhanced adaptive routing capabilities. These features enhance application performance, scalability, and consistency, and their usage is recommended.
