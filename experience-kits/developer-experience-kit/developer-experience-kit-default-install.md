```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2022 Intel Corporation
```

# Intel® Smart Edge Open Developer Experience Kit Default Installation

This section outlines the default installation process for the Developer Experience Kit. Default installation lets you quickly deploy an Intel® Smart Edge Open edge cluster. This process installs the Developer Experience kit without security features enabled. Use the [advanced installation instructions](experience-kits/developer-experience-kit-advanced-install.md) in the next section if you want to install the Developer Experience Kit with security features enabled.

Follow these instructions to create an edge node cluster capable of hosting edge applications. You can then install reference implementations or onboard edge applications to the cluster. 

## Prepare the Provisioning System 

Begin by installing the following software to the provisioning system:

- Git
- Docker
- Docker Compose
- Python

> **Note:** You must be logged in as a root user on the provisioning system to complete the following steps. Run the following command on the provisioning system to login as a root user:
  ```Shell.bash
    $ sudo su -
  ```

### 1. Install Git

Use the following command to install Git to the target system:

```Shell.bash
# apt-get install git
```

### 2. Install Docker and Docker Compose

Install Docker, configure it to use a proxy server, and install Docker Compose. 

i. Install Docker from the Docker Repository

    Follow the installation instructions in the [Docker repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository). Installation by package manager is not supported and will result in errors.

ii. Configure Docker to Use a Proxy Server
    Create or edit the Docker configuration file
```Shell.bash
#mkdir ~/.docker
#vi ~/.docker//config.json
{
 "proxies":
  {
    "default":
    {
      "httpProxy":"http://proxy.example.com:80"
      "httpsProxy": "http://proxy.example.com:80",
      "noProxy": "127.0.0.0/8"
    }
  }
}
```

iii. Set your environment variables. Replace `myproxy.hostname` in the example with the information for your own proxy.

```Shell.bash
# mkdir -p /etc/systemd/system/docker.service.d
# vi /etc/systemd/system/docker.service.d/proxy.conf
[Service]
Environment="HTTP_PROXY=http://myproxy.hostname:8080"
Environment="HTTPS_PROXY=https://myproxy.hostname:8080/"
Environment="NO_PROXY="localhost,127.0.0.1,::1"
```

iv. Reload the file and restart the Docker service

```Shell.bash
# systemctl daemon-reload
# systemctl restart docker.service
```

v. Install Docker Compose

```Shell.bash
#apt install docker-compose
```

### 3. Install Python3 and Related Libraries

Run the following commands to install Python3 and related libraries:

```Shell.bash
# apt update
# apt install python3
# apt install python3-pip
# pip3 install PyYAML
# pip3 install fqdn
```

### 4. Clone the Developer Experience Kit Repository

Clone the [Developer Experience Kit repo](https://github.com/smart-edge-open/open-developer-experience-kits) to the provisioning system:

```Shell.bash
# git clone https://github.com/smart-edge-open/open-developer-experience-kits.git --branch=smart-edge-open-21.12 ~/dek
# cd ~/dek
```

### 5. Generate the Configuration File
Run the following command to generate the `custom.yml` file. It is recommended you generate a new congfiguraion file instead of using the default `default_config.yml` file provided.
```Shell.bash
[Provisioning System] # sudo ./dek_provision.py --init-config > custom.yml
```
### 6. Create the Installation Image
#### 7. Disable Security Features for Default Installation
To install the edge node without security features enabled, edit the following parameters in the `group_vars:` section of the `custom.yml` file:
```Shell.bash
# vi custom.yml
[group_vars:]
[groups:]
[all:]
sgx_enabled: false
platform_attestation_node: false
```
To deploy the Developer Experience Kit with security features enabled, see the [advanced installation instructions](/experience-kits/developer-experience-kit-advanced-install.md).

#### 8. Configure the SR-IOV Network Operator
If you are installing the Developer Experience Kit to a target system that uses a different network adapter than the [validated NIC](https://github.com/smart-edge-open/docs/blob/main/release-notes/release-notes-se-open-DEK-21-12.md), you'll need to update the `custom.yml` file with the exact names of the target node's NIC interfaces. To do this, create a  `[cvl_sriov_nics]` section with a `[Debian]` dictionary.

```Shell.bash
# vi custom.yml
[group_vars:]
[groups:]
[all:]
[cvl_sriov_nics]
[Debian]
c0p0: "<>"
c0p1: "<>"
c1p0: "<>"
c1p1: "<>"
```
#### 9. Disable secure boot
Turn off the flags for secure boot and trusted platform module in the `custom.yml` file:
```Shell.bash
#Secure boot and trusted media platform options.
bios: 
secure_boot: false
tpm: false
```

#### 10. Build and Run the Provisioning Services

The `dek_provision.py` script invokes the Edge Software Provisioner to build and run the provisioning services and prepare the installation media.

To build and run the provisioning services in a single step, run the following command from the root directory of the
Developer Experience Kit repository on the provisioning system:

```Shell.bash
# sudo ./dek_provision.py --run-esp-for-usb-boot --config=custom.yml
```

Alternatively, to specify the Docker registry mirror to be used during the Developer Experience Kit deployment use the `--registry-mirror` option from the provisioning system:
```Shell.bash
# sudo ./dek_provision.py --registry-mirror=http://example.local:5000 --run-esp-for-usb-boot --config=custom.yml
```

The script will create an installation image in the `out` subdirectory of the current working directory.


#### 11. Flash the Installation Image

To flash the installation image onto the flash drive, insert the drive into a USB port on the provisioning system and run the following command:

```Shell.bash
# ./esp/flashusb.sh --image ./out/SEO_DEK-efi.img --bios efi
```

The command will present an interactive menu allowing the selection of the destination device. You can also use the `--dev` option to explicitly specify the device.

## 12. Install the Image on the Target System

Begin the installation by inserting the flash drive into the target system. Reboot the system, and enter the BIOS to boot from the installation media.

#### 13. Log Into the System After Reboot

The system will reboot as part of the installation process.

The login screen will display the system's IP address and the status of the experience kit deployment.
To log into the system, use `smartedge-open` as both the user name and password.

#### 14. Check the Status of the Installation

When logging in using remote console or SSH, a message will be displayed with the status of the deployment. For example:
```Smart Edge Open Deployment Status: in progress```

One of three statuses will be displayed:
- `in progress` - Deployment is in progress.
- `deployed` - Deployment was successful. The Developer Experience Kit cluster is ready.
- `failed` - An error occurred during the deployment.

Check the installation logs by running the following command on the target system:

```Shell.bash
$ sudo journalctl -xefu seo
```
You can find more details in the deployment log found in `/opt/seo/logs`.

## Provisioning guide and troubleshooting

Find detailed information on provisioning process and on resolving common installation problems in the [provisioning guide](/experience-kits/provisioning/provisioning.md).

## Summary and Next Steps

In this guide, you created an Intel® Smart Edge Open edge node cluster capable of hosting edge applications. You can now install sample applications, or reference implementations downloaded from from the Intel® Developer Catalog
- Learn how to [onboard a sample application](/application-onboarding/application-onboarding-cmdline.md) to your cluster.
- Download and run [reference implementations from the Intel® Developer Catalog](https://www.intel.com/content/www/us/en/developer/tools/software-catalog/overview.html?s=ContentType&q=%22smart%20edge%20open%22)
