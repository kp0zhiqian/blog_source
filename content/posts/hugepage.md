---
title: "Understanding Hugepage"
date: 2020-10-20T00:11:23+08:00
draft: false
tags:
    - systems
keywords:
    - Hugepage
---

## Virtual Memory in Linux

In a multi-task system, the operation to the memory of different processes will have conflict problems. To solve this problem, the concept of virtual memory comes. When the process starting, OS will assign virtual memory addresses for the process to use. Also, OS will map the virtual memory addresses to physical memory addresses. So we need to mapping the table to maintain the relationship between the virtual memory address and the physical memory address.

However, if we 1:1 map the virtual and physical memory address, the table would be too large to search and maintain. To solve this problem, the concept of paging and page table comes. 



## Linux Paging and Page Table

In memory management, the OS will separate the physical memory into many pages (default size 4K). When the OS needs to allocate virtual memory addresses from physical memory addresses, it will assign pages. So that the mapping table between virtual memory address to physical memory address will be much smaller. In this case, we also need a table to maintain the mapping between the virtual pages and the physical pages. We called this table a paging table.



## Hugepages

The OS will use a 4K size to divide memory into many pages by default. However, when the memory size becomes large, the number of pages will also be large, which will cause low efficiency when the CPU searches the memory address page by page. To solve this problem, we can only extend the size of the pages to reduce the number of pages. That's the original concept of hugepages. We can define hugepages as 2M, 4M, 6M, and 1G max.



## Hugepage Terminology

### TLB

Translation Lookside Buffer is a cache in CPU with a certain size to store parts of the paging table to accelerate the mapping between virtual memory and physical memory.



### hugetlb

hugetlb is the TLBs to process the mapping of hugepages.



### hugetlbfs

hugetlbfs is a file system type based on memory. TLB can point to the actual hugepage memory through hugetlb. The hugepages assigned by the OS will be used by the memory as the file in the hugetlbfs.





## Hugepage配置

### Permanent Config

Change `/etc/default/grub` content，add

```shell
default_hugepagesz=1G hugepagesz=1G hugepages=16 hugepagesz=2M hugepages=2048 iommu=pt intel_iommu=on isolcpus=1-13,15-27

## isolcpus depends on the cpu of your hosts
## iommu and isolcpus is actually not related to hugepage
```

after `GRUB_CMDLINE_LINUX_DEFAULT`.

generate grub2 configuration

```shell
grub2-mkconfig -o /boot/grub2/grub.cfg
```

reboot

```shell
reboot
```

Check kernel params in cmdline

```shell
grep Huge /proc/cmdline
```

install hugepage filesystem

```shell
mkdir -p /mnt/hugepage
mount -t hugetlbfs hugetlbfs /mnt/hugepage 
#不指定-o参数的话，挂在的hugepage会按照系统默认的大小挂载
# without -o option, it will use the default size to mount the hugepage
# or
mkdir -p /mnt/hugepage_2M
mount -t hugetlbfs none /mnt/hugepage_2M -o pagesize=2MB
# mount 2M huagepages
```



### Check hugepages

```shell
grep Huge /proc/meminfo
AnonHugePages:         0 kB
HugePages_Total:       0  
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB

mount -l | grep hugetlbfs #check if hugepages successfully mounted
```

## References

[理解linux虚拟内存](https://zhenbianshu.github.io/2018/11/understand_virtual_memory.html)

[linux Hugepages特性](http://www.linuxboy.net/linuxjc/55724.html)

[内存大页基础讲解](http://www.linuxboy.net/linuxjc/33901.html)



