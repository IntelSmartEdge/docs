```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```

# Intel® Smart Edge Open Provisioning Process

* [Overview](#overview)
  * [Provisioning Commands](#provisioning-commands)
* [Provisioning System Requirements](#provisioning-system-requirements)
  * [Software Requirements](#software-requirements)
  * [Software Installation Instructions](#software-installation-instructions)
    * [Install Python Dependencies](#install-python-dependencies)
* [Provisioning Process Scenarios](#provisioning-process-scenarios)
  * [Default Provisioning Scenario](#default-provisioning-scenario)
    * [Repository Cloning](#default-repository-cloning)
    * [Configuration](#default-configuration)
    * [Artifacts Building](#default-artifacts-building)
    * [Services Start-Up](#default-services-start-up)
    * [Installation Media Flashing](#default-installation-media-flashing)
    * [System Installation](#default-system-installation)
    * [Services Shut Down](#default-services-shut-down)
  * [Custom Provisioning Scenario](#custom-provisioning-scenario)
    * [Configuration](#custom-configuration)
    * [Artifacts Building](#custom-artifacts-building)
* [Provisioning Configuration](#provisioning-configuration)
  * [Configuration Methods](#configuration-methods)
  * [Command Line Arguments](#command-line-arguments)
  * [Configuration File Generation](#configuration-file-generation)
  * [Configuration File Summary](#configuration-file-summary)
  * [Experience Kit Configuration](#experience-kit-configuration)
  * [Operating System Account](#operating-system-account)
  * [Git Credentials](#git-credentials)
  * [Docker Pull Rate Limit](#docker-pull-rate-limit)
  * [Docker registry Mirror](#docker-registry-mirror)
  * [Docker Hub Credentials](#docker-hub-credentials)
  * [PXE](#pxe)
  * [Secure Boot and TPM](#secure-boot-and-tpm)
  * [BMC](#bmc)
  * [Hosts Configuration](#hosts-configuration) 
  * [Machine Hostname Configuration](#machine-hostname-configuration)
* [Troubleshooting](#troubleshooting)

## Overview

The Intel® Smart Edge Open automated provisioning process relies on the [Intel® Edge Software
Provisioner](https://github.com/intel/Edge-Software-Provisioner) (ESP). It is responsible for the operating system
installation and deployment of an Intel® Smart Edge Open cluster.

The process relies on dedicated [provisioning commands](#provisioning-commands) distributed within each experience kit
repository.

### Provisioning Commands

The Developer Experience Kit provides the `dek_provision.py` command-line utility, using the
Intel® Edge Software Provisioner toolchain to deliver a smooth installation experience.

Other experience kits provide alternatively named provisioning commands, but the name and some configuration defaults
are the only difference between them and the `dek_provision.py` command. The information included in this document also
applies to these alternative commands.

For simplicity, the following parts of this document will always assume that you are provisioning the Developer
Experience Kit and using the following two commands delivered with it:
* `dek_provision.py` &ndash; the main provisioning command,
* `dek_flash.py` &ndash; the utility command used to [flash the installation media](#installation-media-flashing).

These commands (or their equivalents) reside in the root directory of the chosen experience kit repository.

Keep in mind that when you use a different experience kit, the names of these commands will differ, but their behavior
will stay the same.

## Provisioning System Requirements

The provisioning process requires a temporary provisioning system operating on a separate Ubuntu machine. This machine
can be a physical or a virtual machine. The system must be routable from the subnet on which the provisioned machines
are supposed to work.

### Software Requirements

* Ubuntu 20.04
* Docker 18.09.3 or newer
* Docker-compose v1.23.2 or newer
* Python v3.6 or newer
* Pip 20.0 or newer
* Bash v4.3.48 or newer
* Git v2.25.1 or newer

### Software Installation Instructions

You should perform the following steps on a clean provisioning system to prepare it for the provisioning process
execution:

1. [Install Docker](#install-docker)
2. [Install Docker Compose](#install-docker-compose)
3. [Install Git](#install-git)
4. [Install Python dependencies](#install-python-dependencies)

#### Install Docker

Follow the Docker installation guide on the official website and install the tool:
[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/).

#### Install Docker Compose

##### Install Docker Compose
Install the Docker Compose tool according to the official instruction:
[Install Docker Compose](https://docs.docker.com/compose/install/).

##### Install Docker Compose standalone version
Please also install the standalone version of Docker Compose according to the instruction on the website:
[Install docker-compose standalone](https://docs.docker.com/compose/install/compose-plugin/#install-the-plugin-manually).

Verify that the `docker-compose` command is working correctly
```Shell.bash
[Provisioning System] $ docker-compose --version
Docker Compose version v2.6.0
```
If the command output looks like the following after installation of Docker Compose from the official site:
```Shell.bash
[Provisioning System] $ docker-compose --version
Command 'docker-compose' not found, but can be installed with:

apt install docker-compose
```
Do not follow the `apt install` suggestion. Check the [official instruction](https://docs.docker.com/compose/install/compose-plugin/#install-the-plugin-manually). Typically, executing the three simple steps starting after the "Compose standalone" note is enough to install it effectively.

#### Install Git

Install `Git` tool according to the official instruction:
[Install Git](https://git-scm.com/download/linux).

#### Install Python Dependencies

##### Install Pip

The Python dependencies installation process relies on the `pip` utility, which may not be installed on a Ubuntu system
by default. To check if the `pip` program is available, use the `pip --version` command. If its output looks like the
following, the `pip` program is ready to be used:

```sh
[Provisioning System] $ pip --version
pip 20.0.2 from /usr/lib/python3/dist-packages/pip (python 3.8)
```

If the command output looks similar to the following, the command is missing, and you have to install it:

```sh
[Provisioning System] $ pip --version
Command 'pip' not found, but there are 18 similar ones.
```

To install the `pip` command, execute the following command:

```sh
[Provisioning System] $ sudo apt install python3-pip
```

##### Run Pip

Installation of the dependencies requires a `pip` configuration file which is a part of the Developer Experience Kit
repository. It is typically performed only once – unless the same provisioning system is used to install different
releases of the experience kit.

To install the Python packages required by the `dek_provision.py` and the `dek_flash.py` scripts, change the current
directory to the root of the Developer Experience Kit repository (see the
[Repository Cloning](#default-repository-cloning) section) and use the following command:

```sh
[Provisioning System] $ cd <developer-experience-kit>
[Provisioning System] $ sudo pip install -r dek_requirements.txt
```

It is essential to use `sudo` to install these dependencies globally as they also have to be available for the root
user.

## Provisioning Process Scenarios

### Default Provisioning Scenario

The default provisioning process consists of the following stages:

* [Repository Cloning](#default-repository-cloning)
* [Configuration](#default-configuration)
* [Artifacts Building](#default-artifacts-building)
* [Services Start-Up](#default-services-start-up)
* [Installation Media Flashing](#default-installation-media-flashing)
* [System Installation](#default-system-installation)
* [Services Shut Down](#default-services-shut-down)

<a id="default-repository-cloning"></a>
#### Repository Cloning

To be able to run the provisioning utility, clone the chosen experience kit repository. You can checkout the `main`
branch to access the latest experience kit version or select a specific release. In the second case, it is advised to
use the provisioning instruction published with the release to avoid incompatibilities caused by the process evolution.

Due to some limitations of the underlying technology, the length of the absolute path to the directory, which you use
for experience kit cloning, shouldn't exceed 75 characters. This path includes the parent directory and the clone
destination directory. For details and alternative workarounds to this limitation, see the [ESP destination directory
path is too long](#the-esp-destination-directory-path-is-too-long) troubleshooting article.

For your convenience, you can change the current working directory to the directory into which you have cloned the
experience kit (the future instructions will assume that you did so):

```sh
[Provisioning System] $ git clone https://github.com/smart-edge-open/open-developer-experience-kits.git ~/odek
[Provisioning System] $ cd ~/odek
```

If you didn't install provisioning scripts' dependencies on this provisioning system instance before, install them
using the instruction provided in the [Install Python Dependencies](#install-python-dependencies) section.

<a id="default-configuration"></a>
#### Configuration

The default provisioning process of Intel® Smart Edge Open experience kits does not require special configuration
steps. It may, however, be necessary to customize some of them in specific environments. For this purpose, the operator
can set some of the most common parameters using the command line interface. For details, see the [Command Line
Arguments](#command-line-arguments) section.

Less common options can only be adjusted using a [custom configuration file](#configuration-file-summary).
If this is your case, please follow the [Custom Provisioning Scenario](#custom-provisioning-scenario).

<a id="default-artifacts-building"></a>
#### Artifacts Building

To build provisioning services, run the following command from the root directory of the experience kit
repository. You can also use command-line arguments like `--registry-mirror` to specify some typical options:

```sh
[Provisioning System] $ sudo ./dek_provision.py
```

<a id="default-services-start-up"></a>
#### Services Start-Up

```sh
[Provisioning System] $ sudo ./dek_provision.py --run-esp-for-usb-boot
```

> *Note:* ESP creates some network services to support remote installation, these services can be blocked by the
> firewall. Please see [ESP documentation](https://github.com/intel/Edge-Software-Provisioner) to check what services
> are working and in which scenario.

<a id="default-installation-media-flashing"></a>
#### Installation Media Flashing

To flash the installation image onto the flash drive, insert the drive into a USB port on the provisioning system and
run the following command:

```sh
[Provisioning System] # lsblk

NAME                           MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                            7:0    0 31.1M  1 loop /snap/snapd/10707
loop1                            7:1    0 69.9M  1 loop /snap/lxd/19188
loop2                            7:2    0 55.4M  1 loop /snap/core18/1944
sda                              8:0    0  1.8T  0 disk 
├─sda1                           8:1    0  512M  0 part /boot/efi
├─sda2                           8:2    0    1G  0 part /boot
└─sda3                           8:3    0  1.8T  0 part 
  ├─ubuntu--vg-ubuntu--lv-real 253:0    0  880G  0 lvm  
  │ ├─ubuntu--vg-ubuntu--lv    253:1    0  880G  0 lvm  /
  │ └─ubuntu--vg-clean         253:3    0  880G  0 lvm  
  └─ubuntu--vg-clean-cow       253:2    0  400G  0 lvm  
    └─ubuntu--vg-clean         253:3    0  880G  0 lvm  
sdb                              8:16   0  1.8T  0 disk 
sdc                              8:32   1 57.3G  0 disk 
├─sdc1                           8:33   1  1.1G  0 part 
├─sdc2                           8:34   1  3.9M  0 part 
└─sdc3                           8:35   1 56.2G  0 part 
```

The command should list all available block devices. Check which one is the inserted USB drive e.g. "/dev/sdc"
and run the following command:

```sh
[Provisioning System] $ sudo ./dek_flash.py -d <usb_drive>
```

The command will display an interactive menu allowing the selection of a profile to flash. If the `dek_flash.py`
utility cannot determine the BIOS type, it will expect the user to choose one of the two supported types (legacy or
EFI).

After acknowledging that everything is set up correctly by the user, the flashing process will start.

<a id="default-system-installation"></a>
#### System Installation

Begin the installation by inserting the flash drive into the target system. Reboot the system, and enter the BIOS to
boot from the installation media.

> *Note:* Developer Experience Kit does not support provisioning with Secure Boot enabled. Secure Boot can, however, be
> enabled on the reference platform while provisioning. Please refer to [Secure Boot and TPM](#secure-boot-and-tpm).

##### Log Into the System After Reboot

The system will reboot as part of the installation process.

The login screen will display the system's IP address and the status of the experience kit deployment.
To log into the system, use `smartedge-open` as both the user name and password.

##### Check the Status of the Installation

When logging in using a remote console or SSH, a message will be displayed informing about the status of the
deployment, for example:

```text
Smart Edge Open Deployment Status: in progress
```

Three statuses are possible:
- `in progress` &ndash; deployment is in progress,
- `deployed` &ndash; deployment was successful, and the cluster is ready to be used,
- `failed` &ndash; an error occurred during the deployment.

Check the installation logs by running the following command:

```sh
[Provisioned System] $ sudo journalctl -xefu seo
```

Alternatively, you can inspect the deployment log in `/opt/seo/logs`.

<a id="default-services-shut-down"></a>
#### Services Shut Down

```sh
[Provisioning System] $ sudo ./dek_provision.py --stop-esp
```

### Custom Provisioning Scenario

The custom provisioning scenario is very similar to the [default scenario](#default-provisioning-scenario). The only
difference is that it uses the [configuration file](#configuration-file-summary) to adjust some of the
provisioning parameters.

See the [Default Provisioning Scenario](#default-provisioning-scenario) for the description of the common stages.

<a id="custom-configuration"></a>
#### Configuration

Generate a new configuration file as described in the [Configuration File Generation](#configuration-file-generation) section:

```sh
[Provisioning System] $ ./dek_provision.py --init-config > custom.yml
```

<a id="custom-artifacts-building"></a>
#### Artifacts Building

The provisioning command builds all the artifacts in the same way as in the case of the [default
scenario](#default-artifacts-building). The only difference is that the custom config file has to be specified using
the `--config` command-line option:

```sh
[Provisioning System] $ sudo ./dek_provision.py --config=custom.yml
```

## Provisioning Configuration

### Configuration Methods

The provisioning utility and consequently the provisioning process allows two configuration methods:

* via command-line arguments of the provisioning utility,
* via provisioning configuration YAML file.

These methods can be used exclusively or mixed. The configuration options provided by the command-line arguments always
override specific options provided by the configuration file.

Not all the options possible to be customized in the configuration file can also be customized using the command-line
arguments. However, the provisioning script is designed to allow the deployment of a standard experience kit cluster
using the command-line options only.

### Command Line Arguments

For the description of the options available for the command line use, see the provisioning utility help:

```sh
[Provisioning System] $ ./dek_provision.py -h
```

### Configuration File Generation

To generate a custom configuration file, use the provisioning command's `--init-config` option. When executed with this
argument, the tool will print the default provisioning configuration (in the YAML format) to its standard output. The
user has to redirect it to a file of choice to keep it for further use:

```sh
[Provisioning System] $ ./dek_provision.py --init-config > custom.yml
```

The operator can then modify the file to adjust needed options. To instruct the provisioning utility to use the custom
configuration file, use the `--config` option, e.g.:

```sh
[Provisioning System] $ sudo ./dek_provision.py --config=custom.yml
```

### Configuration File Summary

See comments in the generated configuration file for the description of available options.

### Experience Kit Configuration

The system operator can adjust an experience kit's configuration and provide extra files if needed and expected by the
specific deployment variant. These adjustments are made in the configuration file and are defined independently for
each of the provisioned kits (represented by items of the `profiles` list). The operator sets the kit variables using
`group_vars` and `hosts_vars` objects and adds the files to the `sideload` list:

```yaml
profiles:
  - name: Smart_Edge_Open_Developer_Experience_Kits
[…]
    group_vars:
      groups:
        all:
        controller_group:
        edgenode_group:

    host_vars:
      hosts:
        controller:
        node01:

    sideload: []
```

The experience kit variables set in the configuration file override the default values provided by the experience kit.

The provisioning command copies the operator-provided files specified in the `sideload` list from a local location to
the provisioned system.

### Operating System Account

The operator can define operating system user name and password for each of the defined experience kit profiles:

```yaml
profiles:
  - name: Smart_Edge_Open_Developer_Experience_Kits
[…]
    # Credentials of the operating system account that will be created.
    # Account will be added to the sudoers.
    account:
      username: smartedge-open
      password: <secret>
[…]
```

The provisioning tool will generate a random password if the `password` field is missing. The script will also set the
user name to `smartedge-open` if the `username` field is missing. It will print the user name and password to the
screen just before the end of its execution.

### Git Credentials

Access to some experience kit repositories may be limited and controlled using git credentials. In such a case, the
operator must provide these credentials to the provisioning script.

The first method of providing them is through the `git` object of a custom configuration file:

```yaml
git:
  user: '<user-name>'
  password: '<user-password>'
```

The second method is to use the following provisioning command options:

```sh
[provisioning system] # ./dek_provision.py -h
[…]
  --git-user NAME       NAME of the git remote user to be used to clone required Smart Edge Open repositories
  --git-password VALUE  Git remote token to be used to clone required Smart Edge Open repositories
[…]
```

The credentials are used during the provisioning script (e.g., `dek_provision.py`) execution and other contexts like
provisioning services containers and installer system, so the operator has to provide them explicitly.

The script will try to verify if it can access all the repositories specified through the configuration file and fail
if they cannot be accessed anonymously or with the operator-provided credentials. This functionality doesn't always
work, and eventually, it is the operator's responsibility to provide the credentials if needed.


The scenario in which different repositories are accessed using different credentials is currently not supported. All
the repositories must be either public or available for a specific user. The only supported protocols are HTTP and
HTTPS, both anonymous and authenticated with username and password. To authenticate using the Github token, provide it
as the git password.

### Docker Pull Rate Limit

It is possible to use local Docker registry mirrors or Docker Hub
credentials to mitigate the [Docker pull rate limit](https://docs.docker.com/docker-hub/download-rate-limit/) consequences.

#### Docker Registry Mirror

It is the operator's responsibility to deploy a working Docker registry mirror. When it is up and running, the operator
can provide its URL to the provisioning script.

The first method of providing it is through the `docker` object of a custom configuration file:

```yaml
docker:
  registry_mirrors: ['http://example.local:5000']
```

The second method is to use the `--registry-mirror` option of the provisioning script:

```sh
[provisioning system] $ ./dek_provision.py -h
[…]
  --registry-mirror URL
                        add the URL to the list of local Docker registry mirrors
[…]
```

If the custom configuration file contains some `registry_mirrors` list items, then the URL specified using the
`--registry-mirror` option will be appended to the end of the list.

It is important to note that the provisioning script won't configure the provisioning system (i.e., the system used to
run the provisioning tools) to use the registry mirrors. It is the operator's responsibility to set it up. The script
only takes care of the configuration of the installer and the provisioned system.

#### Docker Hub Credentials

The system operator can configure the ESP operating system installer to use specific Docker Hub credentials when
pulling images from Docker Hub. The operator can set these credentials in the configuration file or pass it using
dedicated command-line options.

To set the credentials in the configuration file, adjust the `username` and `password` fields of the `dockerhub`
subsection of the `docker` section:

```yaml
docker:
  dockerhub:
    username: "<user-name>"
    password: "<user-password>"
```

To provide credentials using the command-line, specify the following options:

```sh
[provisioning system] # ./dek_provision.py -h
[…]
  --dockerhub-user NAME
                        NAME of the user to authenticate with DockerHub during Live System stage
  --dockerhub-pass VALUE
                        Password used to authenticate with DockerHub during Live System stage
[…]
```

> *Note:* Credentials provided using the methods above won't affect provisioning and the provisioned systems. It is the
> operator's responsibility to configure them.

### PXE

The Smart Edge Open provisioning process supports network boot with PXE protocol. This scenario does not need any
installation media to carry a bootable image.

> *Note:* PXE protocol is a DHCP extension. Enabling this provisioning feature will disrupt the existing DHCP server
> (if there is any). The operator should use this installation method in compatible networks only.

By default, the PXE boot is not active. The system operator can configure it in the configuration file's `dnsmasq`
section. They should set the `enabled` field to `true` and fill other section fields according to their needs:

```yaml
dnsmasq:
  # If true, then the dnsmasq will be started with rest of the Provisioning System suite.
  enabled: true

  # Domain Name System (DNS) settings
  # These values should be changed in case of default DNS (8.8.4.4 and 8.8.8.8) are not reachable.
  network_dns_primary: ''     # e.g. 8.8.4.4
  network_dns_secondary: ''   # e.g. 8.8.8.8

  # DHCP and network settings
  dhcp_range_minimum: ''      # e.g. 192.168.1.100
  dhcp_range_maximum: ''      # e.g. 192.168.1.250
  network_broadcast_ip: ''    # e.g. 192.168.1.255
  network_gateway_ip: ''      # e.g. 192.168.1.1

  # IP address of the Provisioning System
  host_ip: ''                 # e.g. 192.168.1.2
```

If the operator didn't set any of the above values, the provisioning tool would try to determine them based on the
current network configuration.

> *Note:* Dnsmasq component provides DHCP and TFTP services. The provisioning tool starts them on all the available
> network interfaces.

The PXE provisioning scenario differs from the default scenario in the following steps.

#### Services startup

While starting provisioning service, the script invocation has to include an additional parameter to support PXE boot mode `--run-esp-for-pxe-boot`.

```sh
[provisioning system] $ sudo ./dek_provision.py --config custom.yml --run-esp-for-pxe-boot
```

#### Installation Media Flashing

The PXE scenario does not require any installation media.

#### System installation

Reboot the provisioned system, and enter BIOS to boot from the network with PXE.

> *Note:* There is a known issue related to the PXE EFI boot not working on Dell PowerEdge R750. Please use the legacy
> BIOS version instead.

### Secure Boot and TPM

The provisioning tool can automatically switch the provisioned machine's Secure Boot and TPM features according to the
system operator's preferences. In the Developer Experience Kit, the default behavior is to enable them. The system
operator can change it using a custom configuration file.

If the operator wants to apply the same security settings to all the provisioned machines, they can adjust the
top-level `bios` section:

```yaml
bios:
  tpm: false
  secure_boot: true
```

The operator may override the global security settings per host or experience kit profile. The order of precedence is:
* global settings (lowest priority)
* host settings
* profile settings (top priority)

To override the security settings on the host level, the operator should add the `bios` section to the chosen host (see
the [hosts](#hosts) section for details):

```yaml
hosts:
  - name: node001
    […]
    bios:
      tpm: true
      secure_boot: true
  - […]
```

To override the security settings on the profile level, the operator should add the `bios` section to the chosen profile:

```yaml
profiles:
  - name: Smart_Edge_Open_Developer_Experience_Kits
    […]
    bios:
      tpm: true
      secure_boot: true
  - […]
```

Secure Boot and TPM modification is done in the profile using the [bmc](#bmc) section data, provisioning machine has to have
access to the out-of-band BMC interface of provisioned machine. It is necessary to provide BMC address together with
user and password. If host bios allows in-host BMC access then internal access to BMC can be used and access to
out-of-band interface from provisioning machine is not needed.


### BMC

BMC stands for Baseboard Management Controller, it is used to remotely control the hardware. There is a global section named **bmc**,
it defines BMC address, user name and password, it is only used to provide general authentication information for all
hosts in the cluster. Another possible use case is that in-host BMC passthrough is enabled so it allows to control
hardware settings from inside of given host, assuming the internal address of BMC interface is always the same.

```yaml
bmc:
  address: 169.254.1.1
  user: admin
  password: adminpassword
```

BMC settings control can be overridden for given host in the [hosts](#hosts) section for each configured host.


### Hosts Configuration

The system operator can use the `hosts` section of the configuration file to provide host-specific information and set
some variables for individual hosts.

The host-specific information variables include:
* `mac` &ndash; a MAC address of one of the host's network interfaces (used to identify the host),
* `bmc` &ndash; address and authentication credentials of the host's BMC interface.

The individual settings include:
* `bios` &ndash; security settings (see the [Secure Boot and TPM](#secure-boot-and-tpm) section),
* `name` &ndash; the hostname to be set after the operating system installation (see the
  [Machine Hostname Configuration](#machine-hostname-configuration)).

The operator may override any subset of the fields of the `bmc` and `bios` structures. The provisioning tool will take
the omitted fields' values from the global settings.

See the following configuration file fragment for the `hosts` section example:

```yaml
hosts:
  - name: node001
    mac: 11:22:33:44:55:66
    bios:
      secure_boot: true
    bmc:
      user: root
      password: root
  - name: node002
    mac: aa:bb:cc:dd:ee:ff
    bios:
      tpm: true
    bmc:
      address: node017.cluster
  - name: node003
    mac: 66:55:44:33:22:11
    bios:
      tpm: true
      secure_boot: true
```

### Machine Hostname Configuration

Each host in ***hosts*** section may contain an optional ***name*** parameter.<br/>
When ***name*** is specified, the value will be set as hostname of the provisioned machine which MAC address of any network card matches the given ***mac*** value.<br/>
Example - Let's assume following configuration:
```yaml
hosts:
  - mac: 11:22:33:44:55:66
    bios:
      secure_boot: true
    bmc:
      user: root
      password: root
  - name: node002
    mac: aa:bb:cc:dd:ee:ff
```
This will result in a machine with mac ***11:22:33:44:55:66*** to have a randomly generated hostname (ex. 'ubuntu-d4d06edb08' for Ubuntu profile).
The machine with mac address ***aa:bb:cc:dd:ee:ff*** will have its hostname changed to ***node002*** during provisioning process.

## Troubleshooting

### Required Python dependencies are not installed

#### Problem

The following error message is displayed when you attempt to run one of the provisioning scripts (e.g., `dek_provision.py` or `dek_flash.py`):

```text
ERROR: Required Python dependencies are not installed
```

#### Solution

Install the dependencies using the instruction in the [Install Python Dependencies](#install-python-dependencies) section.

### The ESP destination directory path is too long

#### Problem

The following error message is displayed when you attempt to run the provisioning script (the actual path will differ):

```text
ERROR: The ESP destination directory path is too long.
    Please make the following path shorter by at least 2 characters to be able to proceed:
    /home/user162530/some/very/long/path/into/which/the/experience/kit/was/cloned/esp/data/tmp/build/docker.sock
```

#### Explanation

This error is related to a limitation of the underlying technologies. To understand possible solutions better, let's
dissect the full absolute path into four logical parts:

```text
<absolute-parent-path>/<clone-path>/<esp-path>/data/tmp/build/docker.sock
```


* `<absolute-parent-path>` &ndash; the real, absolute parent path to the directory into which you have cloned the experience
  kit repository,
* `<clone-path>` &ndash; the name of the clone destination directory (it may be provided explicitly or derived by git from
  the repository URL),
* `<esp-path>` &ndash; a configurable ESP working directory,
* `/data/tmp/build/docker.sock` &ndash; a static part, you have no influence on.

The total length of the composite path (converted to a real, absolute path) mustn't exceed 106 characters.

#### Solution #1

The easiest way to deal with this limitation is to choose a convenient parent path. If it is reasonably short, i.e.,
its length is 43 characters or less, then even the default clone destination directory should work.

```sh
[Provisioning System] $ cd /home/userx/some/reasonably/long/path/name
[Provisioning System] $ git clone https://github.com/smart-edge-open/open-developer-experience-kits.git
[Provisioning System] $ cd open-developer-experience-kits
[Provisioning System] $ ./dek_provision.py
INFO: ESP repository is public: https://github.com/intel/Edge-Software-Provisioner
[…]
```

We advise you to keep the parent path even shorter than 43 characters.

#### Solution #2

Another reasonably easy solution is to specify the clone output directory explicitly, to make the composite path fit
the limit even if the parent path exceeds it:

```sh
[Provisioning System] $ cd /home/userx/some/a/little/too/long/parent/path
[Provisioning System] $ git clone https://github.com/smart-edge-open/open-developer-experience-kits.git odek
[Provisioning System] $ cd odek
[Provisioning System] $ ./dek_provision.py
INFO: ESP repository is public: https://github.com/intel/Edge-Software-Provisioner
[…]
```

#### Solution #3

The third solution is to change the default provisioning configuration to set the ESP destination directory to some
shorter path. You can, for example, set it to be relative to the parent directory or even use an absolute path for
it. The main disadvantage of this solution is that it requires provisioning configuration customization. You have to
generate a custom config, then adjust it, and finally provide it to the provisioning command:

```sh
[Provisioning System] $ cd /home/userx/some/a/little/too/long/parent/path
[Provisioning System] $ git clone https://github.com/smart-edge-open/open-developer-experience-kits.git
[Provisioning System] $ cd open-developer-experience-kits
[Provisioning System] $ ./dek_provision.py --init-config > custom.yml
INFO: Provisioning configuration generated
```

After the custom configuration file generation, you have to adjust the `dest_dir` variable in the `esp` section of the
config (it is `'./esp'` by default):

```text
   # ESP destination path.
   # This will be a path where ESP will be cloned to (relative to the script working directory).
-  dest_dir: './esp'
+  dest_dir: '../esp'
```

The provisioning command provided with this custom configuration file shouldn't complain about the path length if the
resulting ESP path fits the limit:

```sh
[Provisioning System] $ ./dek_provision.py -c custom.yml
INFO: ESP repository is public: https://github.com/intel/Edge-Software-Provisioner
[…]
```

#### Solution #4

Another solution relies on the provisioning command considering the ESP destination directory path to be relative to
the current working directory. If you change it, e.g., to the parent directory and then execute the provisioning
command from it, it should work correctly:

```sh
[Provisioning System] $ cd /home/userx/some/a/little/too/long/parent/path
[Provisioning System] $ git clone https://github.com/smart-edge-open/open-developer-experience-kits.git
[Provisioning System] $ ./open-developer-experience-kits/dek_provision.py
INFO: ESP repository is public: https://github.com/intel/Edge-Software-Provisioner
[…]
```

### Docker has to be installed

#### Problem

One of the following error messages may appear when you attempt to run the provisioning script (e.g.,
`dek_provision.py`):

```text
ERROR: Failed to execute 'docker --version':
    [Errno 2] No such file or directory: 'docker'
```

#### Solution

Install Docker software according to the official instruction:
[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/).


### The docker-compose tool has to be installed

#### Problem

The following error message appears when you attempt to run the provisioning script (e.g., `dek_provision.py`):

```text
ERROR: Failed to execute 'docker-compose --version':
    [Errno 2] No such file or directory: 'docker-compose'
```

#### Solution

Install `docker-compose` tool according to the official instruction:
[Install Docker Compose](https://docs.docker.com/compose/install/).


### Failed to confirm that Docker is configured correctly

#### Problem

Basic Docker commands have failed to confirm that Docker is installed or has Internet connectivity.

#### Solution

Verify that both Docker and Docker Compose are installed. See the official Docker documentation for [Docker](https://docs.docker.com/engine/install/) and [Docker Compose](https://docs.docker.com/compose/install/) installation.

Verify network connection. If using proxy, set up proxy for following Docker components:
* [Docker cli](https://docs.docker.com/engine/reference/commandline/cli/#automatic-proxy-configuration-for-containers)
* [Docker daemon](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy)

To investigate further, run provided failing command manually.


### Installation image couldn't be found in the expected location

#### Problem

The following error is displayed when you attempt to run the provisioning script (e.g., `dek_provision.py`):

```text
ERROR: Installation image couldn't be found in the expected location
    <img-file-path>
```

#### Solution

Retry the build attempt by rerunning the same provisioning command:

```sh
[Provisioning System] $ sudo ./dek_provision.py […]
```

If it doesn't help, retry the build one more time with the `--cleanup` flag added. This option will force the complete
rebuild.

```sh
[Provisioning System] $ sudo ./dek_provision.py --cleanup […]
```

### ESP script failed

#### Problem

One of the following error messages is displayed, and the execution of a provisioning script stops:

```text
ERROR: ESP script failed: build.sh
ERROR: ESP script failed: makeusb.sh
ERROR: ESP script failed: run.sh
```

#### Solution

Retry the build attempt by rerunning the same provisioning command with the `--cleanup` flag added:

```sh
[Provisioning System] $ sudo ./dek_provision.py --cleanup […]
```
If it doesn't help, you may inspect the ESP logs, typically located in the ./esp/builder.log file.

### Git has to be installed

#### Problem

The following error message appears when you try to download a repository (e.g., `git clone [...]`):

```text
-bash: git: command not found
```

#### Solution

Install `Git` tool according to the official instruction:
[Install Git](https://git-scm.com/download/linux).


### Provisioning configuration file validation failed

#### Problem

The following error is displayed when you attempt to run the provisioning script with a custom
[configuration file](#configuration-file-summary) (e.g., `dek_provision.py -c <config-path>`):

```text
ERROR: The provisioning configuration file ('<config-path>') validation failed:
    <issue-yaml-path>:
        <issue-description>
    […]
```

The issue list may contain one or more findings. The actual message can look like the following:
```text
ERROR: The provisioning configuration file ('custom.yml') validation failed:
    esp:
        'url' is a required property
    profiles/0/experience_kit/deployment:
        1 is not of type 'string'
```

#### Solution

This error indicates one or more problems with the custom configuration file. This problem isn't related to the YAML
syntax which is correct but rather to its structure not matching the expected schema.

The `<issue-yaml-path>` part of the error message identifies the fragment of the configuration file the
`<issue-description>` refers to. The `profiles/0/experience_kit/deployment` path in the above example refers to the
following region of the configuration part:

```yaml
profiles:
  - name: […]
    […]
    experience_kit:
      […]
      deployment: 1
```

After locating the referred region, analyze the `<issue-description>` part of the error message to determine the exact
problem. The above example shows the case of a wrong type and value of the `deployment` property. With the help of
configuration file comments (see the output of the `dek_provision.py --init-config` command) it should be possible to
fix the issue.

Occasionally, the error messages produced by the provisioning command may include fragments of JSON data. These
fragments may look unfamiliar, but they represent parts of the provided configuration file (converted from YAML to JSON
for validation purposes). Identifying the configuration file region may be easier with a JSON to YAML conversion tool
(e.g., https://www.json2yaml.com/).

### Command 'docker-compose' cannot be executed

#### Problem

The following error message is displayed when the script tries to execute the `docker-compose` command:

```text
ERROR: command: 'docker-compose --version' cannot be executed
    See the Troubleshooting section of the Intel® Smart Edge Open Provisioning Process document
```

The `docker-compose` command has been replaced by the `docker compose` plugin in Docker Compose v2.
Because of this, it is required to install a standalone version of the program that supports the old command version.

#### Solution

Install `Docker Compose` standalone version according to the official instruction:
[Install docker-compose standalone](https://docs.docker.com/compose/install/compose-plugin/#install-the-plugin-manually).
