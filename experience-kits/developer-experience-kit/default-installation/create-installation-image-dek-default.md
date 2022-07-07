```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2022 Intel Corporation
```

# Create the Installation Image

Now that you have prepared the provisioning system, follow these steps to create the installation image. 

##  1. Disable Security Features for Default Installation

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

## 2. Configure the SR-IOV Network Operator

If you are installing the Developer Experience Kit to a target system that uses a different network adapter than the [validated NIC](https://github.com/smart-edge-open/docs/blob/main/release-notes/release-notes-se-open-DEK-21-12.md), you'll need to update the `custom.yml` file with the exact names of the target node's NIC interfaces. 

To do this, create a  `[cvl_sriov_nics]` section with a `[Debian]` dictionary.

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
## 3. Disable secure boot

Change the secure boot and trusted platform module in the `custom.yml`  to `false`:

```Shell.bash
#Secure boot and trusted media platform options.
bios: 
secure_boot: false
tpm: false
```

## 4. Build and Run the Provisioning Services

The `dek_provision.py` script invokes the Edge Software Provisioner to build and run the provisioning services and prepare the installation media.

To build and run the provisioning services in a single step, run the following command from the root directory of the
Developer Experience Kit repository on the provisioning system:

```Shell.bash
# sudo ./dek_provision.py --run-esp-for-usb-boot --config=custom.yml
```

Alternatively, you can specify the Docker registry mirror to be used during the Developer Experience Kit deployment by using the `--registry-mirror` option from the provisioning system:

```Shell.bash
# sudo ./dek_provision.py --registry-mirror=http://example.local:5000 --run-esp-for-usb-boot --config=custom.yml
```

The script creates an installation image in the `out` subdirectory of the current working directory.


## 5. Flash the Installation Image

To flash the installation image onto the flash drive, insert the drive into a USB port on the provisioning system and run the following command:

```Shell.bash
# ./esp/flashusb.sh --image ./out/SEO_DEK-efi.img --bios efi
```

The command presents an interactive menu allowing you to select the destination device. You can also use the `--dev` option to explicitly specify the device.

### Next

After creating the installation image, the final step is to install it on the target system.