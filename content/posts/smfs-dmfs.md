---
title: "SMFS and DMFS in Hardware Offloading"
date: 2022-01-10T00:11:23+08:00
draft: false
tags:
    - networking
    - openvswitch
keywords:
    - network
---

# SMFS and DMFS in Hardware Offloading

## switchdev (ethernet switch device driver model)

[kernel.org - switchdev](https://www.kernel.org/doc/Documentation/networking/switchdev.txt)

> The Ethernet switch device driver model (switchdev) is an in-kernel driver
model for switch devices which offload the forwarding (data) plane from the
kernel.

### Configuration
1. Use "depends NET_SWITCHDEV" in driver's Kconfig to ensure switchdev model
support is built for driver.
2. use `devlink` tool to setup switch mode
`devlink dev eswitch set pci/$PF_PCI mode switchdev`

## switchdev Performance Tuning
[OVS Offload Using ASAP2 Direct](https://docs.nvidia.com/networking/pages/viewpage.action?pageId=39285091#OVSOffloadUsingASAP%C2%B2Direct-ovshwoffloadsconfigOVSHardwareOffloadsConfiguration)

SwitchDev proformance can be furtuer improved by tuning it.

### Steering Mode
OVS-kernel supports two steering modes for rules insertion into hardware.

1.  **SMFS** – Software Managed Flow Steering (as of MLNX_OFED v5.1, this is the default mode)  
    Rules are inserted directly to the hardrware by the software (driver). This mode is optimized for rules insertion.
2.  **DMFS** – Device Managed Flow Steering  
    Rules insertion is done using firmware commands. This mode is optimized for throughput with a small amount of rules in the system.  
### Configuration
The mode can be controlled via sysfs or devlink API in kernels that support it:
```
Sysfs:

# echo smfs > /sys/class/net/<PF netdev>/compat/devlink/steering_mode

Devlink:

# devlink dev param set pci/0000:00:08.0 name flow_steering_mode value "smfs" cmode runtime

Replace smfs param with dmfs for device managed flow steering
```

### Notes
- SMFS and DMFS mode should be set before moving the device to "switchdev" mode.
- Only when moving to SwitchDev will the driver use the mode set by the previous step.
- Steering mode cannot be changed after moving to "switchdev" mode
- Steering mode can only be applicable for "switchdev" mode only, which means no effects to legacy SR-IOV  or other configurations.