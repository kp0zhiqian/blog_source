---
title: "Understanding DMA, IOMMU, and VFIO"
date: 2020-09-04T00:10:57+08:00
draft: false
category: notes
tags:
    - system
keywords:
    - DMA
    - IOMMU
    - VFIO
---
## DMA (Direct Memory Access)

Generally speaking, the read and write operation of memory will all issue by the CPU. The read and write of memory of other devices must issue through the CPU. DMA, as in Direct Memory Access, will release the CPU from such a process. When devices need to r/w the memory, they will report to the CPU. The CPU will give the system bus to the DMA controller. DMA controller will transmit the data between devices and memory. When completing the data transmitting, the CPU will retake the system bus. This process can improve the system performance.

## IOMMU (Input-Output Memory Management Unit)

When we have DMA, devices can directly access the memory address they want, which will be a problem from the security perspective. We should ban such actions. In fact, when transmitting the data, the CPU will tell the DMA controller virtual memory addresses. IOMMU will translate such virtual memory addresses to actual physical memory addresses to keep the physical memory safe. DMA controller can access the memory through the IOMMU. Though we can also access the memory without IOMMU,  it's a bad security practice.

### IOMMU and Virtualization

Before DMA and IOMMU, we used software to simulate hardware to achieve virtualization. With the help of DMA and IOMMU technology, we can expose the physical devices to VM and keep the isolation and security through IOMMU.

## VFIO (Virtual Function I/O)

IOMMU provides the ability to translate virtual memory addresses to physical memory addresses. IOMMU allows physical devices to access memory addresses safely. However, IOMMU only provided a translation ability. How does the user space program use it? Here comes the VFIO. VFIO exposes the IOMMU feature to the user space in a safe way so that the userspace program can develop corresponding drivers to use the IOMMU feature.






