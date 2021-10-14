---
title: "Container, Kubernetes and SR-IOV"
date: 2021-10-14T00:10:57+08:00
draft: false
category: notes
tags:
    - kubernetes
keywords:
    - ansible
    - automation
    - system
---

> This article is the transcript of a tech sharing in my team and assumed the audiences have already knew some background knowladge.

## Container Basic 

> "Container is NOT virtual machine!"
> 

![SR-IOV%20Container%20Test%20f0bdf8e02a57412cbecc70845fe60bb0/Untitled.png](../../post-image/k8s-container/Untitled.png)

Linux provides namespace and cgroup to isolate or limit resources between different process.

Container is a technology that uses namespace and cgroup (also the union file system) to create an isolated environment that can be used by a process.

## Container Runtime

Namespace and cgroup are the mechanisms provided by Linux. The container is only a tech that uses these machines. Container Runtime is the software that actually uses the mechanisms to achieve container tech. However, different people may have different ideas about the way to use the mechanisms. So comes to the two standards, OCI (Open Container Initiative) and CRI (Container Runtime Interface)

### OCI - Open Container Initiative

![SR-IOV%20Container%20Test%20f0bdf8e02a57412cbecc70845fe60bb0/Untitled%201.png](../../post-image/k8s-container/Untitled%201.png)

Docker leads the OCI. OCI defines the way to run a container, how to set up the namespace and cgroup, also includes the format of a container image, configuration, and metadata. 

runC is a full implementation of OCI on Linux.

