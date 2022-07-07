```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2022 Intel Corporation
```

# Intel® Smart Edge Open Developer Experience Kit Default Installation

This section outlines the default installation process for the Developer Experience Kit. Default installation lets you quickly deploy an Intel® Smart Edge Open edge cluster. Follow these instructions to create an edge node cluster capable of hosting edge applications. You can then install reference implementations or onboard edge applications to the cluster.  This process installs the Developer Experience kit without security features enabled. 

The default installation enables you to install and instantiate an Intel® Smart Edge Open edge cluster on a single node. This deployment consists of a Kubernetes Control Plane and an Edge Node. Once the cluster is installed, you will be able to run edge applications, including [reference implementations](reference-implementations.md) built on Intel® Smart Edge Open, and become familiar with operating a stand-alone edge node.


|      |
| :--: |
| [![Smart Edge Open Developer Experience Kit - Edge Node Component Diagram](../../images/dek-component-diagram.png)](images/dek-component-diagram.png) |
| <b>Intel® Smart Edge Open Developer Experience Kit default installation building blocks</b>|

The Developer Experience Kit uses Intel's [Edge Software Provisioner (ESP)](https://github.com/intel/Edge-Software-Provisioner) to streamline deployment of the edge node cluster. The ESP automates the provisioning of the operating system and software stack used by the hardware that hosts the Intel® Smart Edge Open edge cluster. 


Use the [advanced installation instructions](experience-kits/developer-experience-kit-advanced-install.md) in the next section if you want to install the Developer Experience Kit with security features enabled.

The default installation process has 3 stages to:

- Prepare the Provisioning system

- Create the installation image

- Install the image on the target system

### Next

The next section explains how to prepare the provisioning system. 


## Provisioning guide and troubleshooting

Find detailed information on provisioning process and on resolving common installation problems in the [provisioning guide](/experience-kits/provisioning/provisioning.md).

