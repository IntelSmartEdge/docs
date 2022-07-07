```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```



## 1. Run the Deployment Script

The `deploy.sh` script installs all required packages and deploys the single node cluster on.

```Shell.bash
# ./deploy.sh
```

## 2. Generate the Configuration File: 

```
[Provisioning System] # ./dek_provision.py --init-config > custom.yml
```

You can also choose to disable automatic provisioning and deployment of certain features during the setup by using the "Exclude List" feature. The following example excludes Platform Attestation with IsecL or/and SGX. 

## 3. Optional: Disable Unused Security Features

By default, Intel® SecL - DC and Intel® SGX will be enabled in the edge node. If you are not using these security features, you can disable them.
- To disable Intel® SGX support, set the `sgx_enabled` flag to `false` in file `custom.yml`
- To disable Intel® SecL - DC platform attestation support, set the `platform_attestation_node` flag to `false` in file `custom.yaml`

```Shell.bash
groups_var: 
  groups:
    all:
      vars:
        sgx_enabled: false
        platform_attestation_node: false 
```       

#### Optional: Disable SR-IOV Network Operator Configuration

- Set the `sriov_network_operator_configure_enable` flag to `false` in file `custom.yml`
For more information, see [SR-IOV Network Operator Configuration](https://github.com/smart-edge-open/docs/blob/main/components/networking/sriov-network-operator.md#configuration)

```Shell.bash
groups_var: 
  groups:
    all:
      vars:
        sriov_network_operator_configure_enable: false
```

#### Optional: Update NIC Settings

If you are installing the Developer Experience Kit to a system that uses a different network adapter than the [validated NIC](#target-system-requirements), update the interface names in `custom.yaml`. Otherwise, installation will fail. 

For more information, see [SR-IOV Network Operator Configuration](https://github.com/smart-edge-open/docs/blob/main/components/networking/sriov-network-operator.md#configuration)