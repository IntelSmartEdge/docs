```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2019-2020 Intel Corporation
```
# SR-IOV Network Operator

- [SR-IOV Network Operator](#sr-iov-network-operator)
  - [Overview](#overview)
    - [Overview of SR-IOV CNI](#overview-of-sr-iov-cni)
    - [Overview of SR-IOV Device Plugin](#overview-of-sr-iov-device-plugin)
    - [Overview of SR-IOV Network Operator](#overview-of-sr-iov-network-operator)
  - [SR-IOV Network Operator configuration and usage](#sr-iov-network-operator-configuration-and-usage)
    - [Configuration](#configuration)
      - [SR-IOV Network Node Policy](#sr-iov-network-node-policy)
      - [SR-IOV Network](#sr-iov-network)
    - [Usage](#usage)
  - [Limitations](#limitations)
  - [Reference](#reference)

## Overview
Edge deployments consist of both network functions and applications. Cloud-native solutions such as Kubernetes* typically expose only one interface to the application or network function pods. These interfaces are typically bridged interfaces. This means that network functions like Base station or Core network User plane functions and applications such as CDN are limited by the default interface. To address this, two key networking features must be enabled:

1. Enable a Kubernetes like orchestration environment to provision more than one interface to the application and network function pods.
2. Enable the allocation of dedicated hardware interfaces to application and network function pods.

### Overview of SR-IOV CNI
The Single Root I/O Virtualization (SR-IOV) feature provides the ability to partition a single physical PCI resource into virtual PCI functions that can be allocated to application and network function pods. The SR-IOV CNI plugin enables the Kubernetes pod to be attached directly to an SR-IOV virtual function (VF) using the standard SR-IOV VF driver in the container host’s kernel.

### Overview of SR-IOV Device Plugin
The Intel SR-IOV Network device plugin discovers and exposes SR-IOV network resources as consumable extended resources in Kubernetes. This works with SR-IOV VFs in both Kernel drivers and DPDK drivers. When a VF is attached with a kernel driver, the SR-IOV CNI plugin can be used to configure this VF in the pod. When using the DPDK driver, a VNF application configures this VF as required.

### Overview of SR-IOV Network Operator
To enable SR-IOV device resource allocation and CNI, Intel® Smart Edge Open uses the SR-IOV Network Operator which wraps SR-IOV CNI and SR-IOV Device Plugin to one simple component. Operator uses custom resources like `SriovNetworkNodePolicy` and `SriovNetwork` to configure SR-IOV plugins and [Multus](./multus.md) CNI.

## SR-IOV Network Operator configuration and usage

To deploy the SR-IOV Network Operator to the Intel® Smart Edge Open cluster, `sriov_network_operator_enable: True` must be set in `inventory/default/group_vars/all/10-default.yml`. This component is enabled by default in [Developer Experience Kit](../developer-experience-kit-open.md). This will perform Operator install and label every Edge Node as `sriov-operator-node=yes` which is required by config Daemon. It is deploying SR-IOV CNI plugin and SR-IOV Device plugin on labeled node when first `SriovNetworkNodePolicy` is applied. It will also automatically install Multus CNI.
SR-IOV Network Operator is deployed using Makefiles and it requires additional packages to be installed, which Intel® Smart Edge Opens provides with Operator deployment.
Images for Operator are downloaded from Openshift repository and stored in local registry.

### Configuration

SR-IOV Network Operator provides two custom resources to configure SR-IOV network devices and network attachments:

#### SR-IOV Network Node Policy
Specifies SR-IOV network device configuration by e.g. creating VFs from given NIC interface/PCI address or its priority and defines Config Map for SR-IOV Device plugin. To apply resource to the Operator just simply use command: `kubectl apply -f sample-policy.yml`.
Sample Policy can look like:
```yaml
apiVersion: sriovnetwork.openshift.io/v1
kind: SriovNetworkNodePolicy
metadata:
  name: sample-netdevice-policy
  namespace: sriov-network-operator
spec:
  nodeSelector:
    feature.node.kubernetes.io/network-sriov.capable: "true"
  resourceName: intel_sriov_netdevice
  numVfs: 4
  nicSelector:
    vendor: "8086"
    deviceID: "159b"
    rootDevices: ["0000:31:00.0"]
```
> **NOTE:** After applying it, Operator will create 4 VFs from given Interface and create Config Map for SR-IOV Device plugin with defined `intel_sriov_netdevice` as allocable resource and mapped from given interface.

> **NOTE:** If `SriovNetworkNodePolicy` should be applied only on specific node, `kubernetes.io/hostname: "<hostname>"` should be added to `nodeSelector` field. 

#### SR-IOV Network
It defines and configures Network attachment for Multus/SR-IOV CNI. To apply resource to the Operator just simply use command: `kubectl apply -f sample-network.yml`.
Sample network can look like:
```yaml
apiVersion: sriovnetwork.openshift.io/v1
kind: SriovNetwork
metadata:
  name: sriov-seo
  namespace: sriov-network-operator
spec:
  resourceName: intel_sriov_netdevice
  networkNamespace: default
  ipam: |-
    {
      "type": "host-local",
      "subnet": "192.168.2.0/24",
      "routes": [{
        "dst": "0.0.0.0/0"
      }],
      "gateway": "192.168.2.1"
    }
```
> **NOTE:** After applying it, Operator will create Network Attachment Definition for Multus/SR-IOV CNI plugin with defined `sriov-seo` network, which applies to `intel_sriov_netdevice` allocable resource in `default` namespace.

### Usage
To create a pod with an attached SR-IOV device, add the network annotation (`sriov-seo`) to the pod definition and request access to the SR-IOV capable device (`intel.com/intel_sriov_netdevice`):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: samplepod
  annotations:
    k8s.v1.cni.cncf.io/networks: sriov-seo
spec:
  containers:
  - name: samplecent
    image: centos/tools
    resources:
      requests:
        intel.com/intel_sriov_netdevice: "1"
      limits:
        intel.com/intel_sriov_netdevice: "1"
    command: ["sleep", "infinity"]
```

To verify that the additional interface was configured, run `ip a` in the deployed pod. The output should look similar to the following:

```shell
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
4: eth0@if3991: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1480 qdisc noqueue state UP group default
    link/ether 36:5a:98:29:a3:8d brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.245.136.180/32 brd 10.245.136.180 scope global eth0
       valid_lft forever preferred_lft forever
1872: net1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether 1a:6d:18:3b:e9:9d brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.2/24 brd 192.168.2.255 scope global net1
       valid_lft forever preferred_lft forever
```
> **NOTE**: Interface `net1` is SR-IOV net device.

## Limitations
It was observed that on Ubuntu 20.04, configuring SR-IOV devices via `SriovNetworkNodePolicy` using `pfNames` on CLV NIC can cause some problems. To avoid this issue it is recommended to use `rootDevices`(see [example above](#sr-iov-network-node-policy)) instead of `pfNames`. To get PCI address of devices `lshw -c network -businfo` command can be used, i.g:
```bash
$ lshw -c network -businfo
Bus info          Device           Class          Description
=============================================================
pci@0000:04:00.0  eno8303          network        NetXtreme BCM5720 2-port Gigabit Ethernet PCIe
pci@0000:04:00.1  eno8403          network        NetXtreme BCM5720 2-port Gigabit Ethernet PCIe
pci@0000:31:00.0  eno12399         network        Intel Corporation
pci@0000:31:00.1  eno12409         network        Intel Corporation
pci@0000:b1:00.0  ens5f0           network        Intel Corporation
pci@0000:b1:00.1  ens5f1           network        Intel Corporation
```
There is also observed that on Ubuntu 20.04 with only one type of SR-IOV NIC which is CLV, there can be lack of label on the node: `feature.node.kubernetes.io/network-sriov.capable: "true"`. To properly apply CLV SR-IOV NIC configuration via `SriovNetworkNodePolicy` using `nodeSelector` use entries like `kubernetes.io/hostname: "<hostname>"` (`<hostname>` should be replaced by node hostname) or `sriov-network-operator-node: "yes"`. Using [example above](#sr-iov-network-node-policy) `feature.node.kubernetes.io/network-sriov.capable: "true"` can be replaced with given examples.

## Reference
For further details:
- Multus: https://github.com/k8snetworkplumbingwg/multus-cni
- SR-IOV CNI: https://github.com/k8snetworkplumbingwg/sriov-cni
- SR-IOV Network Device Plugin: https://github.com/k8snetworkplumbingwg/sriov-network-device-plugin
- SR-IOV Network Operator: https://github.com/k8snetworkplumbingwg/sriov-network-operator
- SR-IOV network node policy [(`SriovNetworkNodePolicy`)](#sr-iov-network-node-policy): https://docs.openshift.com/container-platform/4.7/networking/hardware_networks/configuring-sriov-device.html
- Ethernet network device [(`SriovNetwork`)](#sr-iov-network): https://docs.openshift.com/container-platform/4.7/networking/hardware_networks/configuring-sriov-net-attach.html
