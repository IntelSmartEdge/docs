```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2022 Intel Corporation
```
# The Intel® Smart Edge Open Developer Experience Kit building blocks

Installing the Developer Experience Kit will install Kubernetes as well as the following building blocks on your edge node and controller.

**Note:** Different building blocks are installed on your system based on your installation method. You will learn more about the default and advanced installation methods in the next section. 

## Edge node building blocks 

| Building Block | Functionality     |     Version | Advanced installation only | 
| :------------- | :------------- | :------------- | :------------- |
|[Calico CNI](https://docs.projectcalico.org/about/about-calico) | Default container network interface |  3.20 |   | 
[Node Feature Discovery (NFD)](/components/resource-management/node-feature-discovery.md) |Detects and advertises the hardware features available in each node of a Kubernetes* cluster | 0.9.0 |   |  
|[cert-manager](https://cert-manager.io/docs/)| Adds certificates and certificate issuers as resource types in the cluster, and simplifies the process of obtaining, renewing and using those certificates |   |   X  | 
| [SR-IOV Network Operator](/components/networking/sriov-network-operator.md) | Additional container network interface |    | X  |
| [Multus CNI](/components/networking/multus.md) | Support for multiple network interfaces | 3.8 |  | X |
| [Harbor](https://goharbor.io/) | Cloud native registry service that stores and distributes container images | 2.3.4  |  |  X |
| [Telemetry](/components/telemetry/telemetry.md) | Remote collection of device data for real-time monitoring|   |   | X |
| [Topology Manager](/components/resource-management/topology-manager.md) |Coordinates the resources allocated to a workload |   | X  |
| [CPU Manager](/components/resource-management/cpu-manager.md) | Allows workloads to be assigned to a designated CPU core. |   | X |
| [Intel® SecL-DC](/components/security/platform-attestation-using-isecl.md) | Isecl components to provide platform attestation on the edge node|   |  X |
| [Intel® SGX](/components/security/application-security-using-sgx.md) | Provides application security |  | X |

## Controller building blocks

| Building Block | Functionality     |     Version | Advanced installation only | 
| :------------- | :------------- | :------------- |  :------------- |
| [SR-IOV Network Operator](/components/networking/sriov-network-operator.md) | Additional container network interface |    |   |
| [Multus CNI](/components/networking/multus.md) | Support for multiple network interfaces | 3.8 |  |
| [Harbor](https://goharbor.io/) | Cloud native registry service that stores and distributes container images | 2.3.4  |  |
| [Telemetry](/components/telemetry/telemetry.md) | Remote collection of device data for real-time monitoring|   |   |
| [Topology Manager](/components/resource-management/topology-manager.md) |Coordinates the resources allocated to a workload |   |   |
| [CPU Manager](/components/resource-management/cpu-manager.md) | Allows workloads to be assigned to a designated CPU core. |   |  |
| [Calico CNI](https://docs.projectcalico.org/about/about-calico) | Default container network interface |  3.20 |   |  X |
| [Node Feature Discovery (NFD)](/components/resource-management/node-feature-discovery.md) |Detects and advertises the hardware features available in each node of a Kubernetes* cluster | 0.9.0 | X  | 
|[cert-manager](https://cert-manager.io/docs/)| Adds certificates and certificate issuers as resource types in the cluster, and simplifies the process of obtaining, renewing and using those certificates |   |   X  | 
| [Intel® SGX DCAP](/components/security/application-security-using-sgx.md) | Provides application security |  | X |


### Next

The next section explains how to install the Intel® Smart Edge Open Developer Experience Kit. 