```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```
- [Overview](#overview)
  - [Operating System](#operating-system)
  - [Package Versions](#package-versions)
  - [Hardware](#hardware)
    - [Servers](#servers)
    - [Dell PowerEdge Server Configuration](#dell-poweredge-server-configuration)
    - [Moro City Server Configuration](#moro-city-server-configuration)
  - [Building Blocks](#building-blocks)
  - [Known Issues](#known-issues)1

# Overview

The 21.12 release of Intel® Smart Edge Open provides Edge developers with the Developer Experience Kit as a cloud-native platform for testing edge applications.
In this release the Developer Experience Kit is updated with:
- enabling hardware security features on Dell PowerEdge R750 Server
- supporting ICX-D as a platform.

This document describes the hardware and software configuration used to test the experience kit. For architecture and installation information, see the [Developer Experience Kit documentation](/experience-kits/developer-experience-kit.md).

## Operating System

Ubuntu 20.04.2 LTS (Focal Fossa)

## Package Versions

 | Package     | Version           |
 | ----------- |:-----------------:|
 | Kubernetes |  1.22.2 |
 | Docker | 20.10.11 |
 | Calico | 3.20 | 
 | Harbor | 2.3.4 |
 | Multus | 3.8 |
 | NFD | 0.9.0 |
 | StatsD | 0.22.2 |
 | Prometheus | 2.30.0 |
 | Grafana | 8.1.5 |
 | cAdvisor | 0.40.0 |
 | SR-IOV Network Operator | ff8c2f1e912b42b45bc2f4591c4d3a4749ae9574 |
 | Cert Manager | 1.6.1 |


## Hardware 

### Servers

The 20.12 release of Intel® Smart Edge Open supports two hardware platforms:
-  Dell PowerEdge R750 Server motherboard
-  Idaville HCC Platform, Moro City (RP)  


### Dell PowerEdge Server Configuration  

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

### Moro City Server Configuration  

1 CPU socket populated with Intel(R) Genuine processor, ICE LAKE-D-22 (ICX-D) HCC, 20C
1 Intel® Ethernet Network Adapter CPK 2x25G
1 uATX form factor board  
4 channel DDR4 with single iMC RDDII/UDIMM, VLP-RDIMM 
2 QSFP28 up to 100GbE 
2 ACPI power states (S0, S5), and non-ACPI power sequences  
1 eMMC module support 
1 RMM RJ45 BMC  
1 BMC down support with on board VGA  
2 x16 PCI Express PCIe 4.0 slots  
1 x8 PCI Express PCIe 3.0 slot (HSIO) 
1 x4 PCI Express PCIe 3.0 m.2 
1 8x lane Mini-SAS HD SATA  
2 USB 3.0  
1 RMM4 DNM  
1 IEEE 1588/SyncE DPLL support  
2 32Gb 2933MT/s RDIMMs  



## Building Blocks
The Developer Experience Kit uses the following building blocks:
- **SR-IOV Network Operator** - Operator for provisioning and configuring the SR-IOV CNI and device plugins supporting Intel network adapters. Use this feature to allocate dedicated high performance SR-IOV virtual interfaces to the application pods on the cluster. 
- **Multus**: A CNI that enables attaching multiple network interfaces to a Kubernetes pod. Typically, in Kubernetes each pod only has one network interface, apart from a loopback. With Multus you can create a multi-homed pod with multiple interfaces. 
- **Harbor**: An open source registry that secures artifacts with policies and role-based access control. Use Harbor to store application container images and Helm charts that can easily be deployed on the node. 
- **Prometheus, Grafana, cAdvisor, StatsD**: Cloud native telemetry, observability, logging, monitoring and dashboard component, used to create an observability framework for application and resource utilization. 
- **Node Feature Discovery (NFD)**: Software that detects hardware features available on each node in a Kubernetes cluster, and advertises those features using node labels. Use the labels created by NFD to deploy applications on nodes that meet required criteria for reliable service. 
- **Calico**: An open source networking and network security solution for containers, virtual machines, and native host-based workloads. Calico supports network policy and high-performance data plane. It is the default CNI on the edge node cluster created by the Developer Experience Kit.
- **Cert Manager**: An open source solution for adding certificates and certificate issuers as resource types in the cluster. It simplifies the process of obtaining, renewing and using those certificates.
- **IsecL**: Intel® Security Libraries enable platform attestation capability on the edge node. (Feature available only on Dell PowerEdge R750 Server)
- **SGX**: Enables application security by configuring secure memory enclaves where sensitive application data is protected. (Feature available only on Dell PowerEdge R750 Server)

## Known Issues

- Edge Software provisioner - Occasionally the USB image is not built and SE-O DEK provision exits with an error. 
  - Mitigation: Retry the ESP based provisioning.
- Edge Software provisioner - sometimes builds an incorrect image and machine fails to boot using the image. 
  - Mitigation: Retry the ESP based provisioning.
- Edge Software provisioner - Occasionally when running ESP's build.sh the /dev/null node of the provisioning server machine is corrupted.
  - Mitigation: run command: `rm -f /dev/null; mknod -m 666 /dev/null c 1 3` or reboot the server
- Edge Software provisioner - Cannot boot USB images using legacy BIOS.
  - Mitigation: Use UEFI BIOS

