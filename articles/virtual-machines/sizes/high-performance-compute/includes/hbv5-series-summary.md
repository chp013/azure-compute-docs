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

HBv5-series VMs are optimized for the most memory bandwidth-intensive HPC applications, such as computational fluid dynamics, automotive and aerospace simulation, weather modeling, energy research, molecular dynamics, computer aided engineering, and more. HBv5 VMs feature 6.7 TB/s of memory bandwidth across 438 GiB (450 GB) of memory (HBM), and up to 368 4th Generation AMD EPYC™ processor cores with 4 GHz max frequencies and no simultaneous multithreading. HBv5-series VMs also provide 800 Gb/s of InfiniBand from NVIDIA Networking to enable supercomputer-scale MPI workloads, and 15 TiB of local NVMe SSD storage with up to 50 GB/s (reads) and 30 GB/s (writes) of block device performance 

All HBv5-series VMs feature 800Gbps per node of NDR InfiniBand (2 x 400 Gbps NDR links per node)  from NVIDIA Networking to enable supercomputer-scale MPI workloads. These VMs are connected in a non-blocking fat tree for optimized and consistent RDMA performance. NDR supports features like Adaptive Routing and the Dynamically Connected Transport (DCT), offload of MPI collectives, optimized real-world latencies due to congestion control intelligence, and enhanced adaptive routing capabilities. These features enhance application performance, scalability, and consistency, and their usage is recommended.
