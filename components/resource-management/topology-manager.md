```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```

# Topology Manager

## Overview

Topology Manager is a Kubelet component that aims to co-ordinate the set of components that are responsible for these optimizations.

## How It Works

Multi-core and Multi-Socket commercial, off-the-shelf (COTS) systems are widely used for the deployment of application and network functions. COTS systems provide a variety of IO and memory features. In order to achieve determinism and high performance, mechanisms like CPU isolation, IO device locality, and socket memory allocation are critical. Cloud-native stacks such as Kubernetes are beginning to leverage resources such as CPU, hugepages, and I/O, but are agnostic to the Non-Uniform Memory Access (NUMA) alignment of these. Non-optimal, topology-aware NUMA resource allocation can severely impact the performance of latency-sensitive workloads. Topology Manager addresses the requirement.

Let us look at an Analytics application that is consuming multiple, high-definition video streams and executing an analytics algorithm. This analytics application pod is compute-, memory-, and network performance-sensitive. To improve performance of the analytics application on a typical dual-socket or multi-NUMA node system, an Orchestrator like Kubernetes needs to place the analytics pod on the same NUMA node where the Network Card is located and the memory is allocated. Without the topology manager, the deployment may be like that shown in the diagram below, where the analytics application is on NUMA 1 and the Device is on NUMA 2. This leads to poor and unreliable performance.

![Pod deployment issue without Topology Manager](tm-images/tm1.png)
_Figure - Pod deployment issue without Topology Manager_

With Topology manager, this issue is addressed and the analytics pod placement will be such that the resource locality is maintained.

![Pod deployment with Topology Manager](tm-images/tm2.png)
_Figure - Pod deployment with Topology Manager_

## How To

### Enable Topology Manager

Set policy to `best-effort` in the ESP provisioning configuration file (before Smart Edge Open deployment):

- generate a custom configuration file with `./dek_provision.py --init-config > custom.yml`
- edit generated file and set `topology_manager: policy` under `group vars: all:`, e.g.

```yaml
profiles:
  - name: SEO_DEK
    [...]
    group_vars:
      groups:
        all:
          topology_manager:
            policy: "best-effort"
```

Available values of Kubernetes Topology Manager policy: `none` (disabled), `best-effort` (default), `restricted`, `single-numa-node`. Refer to the [Kubernetes Documentation](https://kubernetes.io/docs/tasks/administer-cluster/topology-manager/) for details of these policies.

User can also set `reserved_cpus` to a number that suits best. This parameter specifies the logical CPUs that will be reserved for a Kubernetes system Pods and OS daemons.

```yaml
profiles:
  - name: SEO_DEK
    [...]
    group_vars:
      groups:
        all:
          reserved_cpus: "0,1"
```
- use the custom configuration for all the following `dek_provision.py` command invocations, i.e. `./dek_provision.py --config=custom.yml [...]`

### Use Topology Manager

Create a Pod with a `guaranteed` QoS class (requests equal to limits). For example:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: examplePod
spec:
  containers:
    - name: example
      image: alpine
      command: ["/bin/sh", "-ec", "while :; do echo '.'; sleep 5 ; done"]
      resources:
        limits:
          cpu: "8"
          memory: "500Mi"
        requests:
          cpu: "8"
          memory: "500Mi"
```

Then apply it with `kubectl apply`. Check in the kubelet's logs on the node (`journalctl -xeu kubelet`), that Topology Manager obtained all the info about preferred affinity and deployed the Pod accordingly. The logs should be similar to the one below.

```
Nov 05 09:22:52 tmanager kubelet[64340]: I1105 09:22:52.548692   64340 topology_manager.go:308] [topologymanager] Topology Admit Handler
Nov 05 09:22:52 tmanager kubelet[64340]: I1105 09:22:52.550016   64340 topology_manager.go:317] [topologymanager] Pod QoS Level: Guaranteed
Nov 05 09:22:52 tmanager kubelet[64340]: I1105 09:22:52.550171   64340 topology_hints.go:60] [cpumanager] TopologyHints generated for pod 'examplePod', container 'example': [{0000000000000000000000000000000000000000000000000000000000000001 true} {0000000000000000000000000000000000000000000000000000000000000010 true} {0000000000000000000000000000000000000000000000000000000000000011 false}]
Nov 05 09:22:52 tmanager kubelet[64340]: I1105 09:22:52.550204   64340 topology_manager.go:285] [topologymanager] ContainerTopologyHint: {0000000000000000000000000000000000000000000000000000000000000010 true}
Nov 05 09:22:52 tmanager kubelet[64340]: I1105 09:22:52.550216   64340 topology_manager.go:329] [topologymanager] Topology Affinity for Pod: 4ad6fb37-509d-4ea6-845c-875ce41049f9 are map[example:{0000000000000000000000000000000000000000000000000000000000000010 true}]

```
