```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2022 Intel Corporation
```

# Nodus

## Overview

Nodus has been developed as one of [Akraino](https://www.lfedge.org/projects/akraino/) projects to provide Network Controller functionality and support wide range of Kubernetes networking use cases. 
It consists of four major components: 
- OVN control plane
- OVN controller
- Network Function Network(NFN) operator that runs in K8s control plane
- Network Function Network (NFN) agent for K8s nodes

Nodus also provides a CNI plugin based on OVN and OpenVSwitch (OVS). It works with Multus CNI to provide pods with multiple interfaces.

One of the important features of Nodus is the ability to create virtual LAN networks on pod's interfaces at runtime. The CNI plugin also utilises physical interfaces to connect a pod to an external network (WAN) called a "provider network". This functionality is particularly important for SD-WAN CNFs used in the Intel® Smart Edge Open Secure Access Service Edge (SASE) experience kit. The CNF pods act as a proxy between the virtual LANs in the SASE Edge cluster and the WAN. Nodus is enabled by default in the SASE experience kit.
To read about Nodus components and features go to [icn-nodus](https://github.com/akraino-edge-stack/icn-nodus).

## How It Works

Typically, in Kubernetes, each pod has only one network interface (apart from a loopback). With Multus, users can create a multi-homed pod that has multiple interfaces. To accomplish this, Multus acts as a “meta-plugin”, a CNI plugin that can call multiple other CNI plugins to add multiple interfaces to a pod. In the Intel® Smart Edge Open SASE experience kit, Nodus CNI is enabled by default as the secondary CNI whereas Calico act as the primary CNI. In such scenarios where Multus is used, net1 interface is by convention the OVN default interface that connects to Multus. The other interfaces (net2, net3, ...) are added by Nodus according to the pod annotation. 

In a scenario where a CNF pod becomes a proxy between a virtual LAN in the Edge cluster and the WAN, it needs to have two types of interfaces configured:

- A virtual LAN network is configured and attached to one of the pod's virtual interfaces. This network connects application pods belonging to the same OVN network in the cluster. Nodus plugin allows for simplified creation of a virtual OVN network based on the provided configuration.
- A provider network is configured to connect the pod to an external network (WAN). The provider network must be attached to the physical network infrastructure via layer-2 (i.e., via bridging/switching).

To learn about other supported scenarios and see examples of usage, go to [Nodus Usage Guide](https://github.com/akraino-edge-stack/icn-nodus/blob/master/doc/how-to-use.md).

## How To

### Create a virtual LAN network and a provider network

The following examples show sample definitions of a virtual LAN network and provider networks.

1. Vrtual LAN network
```yaml
  apiVersion: k8s.plugin.opnfv.org/v1alpha1
  kind: Network
  metadata:
    name: ovn-port-net
  spec:
    cniType : ovn4nfv
    ipv4Subnets:
    - subnet: 172.16.33.0/24
      name: subnet1
      gateway: 172.16.33.1/24
```
( Source: [https://github.com/akraino-edge-stack/icn-nodus/blob/master/example/ovn-port-net.yaml](https://github.com/akraino-edge-stack/icn-nodus/blob/master/example/ovn-port-net.yaml) )

2. Provider network of type 'direct'
```yaml
  apiVersion: k8s.plugin.opnfv.org/v1alpha1
  kind: ProviderNetwork
  metadata:
    name: directpnetwork
  spec:
    cniType: ovn4nfv
    ipv4Subnets:
    - subnet: 172.16.34.0/24
      name: subnet2
      gateway: 172.16.34.1/24
      excludeIps: 172.16.34.2 172.16.34.5..172.16.34.10
    providerNetType: DIRECT
    direct:
      providerInterfaceName: eth0.101
      directNodeSelector: specific
      nodeLabelList:
      - kubernetes.io/hostname=ubuntu18
```
( Source: [https://github.com/akraino-edge-stack/icn-nodus/blob/master/example/ovn4nfv_direct_pn.yml](https://github.com/akraino-edge-stack/icn-nodus/blob/master/example/ovn4nfv_direct_pn.yml) )

3. Provider network of type 'VLAN'
```yaml
  apiVersion: k8s.plugin.opnfv.org/v1alpha1
  kind: ProviderNetwork
  metadata:
    name: vlanpnetwork
  spec:
    cniType: ovn4nfv
    ipv4Subnets:
    - subnet: 172.16.34.0/24
      name: subnet1
      gateway: 172.16.34.1/24
      excludeIps: 172.16.34.2 172.16.34.5..172.16.34.10
    providerNetType: VLAN
    vlan:
      vlanId: "100"
      providerInterfaceName: eth0
      logicalInterfaceName: eth0.100
      vlanNodeSelector: specific
      nodeLabelList:
      - kubernetes.io/hostname=ubuntu18
```
( Source: [https://github.com/akraino-edge-stack/icn-nodus/blob/master/example/ovn4nfv_vlan_pn.ym](https://github.com/akraino-edge-stack/icn-nodus/blob/master/example/ovn4nfv_vlan_pn.yml) )

To list defined networks, use:

```Shell.bash
  kubectl get networks
```

### Create pods attached to a virtual network

Add an annotation to your pod definition in order to connect the pod replicas to a virtual network ('ovn-port-net') 

```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nodus-deployment-vn
    labels:
      app: nodus-vn
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: nodus-vn
    template:
      metadata:
        labels:
          app: nodus-vn
        annotations:
          k8s.v1.cni.cncf.io/networks: '[
              { "name": "ovn4nfv-k8s-plugin",
                "interface": "net1"
              }]'
          k8s.plugin.opnfv.org/nfn-network: '{ "type": "ovn4nfv", "interface": [{ "name": "ovn-port-net", "interface": "net2" , "defaultGateway": "false"}'
      spec:
        containers:
        - name: nodus-deployment-vn
          image: "busybox"
          command: ["top"]
          stdin: true
          tty: true
```
( Source: [https://github.com/akraino-edge-stack/icn-nodus/blob/master/example/ovn4nfv-deployment-replica-2-with-multus-ovn4nfv-annotations.yam](https://github.com/akraino-edge-stack/icn-nodus/blob/master/example/ovn4nfv-deployment-replica-2-with-multus-ovn4nfv-annotations.yaml) )

#### Verify that the additional interfaces are configured on the pods

Run `ifconfig` in the deployed pod. The output should look similar to the following:

```Shell.bash
  eth0      Link encap:Ethernet  HWaddr B6:66:62:E9:40:0F
            inet addr:10.233.64.14  Bcast:10.233.127.255  Mask:255.255.192.0
            UP BROADCAST RUNNING MULTICAST  MTU:1400  Metric:1
            RX packets:13 errors:0 dropped:0 overruns:0 frame:0
            TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:0
            RX bytes:1026 (1.0 KiB)  TX bytes:0 (0.0 B)

  lo        Link encap:Local Loopback
            inet addr:127.0.0.1  Mask:255.0.0.0
            UP LOOPBACK RUNNING  MTU:65536  Metric:1
            RX packets:0 errors:0 dropped:0 overruns:0 frame:0
            TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:1000
            RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

  net1      Link encap:Ethernet  HWaddr B6:66:62:10:D2:00:1F
            inet addr:10.210.0.30  Bcast:10.210.255.255  Mask:255.255.255.0
            UP BROADCAST RUNNING MULTICAST  MTU:1400  Metric:1
            RX packets:13 errors:0 dropped:0 overruns:0 frame:0
            TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:0
            RX bytes:1026 (1.0 KiB)  TX bytes:0 (0.0 B)

  net2      Link encap:Ethernet  HWaddr B6:66:62:10:21:03
            inet addr:172.16.33.2  Bcast:172.16.33.255  Mask:255.255.255.0
            UP BROADCAST RUNNING MULTICAST  MTU:1400  Metric:1
            RX packets:13 errors:0 dropped:0 overruns:0 frame:0
            TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:0
            RX bytes:1026 (1.0 KiB)  TX bytes:0 (0.0 B)
```
#### Test the network connectivity between the two pods:

Run `ping` command between the two pods specifying the interface connected to the virtual network ('ovn-port-net')

```Shell.bash
  # kubectl exec -it nodus-deployment-84f68d5474-6tsk7 -- ping 172.16.33.2
  PING 172.16.44.2 (172.16.33.4): 56 data bytes
  64 bytes from 172.16.33.2: seq=0 ttl=64 time=0.071 ms
  64 bytes from 172.16.33.2: seq=1 ttl=64 time=0.090 ms
  64 bytes from 172.16.33.2: seq=2 ttl=64 time=0.084 ms
  64 bytes from 172.16.33.2: seq=3 ttl=64 time=0.090 ms
  ...
```

### Create a pod attached to a provider network ('VLAN' or 'direct')
Add an annotation to your pod definition in order to connect the pod to a provider network ('vlanpnetwork')

```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nodus-deploymen-pn
    labels:
      app: nodus-pn
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: nodus-pn
    template:
      metadata:
        labels:
          app: nodus-pn
        annotations:
          k8s.v1.cni.cncf.io/networks: '[{ "name": "ovn-networkobj"}]'
          k8s.plugin.opnfv.org/nfn-network: '{ "type": "ovn4nfv", "interface": [{ "name": "vlanpnetwork", "interface": "net0" }]}'
      spec:
        containers:
        - name: nodus-deployment-pn
          image: "busybox"
          imagePullPolicy: Always
          stdin: true
          tty: true
          securityContext:
            privileged: true
```
( Source: [https://github.com/akraino-edge-stack/icn-nodus/blob/master/example/ovn4nfv_vlan_pn.yml](https://github.com/akraino-edge-stack/icn-nodus/blob/master/example/ovn4nfv_vlan_pn.yml) )

## Reference
For further details on Nodus and more examples on usage go to:
- Nodus: https://github.com/akraino-edge-stack/icn-nodus
- Nodus Usage Guide: https://github.com/akraino-edge-stack/icn-nodus/blob/master/doc/how-to-use.md
