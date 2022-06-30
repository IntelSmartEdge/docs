```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2022 Intel Corporation
```
- [Overview](#overview)
  - [ICN-SDEWAN](#icn-sdewan)
  - [Nodus](#nodus)
  - [KubeVirt](#kubevirt)
  - [Rook-Ceph](#rook-ceph)
  - [SRIOV network operator](#sriov-network-operator)
  - [KMRA](#kmra)
  - [Secure and Trusted Computing](#secure-and-trusted-computing)
  - [Reference Implementation on Intel ESH](#reference-implementation-on-intel-esh)
- [Operating System](#operating-system)
- [Package Versions](#package-versions)
- [Hardware](#hardware)
  - [Server](#server)
  - [Configuration](#configuration)
- [Building Blocks](#building-blocks)
- [Known Issues](#known-issues)

# Overview

The 22.05 release of Intel® Smart Edge Open Secure Access Service Edge Experience Kit offers an integrated solution with SD-WAN connectivity and network security capabilities on the Intel® Smart Edge Open edge platform.
This document describes the hardware configuration and software components of Secure Access Service Edge Experience Kit 22.05 release. For architecture and installation information, see
[Secure Access Service Edge documentation](../experience-kits/secure-access-service-edge.md).

## ICN-SDEWAN

SDEWAN is a solution to enable SDWAN functionalities including multiple WAN link support, WAN traffic management, NAT, firewall, IPSec, Traffic shaping, etc. It focuses on addressing the unique challenges in edge computing environments, like resource limitations, edge overlays, traffic sanitization, automation and cost sensitivity. The solution includes the below components:
- **SDEWAN CNF**: implemented based on OpenWRT, it enhances OpenWRT Luci web interface with SDEWAN controllers to provide Restful API for network functions, configuration, and control.
- **SDEWAN Custom Resource Definition (CRD) Controller**: implemented as k8s CRD Controller, it manages CRDs (e.g. Firewall related CRDs, mwan3 related CRDs, and IPSec related CRDs, etc.) and internally calls SDEWAN Restful API to configure CNF.
- **SDEWAN Overlay Controller**: provides central control of SDEWAN overlay networks by automatically configuring the SDEWAN CNFs through SDEWAN CRD controller located in edge location clusters and hub clusters.


## Nodus

- Nodus is a network controller in Kubernetes that consists of a set of microservices running in the Kubernetes cluster control plane and in the worker nodes. It functions as an OVN network controller, and allows the user to create virtual and provider networks within a cluster and also attach the workloads with the networks. The User can define the virtual network and provider networks based on VLAN and direct interface.

## KubeVirt

- KubeVirt is an open-source project extending Kubernetes\* with a Virtual Machine (VM) support via CRDs and easy-to-deploy KubeVirt agents and controllers. KubeVirt addresses a need to allow non-containerizable applications/workloads inside VMs to be treated as K8s managed workloads. This allows for both VM and container/pod applications to coexist within a shared K8s environment, allowing for communication between the K8s pods, VMs, and services on the same cluster. It is important that virtualization is supported by the system (this might require changing BIOS settings).

## Rook-Ceph

- Ceph is an open-source, highly scalable, distributed storage solution for block storage, shared filesystems, and object storage. Rook-Ceph is an open-source cloud-native storage orchestrator that transforms storage software into self-managing, self-scaling, and self-healing storage services.

## SRIOV network operator

- Operator for provisioning and configuring the SR-IOV CNI and device plugins supporting Intel network adapters. Use this feature to allocate dedicated high performance SR-IOV virtual interfaces to the application pods on the cluster. 

## KMRA

- Key Management Reference Application (KMRA) is a proof-of-concept software created to demonstrate the integration of Intel® Software Guard Extensions (Intel® SGX) asymmetric key capability with a hardware security model (HSM) on a centralized key server. This reference application sets up an NGINX workload to access the private key in an Intel® SGX enclave on a 3rd Generation Intel® Xeon® Scalable processor, using the Public-Key Cryptography Standard(PKCS) #11 interface and OpenSSL. KMRA uses DCAP (Data Center Attestation Primitives) libraries for generating and verifying the ECDSA signed Intel SGX quote. For more details, refer to [Secure Key Management using KMRA](/components/security/secure-key-management-using-sgx.md)

## Secure and Trusted Computing

- <b>Remote Attestation</b>: Trusted computing consists primarily of two activities – measurement, and attestation. Measurement is the act of obtaining cryptographic representations for the system state. Attestation is the act of comparing those cryptographic measurements against expected values to determine whether the system booted into an acceptable state. This is key for Edge Computing in public and private/On-Premises deployment. Intel® Smart Edge Open supports this feature using Intel® Security Libraries (ISecL). For more details, refer to [Platform Attestation using ISecL](/components/security/platform-attestation-using-isecl.md)

- <b>Enclave Trust with Attestation</b>: Handling sensitive data is a key requirement for Edge Computing in public and private/On-Premises deployment. Intel® Smart Edge Open supports this feature using The Intel® Software Guard Extensions (Intel® SGX). Remote attestation allows a remote party to check that the intended software is securely running within an enclave on a system with the Intel® SGX enabled.  For more details, refer to [Application security using SGX](/components/security/application-security-using-sgx.md)

## Reference Implementation on Intel ESH

- Telehealth Reference Implementation (v3.0.0) as Video Conference as a Service(VCaaS) is supported and verified on Intel® Smart Edge Open 22.05.

This document describes the hardware and software configuration used to test the experience kit. For architecture and installation information, see the [Sample Reference Documentation](../sample/se-open-sase-22-05.md).


# Operating System

Ubuntu OS 20.04.03

# Package Versions

 | Package     | Version           |
 | ----------- |:-----------------:|
 | Kubernetes |  1.23.3 |
 | Docker | 20.10.12 |
 | Docker Compose | 1.29.1 |
 | Calico | 3.21 |
 | Harbor | 2.4.1 |
 | Multus | 3.8 |
 | NFD | 0.10.1 |
 | StatsD | 0.22.4 |
 | Prometheus | 2.32.1 |
 | Grafana | 8.4.2 |
 | cAdvisor | 0.43.0 |
 | SR-IOV Network Operator | 1.1.0 |
 | Cert Manager | 1.8.0 |
 | Ansible | 2.9.27 |
 | Docker Compose | 1.29.2 |
 | Golang | go1.17.6 |
 | Gramine | 1.0 |
 | Harbor-helm | 1.8.1 |
 | Helm | 3.8.0 |
 | KMRA | 2.1 |
 | BMRA | 22.01 |
 | Node Exporter | 1.3.1 |
 | Isecl-controlplane-services | 4.2.0 |
 | Isecl-trust-agent | 4.2.0 |
 | SGX device plugin | 0.23.0 |
 | OpenSSL | OpenSSL_1_1_1m |
 | Skopeo | 1.5.2 |
 | ICN-SDEWAN CRD controller | 0.5.2-1 |
 | ICN-SDEWAN OpenWRT CNF | 0.5.2-1 |
 | ICN-SDEWAN Overlay Controller | prod-22.06 |
 | Nodus | 6.0.0 |

# Hardware

## Server

The 22.05 release of Intel® Smart Edge Open Secure Access Service Edge Experience Kit supports the following hardware platform:
-  Dell PowerEdge R750 Server motherboard

## Configuration

The Secure Access Service Edge Experience Kit uses the following configuration:

- 2 Intel® Xeon® Gold 6338N Processors: 2.2G, 32C/64T, 11.2GT/s, 48M Cache, Turbo, HT (185W) DDR4-2666    
- 1 2.5 Chassis    
- 1 N9  
- 1 No Rear Storage    
- 1 Additional Processor Selected    
- 1 NVMe Backplane    
- 1 GPU Enablement    
- 1 iDRAC,Legacy Password    
- 1 iDRAC Group Manager, Disabled    
- 1 2.5" Chassis with up to 16 NVMe Drives    
- 1 PowerEdge 2U LCD Bezel    
- 1 Riser Config 7, 2x8, 2x16 slots    
- 1 Dell EMC Luggage Tag    
- 1 No Quick Sync     
- 1 Performance Optimized    
- 1 3200MT/s RDIMMs     
- 16 32GB RDIMM, 3200MT/s, Dual Rank    
- 1 iDRAC9, Enterprise 15G    
- 1 No Hard Drive     
- 1 1.6TB Enterprise NVMe Mixed Use AG Drive U.2 Gen4 with carrier     
- 1 BOSS-S2 controller card + with 2 M.2 480GB (RAID 1)    
- 1 No Controller     
- 1 Heatsink for 2 CPU with GPU configuration    
- 1 Dual, Hot-Plug,Power Supply Redundant (1+1), 1400W, Mixed Mode     
- 2 C13 to C14, PDU Style, 10 AMP, 6.5 Feet (2m), Power Cord     
- 1 Trusted Platform Module 2.0 V3     
- 1 Order Configuration Shipbox Label (Ship Date, Model, Processor Speed, HDD Size, RAM)      
- 1 Asset Tag - ProSupport (Website, barcode, Onboard MacAddress)      
- 1 BOSS Cables and Bracket for R750 (Riser 1)      
- 1 PowerEdge R750 Shipping Material     
- 1 GPU Ready Configuration Cable Install Kit R750      
- 1 Intel E810-XXV Dual Port 10/25GbE SFP28, OCP NIC 3.0    
- 1 Intel E810-XXV Dual Port 10/25GbE SFP28 Adapter, PCIe Full Height      
- 1 Very High Performance Fan x6 V3     
- 1 Fan Foam, HDD 2U    
- 1 Power Saving Dell Active Power Controller    
- 1 C30, No RAID for NVME chassis Software   
- 1 TPM Module
- 1 Additional Disk for Rook-ceph

# Building Blocks

Except for **SDEWAN**, **Nodus**, **KubeVirt**, **Rook-Ceph**, **SR-IOV Network Operator**, **Intel® SecL - DC**, **Intel® SGX**, **KMRA**, the Secure Access Service Edge Experience Kit supports the following building blocks:

- **Topology Manager**: Coordinates the resources allocated to a workload
- **CPU Manager**: 	Dedicated CPU core for workload
- **Multus**: A CNI that enables attaching multiple network interfaces to a Kubernetes pod. Typically, in Kubernetes each pod only has one network interface, apart from a loopback. With Multus you can create a multi-homed pod with multiple interfaces.
- **Harbor**: An open source registry that secures artifacts with policies and role-based access control. Use Harbor to store application container images and Helm charts that can easily be deployed on the node.
- **Prometheus, Grafana, cAdvisor, StatsD**: Cloud native telemetry, observability, logging, monitoring and dashboard component, used to create an observability framework for application and resource utilization.
- **Node Feature Discovery (NFD)**: Software that detects hardware features available on each node in a Kubernetes cluster, and advertises those features using node labels. Use the labels created by NFD to deploy applications on nodes that meet required criteria for reliable service.
- **Calico**: An open source networking and network security solution for containers, virtual machines, and native host-based workloads. Calico supports network policy and high-performance data plane. It is the default CNI on the edge node cluster created by the Secure Access Service Edge Experience Kit.
- **Cert Manager**: An open source solution for adding certificates and certificate issuers as resource types in the cluster. It simplifies the process of obtaining, renewing and using those certificates.

# Known Issues

- Edge Software provisioner - Occasionally the USB image is not built and SE-O SASE EK provision exits with an error.
  - Mitigation: Retry the ESP based provisioning.
- Edge Software provisioner - sometimes builds an incorrect image and machine fails to boot using the image.
  - Mitigation: Retry the ESP based provisioning.
- Edge Software provisioner - Cannot boot USB images using legacy BIOS.
  - Mitigation: Use UEFI BIOS.
- When the existing PCCS deployment on the cloud instance (AWS) exceeds 24Hrs and user is trying to provision new SASE cloud cluster, provisioning may fail.
  - Mitigation: Reset SGX using "SGX Factory Reset" option in the BIOS.
- When multiple instances of the KMRA ctk_loadkey pods are run, all the pods will receive the same private key as of 22.05 release. Multiple Key configuration is not yet supported.
  - Will be addressed in the future releases.
- For ICN-SDEWAN/Nodus integration solution, IPsec packets with vlan tag travel between SASE POP and edge clusters due to Nodus’s provider network working in vlan mode, and this limits SASE EK to work in LAN environment only for this release.
  - Mitigation: Nodus switches to direct mode for next release.
- Openvino SGX sample in edgeapps repository is not functioning for a broken dependency emanating from a breaking change in Protobuf Python lib.
  - Mitigation: Fix in dependency expected soon using the workaround suggested in Protobuf Python lib.
