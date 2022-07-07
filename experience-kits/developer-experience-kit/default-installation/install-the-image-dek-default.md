```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2022 Intel Corporation
```

# Install the image on the target system

Once you create the installation image, you can begin the installation by inserting the flash drive into the target system, rebooting the system, and entering the BIOS to boot from the installation media.

## Log into the system after reboot

The system will reboot as part of the installation process.

The login screen will display the system's IP address and the status of the experience kit deployment.

To log into the system, use `smartedge-open` as both the user name and password.

## Check the Status of the Installation

When logging in with a remote console or SSH, a message will display the status of the deployment. For example:

```Smart Edge Open Deployment Status: in progress```

There are three possible status messages:

- `in progress` - Deployment is in progress.
- `deployed` - Deployment was successful. The Developer Experience Kit cluster is ready.
- `failed` - An error occurred during the deployment.

Check the installation logs by running the following command on the target system:

```Shell.bash
$ sudo journalctl -xefu seo
```

You can find more details in the deployment log in `/opt/seo/logs`.

## Use the Smart Edge Open cluster

Now that you have created an Intel® Smart Edge Open edge node cluster capable of hosting edge applications, you can:

- [Onboard a sample application](/application-onboarding/application-onboarding-cmdline.md) to your cluster.
- Download and run [reference implementations from the Intel® Developer Catalog](https://www.intel.com/content/www/us/en/developer/tools/software-catalog/overview.html?s=ContentType&q=%22smart%20edge%20open%22)

### Next

To deploy the Developer Experience Kit with security features enabled, review the advanced installation instructions in the next section. 