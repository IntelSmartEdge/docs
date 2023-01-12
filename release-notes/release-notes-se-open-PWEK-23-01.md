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

The 23.01 release of Intel® Smart Edge for builders provides edge solution builders with the Private Wireless Experience Kit with cloud-native capabilities for delivering Private 5G network functions with AI, video, and other edge services with optimized performance on Intel edge platforms.

This document describes the hardware and software configuration required to install the experience kit. For architecture and installation information, see the [Private Wireless Experience Kit documentation](/experience-kits/private-wireless-experience-kit.md).

# New in the release

This release provides bundle design and integration based on Radisys RAN CNFs upgraded version v3.2.2 with FlexRAN 22.07.
 
The upgraded Radisys RAN enhances the performance and stability, and also provides support for the near-Real-time RAN Intelligent Controller (near-RT RIC) functionality through E2 interface support. For bundle design, it provides one guidance of easy CNFs deployment and configuration, and extends to initial multi vendors framework scheme. Staged Deployment through Edge SW Provisioner (ESP) as default option meets diversity deployment requirement better.

For reference edge applications, this release offers an edge application for software-defined multi-access solution - the Multi-Access with Private 5G. The solution has a client software part (running at the PC/UE/IoT device side) and a server software part (running at the Edge Server). The Generic Multi-Access (GMA) client software connects to the GMA server software over cellular and Wi-Fi connectivity. This RI demonstrates a new control protocol to manage traffic steering, splitting, and duplicating across multiple connections.

# Operating System

Centos OS 7.9.2009

# Package Versions
 
| Package         | Version                               |
| --------------- | :-------------------------------------|
| DU-L1           | validated with FlexRAN BBU v22.07     |
| DU-L2           | validated with Radisys L2DU v3.2.2    |
| CU-L2L3         | validated with Radisys L2L3CU v3.2.2  |
| UPF             | validated with Radisys 5GC v3.0.1     |
| AMF and SMF     | validated with Radisys 5GC v2.5.1     |
| Kubernetes      | v1.23.3                               |
| Calico          | v3.21.6                               |
| Centos OS       | 7.9.2009                              |
| RT kernel       | 3.10.0-1127.19.1.rt56.1116.el7.x86_64 |
| linuxptp        | 2.0                                   |
| Intel QAT plugin| 0.23.0                                |
| Wireless Network Ready Intelligent Traffic Management | 5.0.0  |
| Multi-Access with Private 5G | 1.0.0  |


# Hardware 

| Hardware                                         | Notes                                                        |
| ------------------------------------------------ | :----------------------------------------------------------- |
| Two servers with 3rd Generation Intel® Xeon® Scalable Processors                               | Used to host the Kubernetes edge and control plane nodes. The Private Wireless Experience Kit has been validated to run on a Dell EMC PowerEdge R750.               |
| Intel® QuickAssist Adapter 8970              | Need to be added into the Edge Node. Verified firmware version is 4.7.0. |
| One Intel Corporation Ethernet Controller E810-XXV for SFP (rev 02) |  Need to be added into the Edge Node.  Verified firmware version is 3.00 0x80008944 20.5.13 |
| 1588 PTP Grandmaster Clock and GPS Receiver      | gNodeB and RRH need PTP time syncronization.                                                   |
| Intel® ACC100                                | Need to be added into the Edge Node. A Dedicated FEC vRAN Accelerator. |
| Sub6 4x4 RRH                                 | Verified Foxconn Sub6 4x4 RRH - RPQN-7800, 3.3-3.6GHz with firmware version: v1.4.14q 524.        |
| 5G Mobile phone                                  | Please contact your local Intel® representative for more information about mobile phone and USIM card. |

## Servers

The 23.01 release of Intel® Smart Edge Open Private Wireless Experience Kit supports the hardware platform on Dell PowerEdge R750 Server motherboard.

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
1 Intel® Ethernet Controller E810-XXV for SFP (rev 02)   

# Building Blocks
The Private Wireless Experience Kit uses the following building blocks:
- **Intel® QAT Device plugin for Kubernetes**: The Intel® QAT device plugin provides support for Intel QAT devices under Kubernetes and enables users to harness Intel devices to increase performance and efficiency across applications and platforms. QAT takes advantage of the feature that Kubernetes provides a device plugin framework that is used to advertise system hardware resources. For Private Wireless Experience Kit, CU applications consume the QAT resources allocated by QAT device plugin as crypto devices. 
- **SR-IOV Network Operator**: Provides an elegant user interface that simplifies SR-IOV networking set up by provisioning and configuring plugins for the SR-IOV CNI and NIC. 
- **SR-IOV FEC Operator**: Orchestrates and manages the resources exposed by Intel® Forward Error Correction Devices (Intel® FEC Devices). ACC100 is an example of the FEC acceleration devices/hardware that are supported with Private Wireless Experience Kit. The FEC operator is a state machine that configures and monitors resources, acting on them autonomously based on the user interaction. 
- **Node Feature Discovery (NFD)**: Detects hardware features available on each node in a Kubernetes cluster and advertises those features using node labels. 
- **Topology Manager**: Allows users to align their CPU and peripheral device allocations by NUMA (non-uniform memory access) node.
- **Precision Time Protocol (PTP)**: Provides time synchronization between machines connected through Ethernet. The primary clock serves as a reference clock for secondary nodes. A grand master clock (GMC) can be used to precisely set the primary clock.
- **CPU Manager**: CPU manager enables better placement of workloads in Kubelet. APPs and CNFs can get higher performance with exclusive CPUs allocated to them.

# Known Issues

- BLER high issue may happen as power control functionality is not supported yet. 
  - Mitigation: UE position needs to be adjusted manually during test. Radisys will provide this functionality with future version.
- UE attachment failure issue may happen if the 2 QAT VF CU requests are from different QAT PFs. 
  - Mitigation: Private Wireless Experience Kit disables QAT VF automated assignment. The resource is allocated manually. 
