```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```

# CPU Manager

## Overview

CPU Manager is a Kubernetes feature that enables better placement of workloads in the Kubelet, the Kubernetes node agent, by allocating exclusive CPUs to certain pod containers.

## How It Works

A Kubernetes cluster can be divided into namespaces. If a container is created in a namespace that has a default CPU limit, and the container does not specify its own CPU limit, then the container is assigned the default CPU limit
Kubernetes keeps many aspects of how pods execute on nodes abstracted from the user. This is by design. However, some workloads require stronger guarantees in terms of latency and/or performance in order to operate acceptably. The kubelet provides methods to enable more complex workload placement policies while keeping the abstraction free from explicit placement directives.

Default kubelet configuration uses [CFS quota](https://en.wikipedia.org/wiki/Completely_Fair_Scheduler) to manage PODs execution times and enforce imposed CPU limits. For such a solution it is possible that individual PODs are moved between different CPU because of changing circumistances on Kubernetes node. When cetrains pods end itheir lifespan or CPU throttling comes in place, then a pod can be moved to another CPU.

Another solution, default for Intel® Smart Edge Open, supported by Kubernetes is CPU manager. CPU manager uses [Linux CPUSET](https://www.kernel.org/doc/Documentation/cgroup-v1/cpusets.txt) mechanism to schedule PODS to invividual CPUs. Kubernetes defines shared pool of CPUs which initially contains all the system CPUs without CPUs reverved for system and kubelet itself. CPU selection is configurable with kubelet options. Kubernetes uses shared CPU pool to schedule PODs with three QoS classes `BestEffort`, `Burstable` and `Guaranteed`.
When a pod is qualified as `Guaranteed` QoS class then kubelet removes requested CPUs amount from shared pool and assigns the pod exclusively to the CPUs.

## How To

### Configure the CPU manager before cluster deployment

Kubernetes CPU Management needs CPU manager policy to be set to `static` which is a default option in Intel® Smart Edge Open. This can be adjusted in the ESP provisioning configuration file, before deploying an experience kit. Amount of CPUs reserved for Kubernetes and operating system is defined in the same file:

- generate a custom configuration file with `./dek_provision.py --init-config > custom.yml`
- edit generated file and set `policy` or `reserved_cpus` under `group vars: all:`, e.g.

```yaml
profiles:
  - name: SEO_DEK
    [...]
    group_vars:
      groups:
        all:
          policy: "static"
          reserved_cpus: "0,1"
```
- use the custom configuration for all the following `dek_provision.py` command invocations, i.e. `./dek_provision.py --config=custom.yml [...]`

### Create pod definitions

The Kubernetes CPU Manager defines three quality of service classes for pods:
- `BestEffort` QoS class is assigned to pods which do not define any memory and CPU limits and requests. pods from this QoC class run in the shared pool
- `Burstable` QoS class is assigned to pods which define memory or CPU limits and requests which do not match. Pods from `Bustrable` QoS class run in the shared pool.
- `Guaranteed` QoS class is assigned to pods which define memory and CPU limits and requests and those two values are equal. The values set to CPU limits and request have to be integral, fractional CPU specified caused the pod to be run on the shared pool.
  
**Pod defined without any constraints.** This will be assigned `BestEffort` QoS class and will run on shared poll.

```yaml
spec:
  containers:
  - name: nginx
    image: nginx
```

**Pod defined with some constraints.** This will be assigned `Burstable` QoS class and will run on shared poll.

```yaml
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      limits:
        memory: "200Mi"
        cpu: "2"
      requests:
        memory: "100Mi"
        cpu: "1"
```

**Pod defined with constraints, limits are equal to requests and CPU is integral bigger than or equal to one.** This will be assigned `Guaranteed` QoS classs and will run exclusively on CPUs assigned by Kubernetes.

```yaml
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      limits:
        memory: "200Mi"
        cpu: "2"
      requests:
        memory: "200Mi"
        cpu: "2"
```

Pod defined with constraints even when limits are equal to request but CPU is specified as a fractional number will not get exclusive CPUs but will be run on the shared pool. Still, QoS class for such a pod is `Guaranteed`.

**Example POD using CPU Manager feature.**

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: test-pod
  name: test-pod
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: "IfNotPresent"
    resources:
      limits:
        cpu: 1
        memory: "200Mi"
      requests:
        cpu: 1
        memory: "200Mi"
  restartPolicy: Never
  ```

Scheduled pod is assigned `Guaranteed` quality of service class, this can be examined by issuing `kubectl describe pod/test-pod`.

Part of sample ouput is:

```yaml
QoS Class:       Guaranteed
```

Invidual processes/threads processor affinity can be checked on the node where the pod was scheduled with `taskset` command.
Process started by a container with `Guaranteed` pod QoS class has set CPU affinity according to the POD definition. It runs exclusively on CPUs removed from shared pool. All processes spawned from pod assigned to `Guaranteed`  QoS class are scheduled to run on the same exclusive CPU. Processes from `Burstable` and `BestEffort` QoS classes PODs are scheduled to run on shared pool CPUs. This can be examined with example nginx container.

```bash
[root@vm ~]# for p in `top -n 1 -b|grep nginx|gawk '{print $1}'`; do taskset -c -p $p; done
pid 5194's current affinity list: 0,1,3-7
pid 5294's current affinity list: 0,1,3-7
pid 7187's current affinity list: 0,1,3-7
pid 7232's current affinity list: 0,1,3-7
pid 17715's current affinity list: 2
pid 17757's current affinity list: 2
```
