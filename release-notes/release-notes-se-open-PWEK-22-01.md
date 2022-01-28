```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2022 Intel Corporation
```
- [Overview](#overview)
- [New in the release](#new-in-the-release)
- [Operating System](#operating-system)
- [Package Versions](#package-versions)
- [Hardware](#hardware)
  - [Servers](#servers)
  - [Dell PowerEdge Server Configuration](#dell-poweredge-server-configuration)
- [Building Blocks](#building-blocks)
- [Known Issues](#known-issues)

# Overview

The 22.01 release of Intel® Smart Edge Open provides edge developers with the Private Wireless Experience Kit as a cloud-native platform for delivering AI, video, and other edge services together with network services with optimized performance on Intel edge platforms.

This document describes the hardware and software configuration used to test the experience kit. For architecture and installation information, see the 
[Private Wireless Experience Kit documentation](/experience-kits/private-wireless-experience-kit-all-in-one.md).

# New in the release

 This release upgrades the Private Wireless Experience Kit to the new Intel® Smart Edge architecture with upgrades for provisioning, Kubernetes operators, and reference edge applications.

# Operating System

Centos OS 7.9.2009

# Package Versions

| Package        | Version                           |
| --------------- | :------------------------------ |
| DU-L1           | Verified FlexRAN BBU v20.11.    |
| DU-L2           | Verified Radisys L2DU v2.2.     |
| CU-L2L3         | Verified Radisys L2L3CU v2.2.   |
| UPF,AMF and SMF | Verified Radisys 5GC v2.2.      |
|Kubernetes       |v1.22.2                          |
|Calico           |v3.19                            |
|Kube-virt        |v0.42.1                          |
|SR-IOV DP & operator   |DP: v3.3.2, Operator: 4.9.0|
|Centos OS        |7.9.2009                         |
|linuxptp         |2.0                              |
|Intel® Quick Assist Technology (Intel® QAT) Device Plugin |0.17.0                           |
|WNR-ITM          |3.0.0                            |
| RT kernel       | 3.10.0-1127.19.1.rt56.1116.el7.x86_64 |


# Hardware 

| Hardware                                         | Notes                                                        |
| ------------------------------------------------ | :----------------------------------------------------------- |
| Two servers with 3rd Generation Intel® Xeon® Scalable Processors                               | Used to host the Kubernetes edge and control plane nodes. The Private Wireless Experience Kit has been validated to run on the recommended Dell EMC PowerEdge R750 server, for customized server please follow How to Customize the Configuration session in white paper.               |
| Intel® QuickAssist Adapter 8970              | Need to be inserted into the Edge Node. Verified firmware version is 4.7.0. |
| Two Intel Ethernet Network Adapter X710DA4FH NIC | Need to be inserted into the Edge Node.  Verified firmware version is 8.30 0x8000a49d 1.2960.0. |
| 1588 PTP Grandmaster Clock and GPS Receiver      | gNodeB and RRH needs PTP time syncronization.                                                   |
| Intel® ACC100                                | Need to be inserted into the Edge Node. A Dedicated FEC vRAN Accelerator. |
| Sub6 4x4 RRH                                 | Verified Foxconn Sub6 4x4 RRH - RPQN-7800, 3.3-3.6GHz with firmware version: v1.0.3q 432.        |
| 5G Mobile phone                                  | Please contact your local Intel® representative for more information about mobile phone and USIM card. |

## Servers

The 22.01 release of Intel® Smart Edge Open Private Wireless Experience Kit supports the hardware platform on Dell PowerEdge R750 Server motherboard

Note: The same system specification is used for Edge node and Control Plane node. Some accelerator cards will not be needed.

## Dell PowerEdge Server Configuration  
2 Intel® Xeon® Gold 6338N Processors: 2.2G, 32C/64T, 11.2GT/s, 48M Cache, Turbo, HT (185W) DDR4-2666    
1 iDRAC,Legacy Password    
1 iDRAC Group Manager, Disabled    
1 2.5" Chassis with up to 16 NVMe Drives     
1 Riser Config 7, 2x8, 2x16 slots    
1 Performance Optimized    
1 3200MT/s RDIMMs     
16 32GB RDIMM, 3200MT/s, Dual Rank    
1 iDRAC9, Enterprise 15G     
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
1 Very High Performance Fan x6 V3     
1 Fan Foam, HDD 2U    
1 Power Saving Dell Active Power Controller    
1 C30, No RAID for NVME chassis Software   
1 TPM Module
1 Intel® QuickAssist Adapter 8970
1 Intel® vRAN Dedicated Accelerator ACC100 Adapter
2 Intel® Ethernet Network Adapter X710DA4FH NIC

# Building Blocks
The Private Wireless Experience Kit uses the following building blocks:
- **Intel® QAT Device plugin for Kubernetes**: The Intel® QuickAssist Technology (Intel® QAT) device plugin provides support for Intel® QAT devices under Kubernetes and enables users to harness Intel devices to increase performance and efficiency across applications and platforms.  Intel® QAT takes advantage of the feature that Kubernetes provides a device plugin framework that is used to advertise system hardware resources. In the Private Wireless Experience Kit, CU applications consume resources allocated by the Intel® QAT device plugin as crypto devices.
- **SR-IOV Network Operator**: Provides an elegant user interface that simplifies SR-IOV networking set up by provisioning and configuring plugins for the SR-IOV CNI and NIC. 
- **SR-IOV FEC Operator**: Orchestrates and manages the resources exposed by Intel® Forward Error Correction Devices (Intel® FEC Devices). The FEC operator is a state machine that configures and monitors resources, acting on them autonomously based on the user interaction.
- **Node Feature Discovery (NFD)**: Detects hardware features available on each node in a Kubernetes cluster and advertises those features using node labels.
- **Topology Manager**: Allows users to align their CPU and peripheral device allocations by NUMA (non-uniform memory access) node.
- **Precision Time Protocol (PTP)**: Provides time synchronization between machines connected through Ethernet. The primary clock serves as a reference clock for secondary nodes. A grand master clock (GMC) can be used to precisely set the primary clock.
- **Multus**: A CNI that enables attaching multiple network interfaces to a Kubernetes pod. Typically, in Kubernetes each pod only has one network interface, apart from a loopback. With Multus you can create a multi-homed pod with multiple interfaces. 
- **Harbor**: An open source registry that secures artifacts with policies and role-based access control. Use Harbor to store application container images and Helm charts that can easily be deployed on the node. 

# Known Issues

- Sometime system (Dell R750 edge node) hangs during reboot with Ethernet card based on Intel® Ethernet Controller XL710. This is because Intel® Ethernet Controller XL710 based Ethernet cards are not in the DELL official support list.
  - Mitigation: Private Wireless Experience Kit will upgrade to NIC E810 next release.
- 5G CNFs - UE reattach fails sometimes UE moves from idle mode to active mode.
  - Mitigation: This issues will be addressed with future 5G Core CNFs upgrades.