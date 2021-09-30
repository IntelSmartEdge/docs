```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```
- [Overview](#overview)
  - [Operating System](#operating-system)
  - [Package Versions](#package-versions)
  - [Hardware](#hardware)
    - [Server](#server)
    - [Configuration](#configuration)
  - [Building Blocks](#building-blocks)
  - [Known Issues](#known-issues)
  - [Disclaimer](#disclaimer)

# Overview

The 21.09 release of Intel® Smart Edge Open introduces the Developer Experience Kit. Edge developers can use the Developer Experience Kit as a cloud-native platform for testing edge applications.

This document describes the hardware and software configuration used to test the experience kit. For architecture and installation information, see the [Developer Experience Kit documentation](/experience-kits/developer-experience-kit.md).

## Operating System

Ubuntu 20.04.2 LTS (Focal Fossa)

## Package Versions

 | Package     | Version           |
 | ----------- |:-----------------:|
 | Kubernetes |  1.21.1 |
 | Docker | 20.10.2 |
 | Calico | 3.19 | 
 | Harbor | 2.3.2 |
 | Multus | 3.7.1 |
 | NFD | 0.8.2 |
 | StatsD | 0.22.0 |
 | Prometheus | 2.30.0 |
 | Grafana | 8.1.5 |
 | cAdvisor | 0.40.0 |
 | SR-IOV Network Operator | c608feee2dd74b4b99e44d4950b3651040e68a65 |


## Hardware 

### Server

Dell PowerEdge R750 Server R750 motherboard 

### Configuration  

2 Intel® Xeon® Gold 6338N Processors: 2.2G, 32C/64T, 11.2GT/s, 48M Cache, Turbo, HT (185W) DDR4-2666    
1 2.5 Chassis    
1 N9
1 No Rear Storage    
1 Additional Processor Selected    
1 NVMe Backplane    
1 GPU Enablement    
1 iDRAC,Legacy Password    
1 iDRAC Group Manager, Disabled    
1 2.5" Chassis with up to 16 NVMe Drives    
1 PowerEdge 2U LCD Bezel    
1 Riser Config 7, 2x8, 2x16 slots    
1 Dell EMC Luggage Tag    
1 No Quick Sync     
1 Performance Optimized    
1 3200MT/s RDIMMs     
16 32GB RDIMM, 3200MT/s, Dual Rank    
1 iDRAC9, Enterprise 15G    
1 No Hard Drive     
1 1.6TB Enterprise NVMe Mixed Use AG Drive U.2 Gen4 with carrier     
1 BOSS-S2 controller card + with 2 M.2 480GB (RAID 1)    
1 No Controller     
1 Heatsink for 2 CPU with GPU configuration    
1 Dual, Hot-Plug,Power Supply Redundant (1+1), 1400W, Mixed Mode     
2 C13 to C14, PDU Style, 10 AMP, 6.5 Feet (2m), Power Cord     
1 Trusted Platform Module 2.0 V3     
1 Order Configuration Shipbox Label (Ship Date, Model, Processor Speed, HDD Size, RAM)      
1 Asset Tag - ProSupport (Website, barcode, Onboard MacAddress)      
1 BOSS Cables and Bracket for R750 (Riser 1)      
1 PowerEdge R750 Shipping Material     
1 GPU Ready Configuration Cable Install Kit R750      
1 Intel E810-XXV Dual Port 10/25GbE SFP28, OCP NIC 3.0    
1 Intel E810-XXV Dual Port 10/25GbE SFP28 Adapter, PCIe Full Height      
1 Very High Performance Fan x6 V3     
1 Fan Foam, HDD 2U    
1 Power Saving Dell Active Power Controller    
1 C30, No RAID for NVME chassis Software   
1 TPM Module     

## Building Blocks
The Developer Experience Kit uses the following building blocks:
- **SR-IOV Network Operator** - Operator for provisioning and configuring the SR-IOV CNI and device plugins supporting Intel network adapters. Use this feature to allocate dedicated high performance SR-IOV virtual interfaces to the application pods on the cluster. 
- **Multus**: A CNI that enables attaching multiple network interfaces to a Kubernetes pod. Typically, in Kubernetes each pod only has one network interface, apart from a loopback. With Multus you can create a multi-homed pod with multiple interfaces. 
- **Harbor**: An open source registry that secures artifacts with policies and role-based access control. Use Harbor to store application container images and Helm charts that can easily be deployed on the node. 
- **Prometheus, Grafana, cAdvisor, StatsD**: Cloud native telemetry, observability, logging, monitoring and dashboard component, used to create an observability framework for application and resource utilization. 
- **Node Feature Discovery (NFD)**: Software that detects hardware features available on each node in a Kubernetes cluster, and advertises those features using node labels. Use the labels created by NFD to deploy applications on nodes that meet required criteria for reliable service. 
- **Calico**: An open source networking and network security solution for containers, virtual machines, and native host-based workloads. Calico supports network policy and high-performance data plane. It is the default CNI on the edge node cluster created by the Developer Experience Kit. 

## Known Issues

- Edge Software provisioner - Occasionally the USB image is not built and SE-O DEK provision exits with an error. 
  - Mitigation: Retry the ESP based provisioning.
- Edge Software provisioner - sometimes builds an incorrect image and machine fails to boot using the image. 
  - Mitigation: Retry the ESP based provisioning.
- Edge Software provisioner - Occasionally when running ESP's build.sh the /dev/null node of the provisioning server machine is corrupted.
  - Mitigation: run command: `rm -f /dev/null; mknod -m 666 /dev/null c 1 3` or reboot the server
- Edge Software provisioner - Cannot boot USB images using legacy BIOS.
  - Mitigation: Use UEFI BIOS


## Disclaimer

Smart Edge Open 21.09 does not include the latest functional and security updates. Smart Edge Open is targeted to be released in Q4'2021 and will include additional functional and security updates. Customers should update to the latest version as it becomes available.