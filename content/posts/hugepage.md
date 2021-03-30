---
title: "理解Hugepage"
date: 2020-10-20T00:11:23+08:00
draft: false
tags:
    - 系统
keywords:
    - Hugepage
---
# Hugepage



## Linux中的虚拟内存

在任务处理中，多个进程对内存的操作会有冲突的问题，为了解决这个问题，则提出了虚拟内存的概念。在进程开始运行时，操作系统会给进程分配虚拟内存地址供进程使用。而操作系统会将这部分虚拟内存地址根据进程的使用情况，来映射到具体的物理内存地址上。而因为有了这个映射关系，则操作系统就需要维护一个映射表。但是，如果将虚拟内存的地址一对一的映射到物理内存上，则需要维护的映射表的大小就太大了，所以为了解决这个问题，则引出了分页和页表的概念。



## Linux中的分页和页表

在Linux的内存管理中，为了节省空间，操作系统会在启动的时候将物理内存以默认4k的大小分割成各个页。而后在为虚拟内存分配物理内存时，都以页为单位。这样一来，虚拟内存到物理内存的这张映射表的大小就会小了很多。而对于某些大内存的情况，操作系统还设计了多级的映射表，这样映射表的大小又进一步的减小。这虚拟内存页到物理内存页的映射表，则被称为页表。



## Hugepages的来源

由上面我们可以知道，操作系统默认情况下会将内存以4k为单位分成不同的页。但是，在内存非常大的情况下，页的数量也会变的特别多，页的数量变大时，CPU检索内存地址的效率也会下降。所以，为了解决这个问题，只有增大页的大小，以此来减少页的数量，提高检索效率。这也就是Hugepages的由来。我们可以将Hugepages定义为2m, 4m, 6m等大小，最大可以定义为1G。



## Hugepage相关术语

### TLB

Translation Lookside Buffer，是CPU中一块固定大小的cache，用来保存一部分页表的对应关系，用来快速进行虚拟内存到物理内存的映射。



### hugetlb

hugetlb指的是 TLB中，一条hugepage的条目。可以说hugetlb是用来处理映射hugepage的。



### hugetlbfs

hugetlbfs是一种机遇内存的文件系统类型，TLB通过hugetlb来指向hugepage，而这些操作系统分配的hugepage则会作为hugetlbfs内存文件系统中的文件来供内存使用。





## Hugepage配置

### 永久配置

修改`/etc/default/grub`的内容，在内核的启动参数`GRUB_CMDLINE_LINUX_DEFAULT`内增加：

```shell
default_hugepagesz=1G hugepagesz=1G hugepages=16 hugepagesz=2M hugepages=2048 iommu=pt intel_iommu=on isolcpus=1-13,15-27

## isolcpus根据本身机器的cpu数量来配置
## iommu和isolcpus用于虚拟化和dpdk隔离cpu上，与hugepage实际上关系不大
```

生成grub配置

```shell
grub2-mkconfig -o /boot/grub2/grub.cfg
```

重启

```shell
reboot
```

重启后查看内核cmdline

```shell
grep Huge /proc/cmdline
```

安装hugepage文件系统

```shell
mkdir -p /mnt/hugepage
mount -t hugetlbfs hugetlbfs /mnt/hugepage 
#不指定-o参数的话，挂在的hugepage会按照系统默认的大小挂载
#或者
mkdir -p /mnt/hugepage_2M
mount -t hugetlbfs none /mnt/hugepage_2M -o pagesize=2MB
#指定挂在了2MB大小的hugepage
```

### 临时配置

TODO



### 检查Hugepage配置

```shell
grep Huge /proc/meminfo
AnonHugePages:         0 kB
HugePages_Total:       0 # Hugepage的总数量
HugePages_Free:        0 # 剩余的Hugepage数量
HugePages_Rsvd:        0 # 保留的Hugepage数量
HugePages_Surp:        0 # 
Hugepagesize:       2048 kB # hugepage的大小

mount -l | grep hugetlbfs #查看hugetlbfs是否挂载成功
```

## 参考

[理解linux虚拟内存](https://zhenbianshu.github.io/2018/11/understand_virtual_memory.html)

[linux Hugepages特性](http://www.linuxboy.net/linuxjc/55724.html)

[内存大页基础讲解](http://www.linuxboy.net/linuxjc/33901.html)



