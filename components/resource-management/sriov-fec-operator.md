```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```
# SR-IOV FEC Operator

- [SR-IOV FEC Operator](#sr-iov-fec-operator)
  - [Overview](#overview)
    - [Overview of SR-IOV Device Plugin](#overview-of-sr-iov-device-plugin)
    - [Overview of SR-IOV FEC Operator](#overview-of-sr-iov-fec-operator)
  - [SR-IOV Network Operator configuration and usage](#sr-iov-network-operator-configuration-and-usage)
    - [Configuration](#configuration)
      - [SR-IOV FEC Operator configuration variables](#sr-iov-fec-operator-configuration-variables)
      - [SR-IOV FEC Cluster Config Policy](#sr-iov-fec-cluster-config-policy)
    - [Usage](#usage)
  - [Limitations](#limitations)
  - [Reference](#reference)

## Overview
Edge deployments consist of various types of Network Functions. Some of those function such as these related to the Physical layer (or L1) of the networking stack are very compute intensive and can benefit from offloading the processes from CPU to dedicated accelerator. One such process is the FEC (Forward Error Correction) of the DU (Distributed Unit). Cloud-native solutions such as Kubernetes* typically do not provide support for these kind of accelerators to be treated as a native resource out of the box - hence the K8s applications cannot consume the resource if it is not available. Additionally on a platform level, these kind of accelerators are often complex in configuration in order to make them usable by applications. To address this an Operator framework is provided which allows for configuration of said resources in a Cloud Native way, which in combination with K8s CRDs (Custom Resource Definitions) enables to create/treat these accelerators as K8s custom resources consumable by applications.

### Overview of SR-IOV Device Plugin
The Intel SR-IOV Network device plugin discovers and exposes SR-IOV resources as consumable extended resources in Kubernetes. This works with SR-IOV VFs in both Kernel drivers and DPDK drivers. The device plugin is capable of handling accelerator resources bound to DPDK driver. It allows for a DPDK bound ACC100 FEC VF to be discovered and allocatable as a resource within K8s. The device plugin is one of the components deployed by the operator.

### Overview of SR-IOV FEC Operator
The role of the Intel Wireless FEC Accelerator Operator is to orchestrate and manage the resources/devices exposed by a range of Intel's vRAN FEC acceleration devices/hardware within a K8s cluster. The operator is a state machine which will configure the resources and then monitor them and act autonomously based on the user interaction. The operator design of the Intel Wireless FEC Accelerator Operator supports the [Intel vRAN dedicated accelerator ACC100](https://builders.intel.com/docs/networkbuilders/intel-vran-dedicated-accelerator-acc100-product-brief.pdf). For more information on the [Intel Wireless FEC Accelerator Operator see the link in the reference section](#reference)

## SR-IOV Network Operator configuration and usage

To deploy the SR-IOV FEC Operator to the Intel® Smart Edge Open cluster, `sriov_fec_operator_enable: True` and `sriov_fec_operator_configure_enable: True` must be set in the `inventory/default/group_vars/all/10-default.yml` file (or in an alternative deployment `all.yml` file under `./deployments/<deployment_name>` which will take precedence). The flags enable the installation of the Operator and configuration of the device during provisioning respectively. This component is disabled by default in [Developer Experience Kit](../../experience-kits/developer-experience-kit.md). This will perform Operator install and label every Edge Node containing the Intel vRAN dedicated accelerator ACC100 accordingly. The SR-IOV Device plugin will be deployed on labeled node once the operator is installed.
SR-IOV FEC Operator is deployed using Makefiles and it requires additional packages to be installed, these packages are installed by Intel® Smart Edge Open together with Operator deployment.
Images for Operator are built from source and pushed to the local Harbour registry provided by the Smart Edge deployment.

> **NOTE:** The `igb_uio` driver needs to be available on the platform for the operator to work - see [limitations](#limitations) for steps enabling the deployment of the driver.

### Configuration

The following section describes the configuration steps required to configure the FEC devices during the provisioning of Smart Edge Open.

#### SR-IOV FEC Operator configuration variables

In the file:  `inventory/default/group_vars/all/10-default.yml` (or an alternative deployment `all.yml` file under `./deployments/<deployment_name>`) provide configuration for the ACC100 accelerator (the default configuration can be found under `roles/baseline_ansible/kubernetes/operator/sriov_fec_operator/configure/defaults/main.yml`):

```yaml
sriov_fec_cluster_config:
  name: "config1"                             #Name of the specific config.
  cluster_config_name: "default-sriov-cc"     #Name of the cluster config.
  priority: 1                                 #Priority of deployment (lowe number higher priority).
  drainskip: true                             #Allows for skipping the draining of the node after config application.
  selected_node: "node_name"                  #(Optional) field that can be used to target only specific node.
  pf_driver: "igb_uio"                        #The PF driver to be used - on Centos 7 igb_uio is needed to create VFs for the device.
  vf_driver: "vfio-pci"                       #The VF driver to be used.
  vf_amount: 3                                #The amount of VFs to be created for the device.
  bbdevconfig:
    pf_mode: false                            #The mode in which the ACC100 accelerator will be programmed, it is expected that VFs will be used and this is set to false.
    num_vf_bundles: 3                         #Number of VF bundles this should correspond to the vf_amount field.
    max_queue_size: 1024                      #Max queue size this field is not expected to change in most deployments.
    ul4g_num_queue_groups: 0                  #Number of 4G Uplink queue groups - there is in total 8 queue groups that can be distributed between 4G/5G Uplink/Downlink
    ul4g_num_aqs_per_groups: 16               #Number of aqs per group - not expected to change for most deployments
    ul4g_aq_depth_log2: 4                     #Log depth
    dl4g_num_queue_groups: 0                  #Number of 4G Downlink queue groups - there is in total 8 queue groups that can be distributed between 4G/5G Uplink/Downlink
    dl4g_num_aqs_per_groups: 16               #Number of aqs per group - not expected to change for most deployments
    dl4g_aq_depth_log2: 4                     #Log depth
    ul5g_num_queue_groups: 4                  #Number of 5G Uplink queue groups - there is in total 8 queue groups that can be distributed between 4G/5G Uplink/Downlink - here 4 queues are used for 5G Uplink
    ul5g_num_aqs_per_groups: 16               #Number of aqs per group - not expected to change for most deployments
    ul5g_aq_depth_log2: 4                     #Log depth
    dl5g_num_queue_groups: 4                  #Number of 5G Downlink queue groups - there is in total 8 queue groups that can be distributed between 4G/5G Uplink/Downlink - here 4 queues are used for 5G Downlink
    dl5g_num_aqs_per_groups: 16               #Number of aqs per group - not expected to change for most deployments
    dl5g_aq_depth_log2: 4                     #Log depth
```

The above variables are used to template the configuration that is being deployed during provisioning of the platform.

SR-IOV FEC Operator provides custom resource to configure SR-IOV FEC devices it can be used to re-configure the accelerators after the deployment.

#### SR-IOV FEC Cluster Config Policy

Specifies SR-IOV FEC configuration by e.g. creating VFs from given selectors provided by the user in the configuration (PCI address selector and hostname selector are optional, if not provided all discovered devices on all nodes within cluster will be configured). To apply resource to the Operator just simply use command: `kubectl apply -f sample-policy.yml`.
Sample Policy can look like:

```yaml
apiVersion: sriovfec.intel.com/v2
kind: SriovFecClusterConfig
metadata:
  name: config
spec:
  priority: 1
  nodeSelector:
    kubernetes.io/hostname: selected_node_name # Optional
  acceleratorSelector:
    pciAddress: 0000:af:00.0                   # Optional
  physicalFunction:
    pfDriver: "igb_uio"                        # PF driver name 
    vfDriver: "vfio-pci"                       # VF driver name
    vfAmount: 3                                # Amount of VFs to be created (up to 16)
    bbDevConfig:
      acc100:
        # Programming mode: 0 = VF Programming, 1 = PF Programming
        pfMode: false
        numVfBundles: 3                        # numVfBundles needs to be same as vfAmount
        maxQueueSize: 1024
        uplink4G:
          numQueueGroups: 0
          numAqsPerGroups: 16
          aqDepthLog2: 4
        downlink4G:
          numQueueGroups: 0
          numAqsPerGroups: 16
          aqDepthLog2: 4
        uplink5G:
          numQueueGroups: 4
          numAqsPerGroups: 16
          aqDepthLog2: 4
        downlink5G:
          numQueueGroups: 4
          numAqsPerGroups: 16
          aqDepthLog2: 4
```

> **NOTE:** After applying above sample configuration, Operator will create 3 VFs from given accelerator on specified node, bind them to drivers, program the FEC device as per the "bbDevConfig" provided and make the VFs as allocatable resources.

To display the allocatable resources run:

```yaml
# kubectl get node <node_name> -o json | jq '.status.allocatable'
{
  "cpu": "95500m",
  "ephemeral-storage": "898540920981",
  "hugepages-1Gi": "30Gi",
  "intel.com/intel_fec_acc100": "3", # The FEC VF as allocatable resource
  "memory": "115600160Ki",
  "pods": "250"
}
```

### Usage
To create a pod with an attached SR-IOV FEC device, request access to the SR-IOV FEC capable device (`intel.com/intel_fec_acc100`):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: samplepod
spec:
  containers:
  - name: samplecent
    image: centos/tools
    resources:
      requests:
        intel.com/intel_sriov_fec_acc100: "1"
      limits:
        intel.com/intel_sriov_fec_acc100: "1"
    command: ["sleep", "infinity"]
```

To verify that the additional interface was configured, run following command inside the created pod, the output should look similar to the following:

```shell
# printenv | grep INTEL_FEC
PCIDEVICE_INTEL_COM_INTEL_FEC_ACC100=0000:b0:00.0
```

> **NOTE**: The `0000:b0:00.0` is the device available within the pod.

## Limitations

There is an expectation that the PF and VF drivers to be used for the handling of the device are provided by platform.
This is the case with Smart Edge Open platform, the `igb_uio` is provided by enabling appropriate role with the `install_userspace_drivers_enable: true` flag in `inventory/default/group_vars/all/10-default.yml` (or alternative `all.yml` file if deploying specific deployment under `./deployments/<deployment name>`).

## Reference

For further details:

- [SR-IOV FEC Operator documentation](https://github.com/smart-edge-open/openshift-operator/blob/main/spec/openshift-sriov-fec-operator.md)
- [Intel vRAN dedicated accelerator ACC100 product brief](https://builders.intel.com/docs/networkbuilders/intel-vran-dedicated-accelerator-acc100-product-brief.pdf)
