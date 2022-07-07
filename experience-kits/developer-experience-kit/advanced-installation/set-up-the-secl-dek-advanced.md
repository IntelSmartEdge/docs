```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```

# Set up the Intel® SecL - DC Controller Node, Intel® SGX Provisioning Certificate Caching Service (PCCS), and KMRA app-hsm(Key server) Service

The following steps are required if you are installing the Developer Experience Kit with security features enabled for platform attestation with Intel® SecL - DC and application security with Intel® SGX.
- Make password less from Ansible machine to AWS instance by copying ssh public using ssh-copy-id (Ex. ssh-copy-id -i ~/.ssh/<my_key> <user-name>@<aws-instance-ip>)
- Update following things in `inventory/default/group_vars/all/10-default.yml`
  - Comment out proxy settings(http_proxy, https_proxy, ftp_proxy and all_proxy) and set  no_proxy to ""
  - Add host name of AWS instance public IP in `/etc/hosts` for both IPv4 and IPv6.
  - Set `pccs_user_password`. Make sure password should contain at least 12 characters.(This setting is for application security with Intel® SGX feature).
  - Set `kmra_apphsm_ip` and `sgx_pccs_ip` to AWS instance public IP. These parameters are required for KMRA feature.
  - Set `pccs_api_key` which is required by PCCS to access Intel® PCS servers. To get API key follow the instructions [here](/components/security/application-security-using-sgx.md#How-to-subscribe-to-Intel-PCS-Service)
- Update following things in `inventory.yml`
  - Set `deployment` to `verification_controller`.
  - Provide public IP address of AWS instance where the deployment is expected to install the IsecL controller - the same one for controller and edge node group.
  - Provide the username.

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

- If you are enabling platform attestation with Intel® SecL - DC, update `deployments/verification_controller/all.yml` with following changes
  1. IP addresses of edge nodes(where trust agents run) in `isecl_ta_san_list`
  2. Make sure `platform_attestation_controller` is set to `true`
  3. Set `isecl_control_plane_ip` to AWS instance public ip in `inventory/default/group_vars/all/10-default.yml`
- If you are enabling Intel® SGX, set `pccs_enable` to `true` in `deployments/verification_controller/all.yml`
- If you are enabling Secure Key management(KMRA) feature, make sure `kmra_enable` is set to `true` in `deployments/verification_controller/all.yml`. SGX feature is prerequisite for KMRA feature, so SGX feature should be enabled in verification control cluster as well as edge cluster.

Developer Experience Kit users can disable certain features to be automatically provisioned and deployed during the setup. This is made available to the users using the "Exclude List" feature. As an example if user wants to exclude Platform Attestation with IsecL or/and SGX then following steps needs to be followed: