```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```

# Intel® Smart Edge Open Developer Experience Kit -- Advanced Installation

This section outlines the advanced installation for the Intel® Smart Edge Open Developer Experience Kit. Follow this process to create an edge node with integrated platform security features including:

- Platform attestation using Intel® Security Libraries for Data Center (Intel® SecL - DC)
- Application security through Intel® Software Guard Extensions (Intel® SGX) and Intel® Software Guard Extensions Data Center Attestation Primitives (Intel® SGX DCAP)

When you follow the advanced installation, the Developer Experience Kit will have two clusters:

- Developer experience kit cluster in the cloud that hosts IsecL and SGX control plane services. These control plane services enable platform attestation and Secure enclave for Edge applications and services. 

- Developer experience kit cluster at the edge (typically on-premises) that hosts edge services and applications. 

The following diagram shows the architecture of these two clusters.

|      |
| :--: |
| [![Smart Edge Open Developer Experience Kit - Deployment Diagram](../../images/dek-deploy.png)](images/dek-deploy.png) |
|<b> Developer Experience Kit advanced installation architecture</b>| 


## Advanced installation component stack

This diagram shows the component stack of the developer experience kit deployed through advanced installation, including the additional security features.

|      |
| :--: |
| [![Smart Edge Open Developer Experience Kit - Edge Node Component Diagram](../../images/dek-node-component-diagram.png)](images/dek-node-component-diagram.png) |
| <b>Developer Experience Kit components of the edge and cloud clusters</b> |

The integrated security features also require you to deploy remote attestation services on an Amazon Web Services (AWS) EC2 instance as shown in the image below. 

|      |
| :--: |
| [![Smart Edge Open Developer Experience Kit - IsecL Controller and SGX DCAP Node Component Diagram](../../images/verification-node-component-diagram.png(images/verification-node-component-diagram.png) |
|<b> Remote attestation services deployed as a controller node on AWS</b> |

### Next

The next section explain how to set up the security features for the Intel® Smart Edge Open Developer Experience Kit. 


