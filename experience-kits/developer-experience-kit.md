```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```

# Intel® Smart Edge Open Developer Experience Kit

## Overview

Intel® Smart Edge Open experience kits provide customized infrastructure deployments for common network and on-premises edge use cases. Combining Intel cloud-native technologies, wireless networking, and high-performance compute, experience kits let you deliver AI, video, and other services optimized for performance at the edge.

The Developer Experience Kit (DEK) lets you easily install and instantiate an Intel® Smart Edge Open edge cluster. Once the cluster has been installed, you can onboard edge applications and run reference implementations -- example end-to-end solutions built on Intel® Smart Edge Open -- to get familiar with operating a stand-alone edge node or to start creating your own solution.

## How It Works

[![Smart Edge Open Developer Experience Kit Edge Node Component Diagram](images/dek-component-diagram.png)](images/dek-component-diagram.png)

*Intel® Smart Edge Open Developer Experience Kit building blocks*

The Developer Experience Kit uses [Edge Software Provisioner](https://github.com/intel/Edge-Software-Provisioner), which automates the process of provisioning bare-metal or virtual machines with an operating system and software stack. Intel® Smart Edge Open provides a fork of the [Ubuntu OS ESP
Profile](https://github.com/intel/rni-profile-base-ubuntu) tailored for its specific needs.

### Building Blocks

Building blocks provide specific functionality in the platform you'll deploy. Each experience kit installs a set of building blocks as part of deployment. You can use additional building blocks to customize your platform, or develop your own custom solution by combining building blocks. 

The Developer Experience Kit installs Kubernetes and the following building blocks:

| Building Block | Functionality     |
| :------------- | :------------- |
|[Calico CNI](https://docs.projectcalico.org/about/about-calico) | Default container network interface |
[SR-IOV Network Operator](/components/networking/sriov-network-operator.md) | Additional container network interface |
[Multus CNI](/components/networking/multus.md) | Support for multiple network interfaces |
[Harbor](https://goharbor.io/) | Cloud native registry service that stores and distributes container images |
[Telemetry](/components/telemetry/telemetry.md) | Remote collection of device data for real-time monitoring|
[Node Feature Discovery (NFD)](/components/resource-management/node-feature-discovery.md) | Detects and advertises the hardware features available in each node of a Kubernetes* cluster |
[Topology Manager](/components/resource-management/topology-manager.md) | Coordinates the resources allocated to a workload |
[Core Pinning](/components/resource-management/core-pinning.md) | Dedicated CPU core for workload |

For information on the versions installed, see the Developer Experience Kit [release notes](https://github.com/smart-edge-open/docs/blob/main/release-notes/release-notes-se-open-DEK-21-09.md#package-versions)

## Get Started
The instructions below walk you through provisioning the operating system and Developer Experience Kit on a target system. After completing these instructions, you will have created a single edge node cluster capable of hosting edge applications. You can then optionally install reference implementations from the Intel® Edge Software Hub.

[![Smart Edge Open Developer Experience Kit Edge Node Component Diagram](images/dek-workflow-diagram.png)](images/dek-workflow-diagram.png)

### Requirements
You will need two machines: a provisioning system where you will build a bootable image of the experience kit, and a target system where you will install the experience kit to create an edge cluster.

#### Provisioning System  
- Memory: At least 4GB RAM
- Hard drive: At least 20GB
- USB flash drive
- Operating system: Ubuntu 20.04.
- Git
- Docker and Docker Compose
- Python 3.6 or later, with the PyYAML module installed
- Internet access
   
> NOTE: You must install Docker from the [Docker repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository). Installation by Docker package is not supported.
  
> NOTE: You must add the user account on the provisioning system to /etc/sudoers.

#### Target System
- A server with two sockets, each populated with a 3rd Generation Intel® Xeon® Scalable Processor
- Memory: At least 32GB RAM 
- Hard drives: Two SATA SSDs, one for booting and one for data caching
- Network adapters: Two NICs, one connected to each socket
- Connection to the provisioning system

> NOTE: The DEK was validated on a system using the following network interface card: 
> - Intel Corporation Ethernet Controller E810-C for SFP (rev 02) NIC. 
> 
> To install the DEK on a system with a different NIC, update the interface names in open-developer-experience-kits/inventory/default/group_vars/all/10-default.yaml. Otherwise, installation will fail. 
> 
> For more information, see the [SR-IOV Network Operator Configuration](https://github.com/smart-edge-open/docs/blob/main/components/networking/sriov-network-operator.md#configuration)
> 
> View the full specs of the [validated system](https://github.com/smart-edge-open/docs/blob/main/release-notes/release-notes-se-open-DEK-21-09.md). 

> NOTE: The provisioning process will install Ubuntu 20.04 on the target machine. Any existing operating system will be overwritten.

#### Knowledge

Basic knowledge of operating system, Kubernetes, and server administration.

### Install the Developer Experience Kit

The Developer Experience Kit provides a command line utility (`dek_provision.py`) which uses the
Intel® Edge Software Provisioner toolchain to deliver a smooth installation experience.

<!--
The Intel® Smart Edge Open provisioning process can rely on the experience kit's default configuration or
a configuration provided by the system operator. In the case of the default configuration, the
provisioning process is a little simpler and as such it is suggested as a good starting point for
people new to the Smart Edge Open solution.
-->

<!-- #### Quick Start (Default Configuration) -->

You must be logged in as root on the provisioning system for the following steps. To become the root user, run the following command:

```Shell.bash
[Provisioning System] $ sudo su -
```
> NOTE: In order for the provisioning script to have the proper permissions, you must run the `sudo` command as shown above. Using `sudo` with the `dek_provision.py` command will not work.

#### Clone the Repository

Clone the kit's repository to the provisioning system:

```Shell.bash
[Provisioning System] # git clone https://github.com/smart-edge-open/open-developer-experience-kits.git --branch=smart-edge-open-21.09 ~/dek
[Provisioning System] # cd ~/dek
```

#### Create the Installation Image

##### Run the Provisioning Script
The `dek_provision.py` script builds and runs the provisioning services and prepares the installation media.

##### Build and Run the Provisioning Services

To build and run the provisioning services in a single step, run the following command from the root directory of the
Developer Experience Kit repository:

```Shell.bash
[Provisioning System] # ./dek_provision.py --run-esp-for-usb-boot
```

Alternatively, to specify the Docker registry mirror to be used during the Developer Experience Kit deployment use the `--registry-mirror` option:
```Shell.bash
[Provisioning System] # ./dek_provision.py --registry-mirror=http://example.local:5000 --run-esp-for-usb-boot
```

The script will create an installation image in the `out` subdirectory of the current working directory.


##### Flash the Installation Image

To flash the installation image onto the flash drive, insert the drive into a USB port on the provisioning system and run the following command:

```Shell.bash
[Provisioning System] # ./esp/flashusb.sh --image ./out/SEO_DEK-efi.img --bios efi
```

The command should present an interactive menu allowing the selection of the destination device. You can also use the `--dev` option to explicitly specify the device.

#### Install the Image on the Target System

Begin the installation by inserting the flash drive into the target system. Reboot the system, and enter the BIOS to boot from the installation media.

##### Log Into the System After Reboot

The system will reboot as part of the installation process.

The login screen will display the system's IP address and the status of the experience kit deployment.
To log into the system, use `smartedge-open` as both the user name and password.

#### Check the Status of the Installation
When logging in using remote console or SSH, a message will be displayed that informs about status of the deployment, for example:
```Smart Edge Open Deployment Status: in progress```

Three statuses are possible:
- `in progress` - Deployment is in progress.
- `deployed` - Deployment was successful. The Developer Experience Kit cluster is ready.
- `failed` - An error occurred during the deployment.

Check the installation logs by running the following command:

```Shell.bash
[Provisioned System] $ sudo journalctl -xefu seo
```
Alternatively, you can inspect the deployment log found in `/opt/seo/logs`.

## Troubleshooting
Find information on resolving common installation problems in the [provisioning guide](/provisioning/provisioning.md#troubleshooting).

## Summary and Next Steps
In this guide, you created an Intel® Smart Edge Open edge node cluster capable of hosting edge applications. You can now install sample applications, or reference implementations downloaded from from the Intel® Developer Catalog
- Learn how to [onboard a sample application](/application-onboarding/application-onboarding-cmdline.md) to your cluster.
- Download and run [reference implementations from the Intel® Developer Catalog](https://www.intel.com/content/www/us/en/developer/tools/software-catalog/full-catalog.html?s=Newest&q=%22smart+edge+open%22)