[Open Container Initiative](https://opencontainers.org/)

[opencontainers/runc](https://github.com/opencontainers/runc)

### CRI - Container Runtime Interface

Container Runtime Interface is a container runtime standard published by Kubernetes. Kubernetes is a platform that manages and orchestrates large-scale containers. This platform combines many components, container run time is one of them. Any runtime software that achieves the CRI can be used by Kubernetes as a container runtime component. Besides CRI, Kubernetes also have standards like CNI(Container Network Interface) and CSI (Container Storage Interface)

OCI is a standard that defines how to create/run/manage a container. CRI is a standard that defines how runtime software can be used by Kubernetes.

[kubernetes/kubernetes](https://github.com/kubernetes/kubernetes/blob/242a97307b34076d5d8f5bbeb154fa4d97c9ef1d/docs/devel/container-runtime-interface.md)

[Introducing Container Runtime Interface (CRI) in Kubernetes](https://kubernetes.io/blog/2016/12/container-runtime-interface-cri-in-kubernetes/)

## Docker

Docker is a container management tool that has features like build/pull/run container. After installing Docker, you will get at least 3 components: runC, containerd, dockerd, also include a docker cli client. The relationship between the 3 main components can be described as below chart.

![SR-IOV%20Container%20Test%20f0bdf8e02a57412cbecc70845fe60bb0/Untitled%202.png](../../post-image/k8s-container/Untitled%202.png)

1. Docker CLI client send commands to the Dockerd who is the daemon of Docker.
2. Dockerd then call containerd to create container.
3. Containerd will fork a child process containerd-shim
4. containerd-shim will use runC to create the target container.

![SR-IOV%20Container%20Test%20f0bdf8e02a57412cbecc70845fe60bb0/Untitled%203.png](../../post-image/k8s-container/Untitled%203.png)

## Podman

Podman is also a container management tool. But unlike Docker needs a daemon running at the backend, Podman directly creates and manages the container. Podman also uses runC, which means it implements OCI standards.

![SR-IOV%20Container%20Test%20f0bdf8e02a57412cbecc70845fe60bb0/Untitled%204.png](../../post-image/k8s-container/Untitled%204.png)

Podman and Docker are the same when you using the cli

## Cri-o & Crictl

cri-o is a container runtime published by Redhat, implementing both OCI and CRI standards. It uses OCI to communicate with runC, CRI to communicate with Kubernetes. The purpose of cri-o is not a tool like Docker or Podman but a runtime component that can be used by Kubernetes. For this reason, cri-o is actually only running as a daemon. cri-o works like a component of Kubernetes, so it has no cli interface, which means we cannot use cri-o directly from cli. But there's also an open-source tool called crictl. crictl works as a cri-o client, but only has limited features. crictl is usually used by k8s developers as a dev tool. Cri-o achieves CRI to the north Kubernetes, achieves OCI to the south by runC.

![SR-IOV%20Container%20Test%20f0bdf8e02a57412cbecc70845fe60bb0/Untitled%205.png](../../post-image/k8s-container/Untitled%205.png)

Openshift is just a platform developed based on Kubernetes with many more advanced features. From the perspective of the low-level implementation, Openshift and Kubernetes is basically the same. It uses cri-o as it's container runtime, which is why we use cri-o and crictl in our SR-IOV container testing.

## CNI

Similar with CRI, CNI is also a standard published by Kubernetes. Instead of defining how to be a container runtime component, CNI defines how to config and use the container network. CNI is made up by some network api standard and library. CNI only focus on configure the network resource when creating the container and releasing the network resource when deleting the container. 

Currently, CNI has already Kubernetes built-in component. 

## CNI Plugin

CNI plugin must be an executable file in a certain path. This file will be called by Kubernetes to insert the network port to the container namespace (like one point of a veth pair) and also do necessary change on the host (like connect the other point of a veth pair to bridge), and then assign IP to the port and route. 

### SR-IOV CNI Plugin

[openshift/sriov-cni](https://github.com/openshift/sriov-cni)

🕹️ Example with SR-IOV CNI Plugin

```bash
# Complie the excutable
git clone https://github.com/openshift/sriov-cni.git
pushd sriov-cni
sudo yum install -yq go
make build  # build sriov-cni binary

# Move excutable to certain path
cp -f build/sriov /usr/libexec/cni/

# Create configuation
{ 
    "cniVersion":"0.3.1",
    "name":"sriov-net",
    "type":"sriov",
    "vlan":0,
    "spoofchk":"off",
    "vlanQoS":0,
    "ipam": {
      "type":"host-local",
      "subnet":"192.168.111.0/24",
      "rangeStart":"192.168.111.${START_IP}",
      "rangeEnd":"192.168.111.${END_IP}",
      "routes":[{"dst":"0.0.0.0/0"}],
      "gateway":"192.168.111.254"
    },
    "deviceID": "${VF_PCI_ID}"
}

# Place the configuration file to /etc/cni/net.d/
```

## How does the Openshift create SR-IOV network?

### Openshift/Kubernetes Basic Concepts

Control plane: Where the brain of Kubernetes is, make decision, schedule container/jobs

Worker plane(node): Where the actual container is running, worker node use kubelet to communicate with control node.

The concept of control node and worker node is similar to the control plane and data plane in SDN.

![SR-IOV%20Container%20Test%20f0bdf8e02a57412cbecc70845fe60bb0/Untitled%206.png](../../post-image/k8s-container/Untitled%206.png)

### How to operate Openshift/Kubernetes

The way to config such container platform is to describe the state of the cluster, like the way we use in Ansible. And submit this yaml file to the k8s backend.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

In a yaml file like this, we will define what our application needs. So if our application needs SR-IOV function, we will also define it in such yaml file. 

🎯After submitting such yaml file to the openshift control plane, openshift will find the worker node who has the SR-IOV function, and schedule the pod to it. Then CNI plugin will do its work.

![SR-IOV%20Container%20Test%20f0bdf8e02a57412cbecc70845fe60bb0/Untitled%207.png](../../post-image/k8s-container/Untitled%207.png)

### ⁉️How does the openshift know which worker node has the SR-IOV function?

Openshift has a thing called operator, it's actually a customized controller. SR-IOV has its SR-IOV operator. This operator will run a deamonSet on every worker node to init and register the SR-IOV  function to the control node. Then when a Pod needs SR-IOV function, the controller will know where to schedule it. After the Pod is scheduled to this worker node, the operator will call the CNI to setup the container's network. 

CNI won't config the actually network resource like SR-IOV, it just move the network resource into the container. Operator will create the corresponding network function.

[About Single Root I/O Virtualization (SR-IOV) hardware networks - Hardware networks | Networking | OpenShift Container Platform 4.7](https://docs.openshift.com/container-platform/4.7/networking/hardware_networks/about-sriov.html)

So the process will be like:

1. Create SR-IOV network operator
2. Operator create deamonSet on worker node.
3. daemonSet and operator initiate the SR-IOV function and report to the openshift control node
4. openshift receive a request to create a Pod with SR-IOV VF
5. openshift pick up SR-IOV function worker node from its database, and schedule the Pod to the certain worker node
6. Worker node call SR-IOV CNI Plugin to configure container network and IP/route

### Script to create and run a container that use SR-IOV

[kp0zhiqian/crio-sriov-example](https://github.com/kp0zhiqian/crio-sriov-example)

### Steps for using SR-IOV in container against a RHEL baremetal simulation enviroment.

1. Install crio, crictl
2. pull container image
3. install CNI plugin
4. create CNI configuration
5. create Pod configuration
6. create Pod
7. create container, put it into pod
8. verify the container is using sriov vf
9. send some traffic to check more.

## MORE: What is Pod and why we use it.

In a system like Kubernetes or Openshift, Pod is the thing that we normally deal with, not container. Pod is a group of container that share one network, storage and hostname. It's just like a logical host. Containers in a Pod are like processes on this host.

### Why use Pod?

- In production, we normally have the env that different process who depend on each other. The recommendation is put process in its own container. So in this case, it will need a way to combine a group of contaienr — Pod
- If one of the containers from a service group die, how will we define the state of other container in the same group?
- The containers in a Pod share the same network namespace and storage, which will be more convenient for the same service.

### Pod is also a container? 

The way to implement Pod is to create a container and run "pause" command inside the container. All the service container of the pod will inherit the namespace of "pause" container. When we create and delete a pod, we actually create and delete a pause container.