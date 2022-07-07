```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2022 Intel Corporation
```
# Install the Intel® Smart Edge Open Developer Experience Kit

|      |
| :--: |
| [![Smart Edge Open Developer Experience Kit Workflow Diagram](../images/dek-workflow-diagram.png)](images/dek-workflow-diagram.png) |
| <b>The Intel® Smart Edge Open Developer Experience Kit installation workflow</b>

## System requirements

You will need two machines:

- **A provisioning system:** where you run the Edge Software Provisioner to build a bootable image of the experience kit.

- **A target system:** where you install the bootable image to create an edge cluster.

> **Note:** You must add the provisioning system's user account to `/etc/sudoers` for installation to succeed.

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
- Connection to the provisioning system

> **NOTE:** The provisioning process will install Ubuntu 20.04 LTS on the target machine. Any existing operating system will be overwritten.

## Install the Developer Experience Kit

There are two ways of installing the Developer Experience kit on your target system:

- **The default installation:** Installs the Developer Experience kit without security features enabled. 
- **The advanced installation:** Installs the Developer Experience kit with security features enabled. 


### Next

The next section explains the default installation process. 