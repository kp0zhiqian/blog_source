---
title: "Transcript of XDP Tech Talk"
date: 2020-5-21T00:11:23+08:00
draft: false
tags:
    - XDP
keywords:
    - xdp
    - ebpf
---

> This is the transcript of my XDP 5 mins tech talk in QE all hands. Only for sharing and archiving.


XDP is a technology that can achieve high-speed networking performance. Allowing packets to be reflected, filtered, or redirected, but without traversing kernel networking stack.

After hearing this, some may think: Oh, I’ve heard of this! Like the famous DPDK, I can bypass the Kernel and do whatever I want in userspace to the incoming packets. In this case, if we use our network program in the user-space by using DPDK, we’ll call it user-space networking. 

But XDP does the opposite: XDP moves the user-space program into the kernel. and allow us to run our network program or function as soon as the packets flow into the NIC, but before it starts moving up into the kernel network stack. This will lead to a great increase in packets-processing speed. And since XDP doesn’t bypass the kernel network stack, so you won’t need to reimplement other functions that the kernel provided, this is much more convenient in certain scenarios. 

But here’s the question, how does the kernel allow the user to run their programs within the kernel space? Before answering this, we need to take a look at something called BPF.

Simply saying, BPF is an in-kernel virtual machine model, which allows us to write our program and run it in the BPF VM by using the hooks and probe that BPF provided. As you can see from this diagram, if we want to use BPF, the first thing besides writing code is to generate the BPF bytecode which can be compiled from other code like C code. 

Then we need to load the BPF bytecode to BPF VM through the verifier. The verifier will check if there’s something bad in your code that may have an impact on the kernel. You can assume the XDP program is just a BPF program that uses the XDP feature. 

This kind of XDP program is the data-plane of XDP, it can only do whatever you’ve already programmed in the code statically. If we want to control this XDP program dynamically, we also need a userspace program that controls it via the BPF maps. We won’t cover too much about this part in this talk.

From this diagram, you can see the basic workflow. The XDP program will return actions after the traffic hits it. The actions are XDP_DROP, PASS, TX, ABORTED, and REDIRECT. The meaning of these actions is pretty straightforward. After getting the actions from the XDP program, the XDP packet processor will do the action to the packets. 

The XDP I was talking about all along the way is called the “native mode XDP”, which means the nic driver has XDP support and the XDP program will be running before SK buffer is allocated. There are two more modes of loading the XDP program. 

Generic mode and hardware offload mode. Generic mode, means the driver doesn’t need to support XDP, the kernel will implement the xdp feature, the XDP program will run after the SK buffer is allocated. In this case, there’s no performance benefit, but the good thing is you don’t need the driver support to run an XDP program. () As for hardware offload mode, it means the XDP program can load and run directly inside the network card, which will have the best performance, but only a handful of NICs can support this. 