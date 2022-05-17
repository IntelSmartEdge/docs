```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2022 Intel Corporation
```
- [Overview](#overview)
  - [Secure Key Management using KMRA](#Secure-Key-Management-using-KMRA)
  - [Secure and Trusted Computing](#secure-and-trusted-computing)
  - [RAN Intelligence and its Application](#ran-intelligence-and-its-application)
  - [Reference Implementation on Intel ESH](#reference-implementation-on-intel-esh)
  - [Operating System](#operating-system)
  - [Package Versions](#package-versions)
  - [Hardware](#hardware)
    - [Servers](#servers)
    - [Dell PowerEdge Server Configuration](#dell-poweredge-server-configuration)
  - [Building Blocks](#building-blocks)
  - [Known Issues](#known-issues)

# Overview

The 22.03 release of Intel® Smart Edge Open provides Edge developers with the Developer Experience Kit as a cloud-native platform for testing edge applications.
In this release the Developer Experience Kit is updated with:

## Secure Key Management using KMRA

- Key Management Reference Application (KMRA) is a proof-of-concept software created to demonstrate the integration of Intel® Software Guard Extensions (Intel® SGX) asymmetric key capability with a hardware security model (HSM) on a centralized key server. This reference application sets up an NGINX workload to access the private key in an Intel® SGX enclave on a 3rd Generation Intel® Xeon® Scalable processor, using the Public-Key Cryptography Standard(PKCS) #11 interface and OpenSSL. TLS is the Transport Layer Security protocol for applications that secure communications over computer networks against eavesdropping or need to identify the party at the other end. It is widely used by Internet servers, including the majority of HTTPS websites. KMRA uses DCAP (Data Center Attestation Primitives) libraries for generating and verifying the ECDSA signed Intel SGX quote. For more details, refer [Secure Key Management using KMRA](/components/security/secure-key-management-using-sgx.md)

## Secure and Trusted Computing

### Remote Attestation
- Trusted computing consists primarily of two activities – measurement, and attestation. Measurement is the act of obtaining cryptographic representations for the system state. Attestation is the act of comparing those cryptographic measurements against expected values to determine whether the system booted into an acceptable state. This is key for Edge Computing in public and private/On-Premises deployment. Intel® Smart Edge Open supports this feature using Intel® Security Libraries (ISecL).

### Enclave Trust with Attestation:
- Handling sensitive data key requirement for Edge Computing in public and private/On-Premises deployment. Intel® Smart Edge Open supports this feature using The Intel® Software Guard Extensions (Intel® SGX). Remote attestation allows a remote party to check that the intended software is securely running within an enclave on a system with the Intel® SGX enabled.

## RAN Intelligence and its Application

- With the increased demand for vRAN/Cloud-native RAN deployments O-RAN Alliance has defined a scalable Architecture and Interfaces for deploying 5G network functions and services. Intel® Smart Edge Open DEK supports reference implementation of
### ORAN Near Real-Time RIC:
- A logical function that enables near-real-time control and optimization of O-RAN elements and resources via fine-grained data collection and actions. DEK uses ONF SD-RAN as a reference implementation for this.
### xAPP:
- Defined as an independent software plug-in to the Near-RT RIC platform to provide functional extensibility to the RAN. DEK supports Intelligent Connection Management Application for Automated Handover a reference xAPP.  

## Reference Implementation on Intel Developer Catalog

- EII/PCB Defect detection, Telehealth Remote Monitoring, Intelligent Traffic Management and Smart VR- Live Streaming of Immersive Media Reference Implementation are  supported on Intel® Smart Edge Open 21.12 onwards.
- These Reference Implementation are available at the Intel® Developer Catalog https://www.intel.com/content/www/us/en/developer/tools/software-catalog/overview.html?s=Newest&f:@emtcontenttype_en=[Software%20Reference%20Implementations]


This document describes the hardware and software configuration used to test the experience kit. For architecture and installation information, see the [Developer Experience Kit documentation](/experience-kits/developer-experience-kit.md).

## Operating System

Ubuntu 20.04.2 LTS (Focal Fossa)

## Package Versions

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
 | Prometheus | 2.30.0 |
 | Grafana | 8.4.2 |
 | cAdvisor | 0.43.0 |
 | SR-IOV Network Operator | 1.1.0 |
 | Cert Manager | 1.6.1 |
 | isecl-trust-agent | 4.2 |
 | isecl-controlplane-services | 4.2 |
 | ansible | 2.9.27 |
 | Helm | 3.8.0 |
 | KMRA | 2.1 |
 | BMRA | 22.01 |

## Hardware

### Servers

The 22.03 release of Intel® Smart Edge Open supports the following hardware platform:
-  Dell PowerEdge R750 Server motherboard


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
1 Additional Disk for Rook-ceph


## Building Blocks
The Developer Experience Kit uses the following building blocks:
- **SR-IOV Network Operator** - Operator for provisioning and configuring the SR-IOV CNI and device plugins supporting Intel network adapters. Use this feature to allocate dedicated high performance SR-IOV virtual interfaces to the application pods on the cluster.
- **Multus**: A CNI that enables attaching multiple network interfaces to a Kubernetes pod. Typically, in Kubernetes each pod only has one network interface, apart from a loopback. With Multus you can create a multi-homed pod with multiple interfaces.
- **Harbor**: An open source registry that secures artifacts with policies and role-based access control. Use Harbor to store application container images and Helm charts that can easily be deployed on the node.
- **Prometheus, Grafana, cAdvisor, StatsD**: Cloud native telemetry, observability, logging, monitoring and dashboard component, used to create an observability framework for application and resource utilization.
- **Node Feature Discovery (NFD)**: Software that detects hardware features available on each node in a Kubernetes cluster, and advertises those features using node labels. Use the labels created by NFD to deploy applications on nodes that meet required criteria for reliable service.
- **Calico**: An open source networking and network security solution for containers, virtual machines, and native host-based workloads. Calico supports network policy and high-performance data plane. It is the default CNI on the edge node cluster created by the Developer Experience Kit.
- **Cert Manager**: An open source solution for adding certificates and certificate issuers as resource types in the cluster. It simplifies the process of obtaining, renewing and using those certificates.
- **IsecL**: Intel® Security Libraries enable platform attestation capability on the edge node. (Feature available only on Dell PowerEdge R750 Server). It supports following features
  - Secure platform attestation using UEFI secure boot and TPM 2.0
  - Remote attestation with measurements extended to System TPM quote as per the Trusted Compute Group specification (https://trustedcomputinggroup.org/tpm-2-0-library-specification-approved-isoiec-international-standard/)
  - Remote attestation support multiple system boot measurement options(Flavours like PLATFORM, HOST, OS, SOFTWARE).
  - Data  sovereignty allows physical TPMs to be written with Asset Tags containing any number of key/value pairs to identify hosts that meet specific compliance requirements and can run controlled workloads.
  - Conditional K8s node labelling depending on the verified measurements “flavour” (and Asset Tags)
- **SGX**: Enables application security by configuring secure memory enclaves where sensitive application data is protected. (Feature available only on Dell PowerEdge R750 Server). It supports following features
  - Data Centre Attestation Primitives(DCAP) ECDSA based attestation for SGX
  - Intel SGX device plugin
  - Basic infrastructure support to run Gramine/SGX based applications.
  - Note:
    - This is a Pre-production release.
    - No security related claims are made in this release.
    - This release demonstrates the concept of secure enclaves for run time security using Intel SGX DCAP. The steps to build and deploy a sample are documented. The actual implementation is customer (application) specific.
- **Rook-Ceph**: Ceph is an open-source, highly scalable, distributed storage solution for block storage, shared filesystems, and object storage. Rook-Ceph is an open-source cloud-native storage orchestrator that transforms storage software into self-managing, self-scaling, and self-healing storage services.
- **KubeVirt**: KubeVirt is an open-source project extending Kubernetes\* with a Virtual Machine (VM) support via Custom Resource Definitions (CRDs) and easy to deploy KubeVirt agents and controllers. KubeVirt addresses a need to allow non-containerizable applications/workloads inside VMs to be treated as K8s managed workloads. This allows for both VM and container/pod applications to coexist within a shared K8s environment, allowing for communication between the K8s pods, VMs, and services on the same cluster. It is important that virtualization is supported by the system (this might require changing BIOS settings).

- **KMRA**: Key Management Reference Application (KMRA) provides end to end solution for run time protection of customer private keys by provisioning them into Intel SGX enclaves. The keyserver is deployed on the AWS cloud and Crypto-toolkit along with NGINX workload is deployed on the edge-node after DEK provisioning. KMRA also supports Multitenancy by running mutliple instances of Intel SGX enclaves.

## Known Issues

- Edge Software provisioner - Occasionally the USB image is not built and SE-O DEK provision exits with an error.
  - Mitigation: Retry the ESP based provisioning.
- Edge Software provisioner - sometimes builds an incorrect image and machine fails to boot using the image.
  - Mitigation: Retry the ESP based provisioning.
- Edge Software provisioner - Cannot boot USB images using legacy BIOS.
  - Mitigation: Use UEFI BIOS
- In Dell PowerEdge R750 TPM endorsement key (EK) is not signed by known Certificate Authority. Our application(ISecL HVS) failed to verify TPM EK certificate.
  - Mitigation: Admin has to provision the root CA certificate of TPM EK to HVS in out of band mode.
- When the existing PCCS deployment on the cloud instance (AWS) exceeds 24Hrs and user is trying to provision new DEK cluster sometimes there might be failure of provisioning.
  - Mitigation: Reset SGX using "SGX Factory Reset" option in the BIOS.
- When multiple instances of the KMRA ctk_loadkey pods are run, all the pods will receive the same private key as of 22.03 release. Multiple Key configuration is not yet supported.
  - Mitigation:  Multiple Key configuration support will be provided in next release   
- Client Key configurations at Apphsm is static as of 22.03 release.
  - Mitigation: Apphsm will be upgraded in next release.
