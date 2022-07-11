```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```

# Configure security features for the Intel® Smart Edge Open Developer Experience Kit

Follow these steps to begin installing the Intel® Smart Edge Developer Experience Kit with security features enabled for platform attestation with Intel® SecL - DC and application security with Intel® SGX:

1.  Make password-less from the Ansible machine to the AWS instance by copying the ssh public using ssh-copy-id (Ex. `ssh-copy-id -i ~/.ssh/<my_key> <user-name>@<aws-instance-ip>`).

2. Update following items in the  `inventory/default/group_vars/all/10-default.yml`:
    - Comment out proxy settings(`http_proxy`, `https_proxy`, `ftp_proxy`, and `all_proxy`) and set  `no_proxy` to `""`.
    - Add hostname of the AWS instance public IP in `/etc/hosts` for both IPv4 and IPv6.
    - Set `pccs_user_password`. Password should contain at least 12 characters.(This setting is for application security with Intel® SGX features).
    - Set `kmra_apphsm_ip` and `sgx_pccs_ip` to the AWS instance public IP. These parameters are required for the KMRA feature.
    - Set `pccs_api_key`. This is required by PCCS to access Intel® PCS servers. Follow [these instructions](/components/security/application-security-using-sgx.md#How-to-subscribe-to-Intel-PCS-Service) to get the API key.

3. Update following items in `inventory.yml`
    - Set `deployment` to `verification_controller`.
    - Provide public IP address of AWS instance where the deployment is expected to install the IsecL controller - the same one for controller and edge node group.
    - Provide the username.

This is an example of a completed `inventory.yml` file.

```Shell.bash
groups_var: 
  groups:
    all:
      vars:
        cluster_name: verification_controller_cluster        
        deployment: verification_controller
        single_node_deployment: true
        limit: 
    controller_group:
      hosts:
        controller:
          ansible_host: <IP-address>
          ansible_user: <user-name>
    edgenode_group:
      hosts:
        node01:
          ansible_host: <IP-address>
          ansible_user: <user-name>
```    
## Enable platform attestation with Intel® SecL - DC

Make the following changes to `deployments/verification_controller/all.yml` to enable platform attestation with Intel® SecL - DC.
  - Add the IP addresses of edge nodes(where trust agents run) to `isecl_ta_san_list`
  - Set `platform_attestation_controller` to `true`
  - If you are enabling Intel® SGX, set `pccs_enable` to `true
  - If you are enabling the Secure Key management(KMRA) feature, set `kmra_enable` to `true`. The SGX feature is prerequisite for KMRA feature, so SGX feature should be enabled in verification control cluster as well as edge cluster.

You will also need to set `isecl_control_plane_ip` to the AWS instance public IP in `inventory/default/group_vars/all/10-default.yml`.

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

## Optional: Disable Unused Security Features

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

You can also disable SR-IOV Network Operator configuration by setting the `sriov_network_operator_configure_enable` flag to `false` in file `custom.yml`.

For more information, see [SR-IOV Network Operator Configuration](https://github.com/smart-edge-open/docs/blob/main/components/networking/sriov-network-operator.md#configuration)

```Shell.bash
groups_var: 
  groups:
    all:
      vars:
        sriov_network_operator_configure_enable: false
```

## Optional: Update NIC Settings

If you are installing the Developer Experience Kit to a system that uses a different network adapter than the [validated NIC](#target-system-requirements), update the interface names in `custom.yaml`. Otherwise, installation will fail. 

For more information, see [SR-IOV Network Operator Configuration](https://github.com/smart-edge-open/docs/blob/main/components/networking/sriov-network-operator.md#configuration)

### Next

Now that you have properly configured your security features, you can create and flash the installation image. 