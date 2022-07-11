```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2022 Intel Corporation
```
# Install the Intel® Smart Edge Open Developer Experience Kit - requirements

There are two ways to install the Developer Experience kit on your target system:

- **The default installation:** Installs the Developer Experience kit without security features enabled. 
- **The advanced installation:** Installs the Developer Experience kit with security features enabled. 

This page includes the requirements to complete both a default installation and an advanced installation. 

|      |
| :--: |
| [![Smart Edge Open Developer Experience Kit Workflow Diagram](../images/dek-workflow-diagram.png)](images/dek-workflow-diagram.png) |
| <b>The Intel® Smart Edge Open Developer Experience Kit installation workflow</b>

## System requirements

You will need:

- **A provisioning system:** where you run the Edge Software Provisioner to build a bootable image of the experience kit.

- **A target system:** where you install the bootable image to create an edge cluster.

- **AWS EC2 instance:** Only required if you need security features enabled and you are completing the advanced installation. 

> **Note:** You must add the provisioning system's user account to `/etc/sudoers` for installation to succeed.

If you choose to enable security features in your installation (advanced installation), you will also need an AWS EC2 t2.medium instance to host the controller node.

### Provisioning system 

The provisioning system must meet the following specifications:

- At least 4 GB RAM 
- At least 20 GB hard drive
- USB flash drive
- Ubuntu 20.04 LTS
- Internet access
  
### Target system 

Below are the minimum requirements for running the Developer Experience Kit. See the full specs for the [supported hardware](https://github.com/smart-edge-open/docs/blob/main/release-notes/release-notes-se-open-DEK-21-12.md). 

- A server with two 3rd Generation Intel® Xeon® Scalable Processors
- At least 32 GB RAM 
- Two SATA SSDs
- Two NICs 
    > **NOTE:** This configuration was validated using two Intel Corporation Ethernet Controller E810-C for SFP (rev 02) NICs.
- Connection to the provisioning system

> **NOTE:** The provisioning process will install Ubuntu 20.04 LTS on the target machine. Any existing operating system will be overwritten.

### AWS EC2 Instance (Advanced installation only)

Advanced installations that enable either platform attestation using Intel® SecL - DL or application security using Intel® SGX will also require an AWS EC2 t2.medium instance with the following system requirements:
   - Two vCPUs
   - 4 GB RAM
   - 100 GB disk space
   - Ubuntu 20.04 LTS
- A Linux system from which deployment of the controller node is initiated

The Developer Experience Kit uses the [Edge Software Provisioner (ESP)](https://github.com/intel/Edge-Software-Provisioner), which automates the process of provisioning bare-metal or virtual machines with an operating system and software stack. Intel® Smart Edge Open provides a fork of the [Ubuntu OS ESP Profile](https://github.com/intel/rni-profile-base-ubuntu) tailored for its specific needs.


### Next

The next section explains how to prepare the provisioning system for installation.