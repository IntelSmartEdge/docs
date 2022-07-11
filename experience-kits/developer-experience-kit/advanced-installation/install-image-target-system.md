```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```
# Install the Image on the Target System

## Optional: Enable Security Support in the BIOS

Update the provisioning system's BIOS settings to enable support for any security features you have installed: 
- If you have installed Intel® SGX, refer to this guide for [enabling Intel® SGX in the BIOS](/components/security/application-security-using-sgx.md#enable-intel-sgx-in-bios)

## Boot from the Flash Drive

Insert the flash drive into the target system. Reboot the system, and enter the BIOS to boot from the installation media.

## Log Into the System After Reboot

The system will reboot as part of the installation process.

The login screen will display the system's IP address and the status of the experience kit deployment.
To log into the system, use `smartedge-open` as both the user name and password.

## Check the Status of the Installation

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

## Provisioning guide and troubleshooting

Find detailed information on provisioning process and on resolving common installation problems in the [provisioning guide](/experience-kits/provisioning/provisioning.md).

## 4. Use the Smart Edge Open cluster

Now that you have created an Intel® Smart Edge Open edge node cluster capable of hosting edge applications, you can:

- [Onboard a sample application](/application-onboarding/application-onboarding-cmdline.md) to your cluster.
- Download and run [reference implementations from the Intel® Developer Catalog](https://www.intel.com/content/www/us/en/developer/tools/software-catalog/overview.html?s=ContentType&q=%22smart%20edge%20open%22)
- Learn how to run sample SGX openVINO application which uses Intel SGX for running secure workloads inside an enclave(Only if SGX feature is enabled in edge node) [Intel SGX OpenVINO sample application](https://github.com/smart-edge-open/edgeapps/blob/main/applications/sgx/openvino-ssd/README.md)
- Learn how to run the KMRA sample NGINX application which uses Intel SGX for managing keys securely(Only if KMRA is enabled feature is enabled) - [KMRA Reference application guide](https://github.com/smart-edge-open/edgeapps/tree/main/applications/sgx/kmra#readme)
