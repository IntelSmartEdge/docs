```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```

# Intel® Smart Edge Open Provisioning Process

* [Overview](#overview)
* [Preconditions and software requirements](#preconditions-and-software-requirements)
  * [Software requirements](#software-requirements)
  * [Preconditions](#preconditions)
* [Provisioning Process Scenarios](#provisioning-process-scenarios)
  * [Default Provisioning Scenario](#default-provisioning-scenario)
    * [Repository Cloning](#default-repository-cloning)
    * [Configuration](#default-configuration)
    * [Artifacts Building](#default-artifacts-building)
    * [Services Start Up](#default-services-start-up)
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
  * [Git Credentials](#git-credentials)
  * [Docker Pull Rate Limit](#docker-pull-rate-limit)
  * [Docker registry Mirror](#docker-registry-mirror)
  * [Docker Hub Credentials](#docker-hub-credentials)
  * [PXE](#pxe)
  * [Secure Boot and TPM](#secure-boot-and-tpm)
  * [BMC](#bmc)
  * [Hosts](#hosts) 
  * [Changing machine hostname](#changing-machine-hostname)
* [Troubleshooting](#troubleshooting)

## Overview

The Intel® Smart Edge Open automated provisioning process relies on the [Intel® Edge Software
Provisioner](https://github.com/intel/Edge-Software-Provisioner) (ESP). It provides a method of
installation operating system automatically and the Intel® Smart Edge Open cluster deployment.

The Developer Experience Kit provides the `dek_provision.py` command-line utility, using the
Intel® Edge Software Provisioner toolchain to deliver a smooth installation experience.

The provisioning process requires a temporary provisioning system operating on a separate machine
and routable from the subnet the provisioned machines are supposed to work on.

## Preconditions and software requirements

### Software requirements
* Ubuntu 20.04
* Docker 18.09.3 or greater
* Docker-compose v1.23.2 or greater
* Python v3.6 or greater
* Pip 20.0 or greater
* Bash v4.3.48 or greater
* Git v2.25.1 or greater

### Preconditions
The following steps should be performed on a clean machine in order to prepare the system for the provisioning process:

1. [Install Docker](#docker-has-to-be-installed)
2. [Install Docker Compose](#the-docker-compose-tool-has-to-be-installed)
3. [Install Git](#git-has-to-be-installed)
4. [Install Python dependencies](#install-python-dependencies)

#### Install Python Dependencies

##### Install Pip

The Python dependencies installation process relies on the `pip` utility, which may not be installed on a Ubuntu system
by default. To check if the `pip` program is available, use the `pip --version` command. If its output looks like the
following, the `pip` program is ready to be used:

```Shell.bash
[Provisioning System] $ pip --version
pip 20.0.2 from /usr/lib/python3/dist-packages/pip (python 3.8)
```

If the command output looks similar to the following, the command is missing, and you have to install it:

```Shell.bash
[Provisioning System] $ pip --version
Command 'pip' not found, but there are 18 similar ones.
```

To install the `pip` command, execute the following command:

```Shell.bash
[Provisioning System] $ sudo apt install python3-pip
```

##### Run Pip

Installation of the dependencies requires a `pip` configuration file which is a part of the Developer Experience Kit
repository. It is typically performed only once – unless the same provisioning system is used to install different
releases of the experience kit.

To install the Python packages required by the `dek_provision.py` and the `dek_flash.py` scripts, change the current
directory to the root of the Developer Experience Kit repository (see the
[Repository Cloning](#default-repository-cloning) section) and use the following command:

```Shell.bash
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

Each of the Intel® Smart Edge Open experience kits comes with its provisioning utility tailored to the kit's
requirements. This script resides in the root directory of an experience kit repository, and its name matches the
following pattern: `<experience-kit-name-abbreviation>_provision.py`, e.g., `dek_provision.py`.

To be able to run the provisioning utility, clone the chosen experience kit repository. You can checkout the `main`
branch to access the latest experience kit version or select a specific release. In the second case, it is advised to
use the provisioning instruction published with the release to avoid incompatibilities caused by the process evolution.
For convenience, you can change the current directory to the directory the kit is cloned to, e.g.:

```Shell.bash
[Provisioning System] # git clone https://github.com/smart-edge-open/open-developer-experience-kits.git --branch=smart-edge-open-21.09 ~/dek
[Provisioning System] # cd ~/dek
```

If you didn't install provisioning scripts' dependencies on this provisioning system instance before, install them
using the instruction provided in the [Install Python Dependencies](#install-python-dependencies) section.

<a id="default-configuration"></a>
#### Configuration

The Intel® Smart Edge Open default provisioning process is designed not to require any special configuration steps. The
provisioning scripts should work without any configuration options specified. In some environments, it may, however, be
necessary to customize some of them.  For this purpose, the operator can set some of the most common parameters using
the command line interface.  For details, see the [Command Line Arguments](#command-line-arguments) section.

If there is a need to adjust
[configuration parameters exposed by the configuration file](#configuration-file-summary)
then the [Custom Provisioning Scenario](#custom-provisioning-scenario) should be followed.


<a id="default-artifacts-building"></a>
#### Artifacts Building

To build the provisioning services, run the following command from the root directory of the Developer Experience Kit
repository. You can also use command-line arguments like `--registry-mirror` to specify some typical options:

```Shell.bash
[Provisioning System] # ./dek_provision.py
```

<a id="default-services-start-up"></a>
#### Services Start-Up

```Shell.bash
[Provisioning System] # ./dek_provision.py --run-esp-for-usb-boot
```

> NOTE: ESP creates some network services to support remote installation, these services can be blocked by the firewall.
Please see [ESP documentation](https://github.com/intel/Edge-Software-Provisioner) to check what services are working in which scenario.

<a id="default-installation-media-flashing"></a>
#### Installation Media Flashing

To flash the installation image onto the flash drive, insert the drive into a USB port on the provisioning system and
run the following command:

```Shell.bash
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

```Shell.bash
[Provisioning System] # ./dek_flash.py -d <usb_drive>
```

The script should present an interactive menu allowing the selection of a certain profile to flash. In case the script cannot determine a specific bios type from default or provided configuration file, it will also give an option to select legacy "bios" or "efi" type.

After acknowledging that everything is set up correctly by the user, the flashing process will start.

<a id="default-system-installation"></a>
#### System Installation

Begin the installation by inserting the flash drive into the target system. Reboot the system, and enter the BIOS to
boot from the installation media.

> NOTE: Developer Experience Kit does not support provisioning with Secure Boot enabled, however, Secure Boot can be enabled on reference platform while provisioning. Please refer to [Secure Boot and TPM](#secure-boot-and-tpm).

##### Log Into the System After Reboot

The system will reboot as part of the installation process.

The login screen will display the system's IP address and the status of the experience kit deployment.
To log into the system, use `smartedge-open` as both the user name and password.

##### Check the Status of the Installation

When logging in using remote console or SSH, a message will be displayed that informs about the status of the deployment, for example,
```Smart Edge Open Deployment Status: in progress```

Three statuses are possible:
- `in progress` - deployment is in progress
- `deployed` - deployment was successful - Developer Experience Kit cluster is ready
- `failed` - error occurred during the deployment

Check the installation logs by running the following command:

```Shell.bash
[Provisioned System] $ sudo journalctl -xefu seo
```
Alternatively, you can inspect the deployment log found in `/opt/seo/logs`.

<a id="default-services-shut-down"></a>
#### Services Shut Down

```
[Provisioning System] # ./dek_provision.py --stop-esp
```

### Custom Provisioning Scenario

The custom provisioning scenario is very similar to the [default scenario](#default-provisioning-scenario). The only
difference is that it uses the [configuration file](#configuration-file-summary) to adjust some of the
provisioning parameters.

See the [Default Provisioning Scenario](#default-provisioning-scenario) for the description of the common stages.

<a id="custom-configuration"></a>
#### Configuration

Generate a new configuration file as described in the [Configuration File Generation](#configuration-file-generation) section:

```bash
[Provisioning System] # ./dek_provision.py --init-config > custom.yml
```

<a id="custom-artifacts-building"></a>
#### Artifacts Building

The provisioning artifacts are built in the same way as in the case of the
[default scenario](#default-artifacts-building). The only difference is that the custom config file has to be specified using
the `--config` command-line option:

```bash
[Provisioning System] # ./dek_provision.py --config=custom.yml
```

## Provisioning Configuration

### Configuration Methods

The provisioning utility and consequently the provisioning process allows two configuration methods:

* Configuration via command line-arguments of the provisioning utility
* Configuration via provisioning configuration YAML file

These methods can be used exclusively or mixed. The configuration options provided by the
command line arguments always override specific options provided by the configuration file.

Not all the options possible to be customized in the configuration file can also be customized using the command line
arguments.  However, the provisioning script is designed to allow the deployment of a standard experience kit cluster
using the command-line options only.

### Command Line Arguments

For the description of the options available for the command line use, see the provisioning utility help:

```
[Provisioning System] # ./dek_provision.py -h
```

### Configuration File Generation

To generate a custom configuration file, use the `--init-config` option of the provisioning utility.  When handling
this command, the utility prints an experience kit default configuration in the YAML format to the standard output.  It
has to be redirected to a file of choice to keep it for further use:

```bash
[Provisioning System] # ./dek_provision.py --init-config > custom.yml
```

The operator can then modify the file to adjust needed options. To instruct the provisioning utility to use the custom
configuration file, use the `--config` option, e.g.:

```
[Provisioning System] # ./dek_provision.py --config=custom.yml
```

### Configuration File Summary

For the description of the options available for the configuration file, see comments within the generated
configuration file.

### Experience Kit Configuration

For each provisioned experience kit, it is possible to adjust its configuration and specify user-provided files if
needed for a specific deployment variant. Both of these configuration adjustments are currently possible
through the [configuration file](#configuration-file-summary) only. For each of the provisioned experience kits (item
of the `profiles` list), it is possible to set its deployment variables through the `group_vars`, and `hosts_vars`
objects and the list of operator-provided files through the `sideload` list:

```
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

The experience kit configuration variables specified in the provisioning configuration override the default values
provided by the experience kit, so there is no need to adjust them in the
[default provisioning scenario](#default-provisioning-scenario).

The operator-provided files specified in the `sideload` list are read from a local location, copied to the
provisioning artifacts, and finally to the provisioned system.

### Profile Credentials

For each provisioned experience kit, it is possible to set profile login credentials.

* If username is not set by the user - it will be set to "smartedge-open".
* If password is not set by the user - it will be randomly generated and displayed at the end of the ESP script runtime.

```
profiles:
  - name: Smart_Edge_Open_Developer_Experience_Kits
[…]
    # Credentials of the operating system account that will be created.
    # Account will be added to the sudoers.
    account:
      username: smartedge-open
      password: smartedge-open
[…]
```

### Git Credentials

The access to some of the experience kits may be limited and controlled using git credentials.
In such a case, the operator has to provide these credentials to the provisioning script.

The first method of providing them is through the `git` object of a custom
[configuration file](#configuration-file-summary):

```yml
git:
  user: '<user-name>'
  password: '<user-password>'
```

The second method is to use the git credentials options of the provisioning script:

```bash
[provisioning system] # ./dek_provision.py -h
[…]
  --git-user NAME    Name of the user to be used to clone required Smart Edge Open repositories
  --git-password VALUE  Password to be used to clone required Smart Edge Open repositories
[…]
```

The credentials are used during the provisioning script (e.g., `dek_provision.py`) execution and other contexts like
provisioning services containers and installer system, so the operator has to provide them explicitly.

The script will try to verify if it can access all the repositories specified through the configuration file and fail
if they cannot be accessed anonymously or with the operator-provided credentials. This functionality doesn't always
work, and eventually, it is the operator's responsibility to provide the credentials if needed.

The scenario in which different repositories can be accessed using different credentials is currently not
supported. All the repositories must be either public or available for the specific user. The only supported protocols
are HTTP and HTTPS, both anonymous and authenticated with user and password. Github's token authentication is supported,
the token value should be used to specify git password.

### Docker Pull Rate Limit

It is possible to use local Docker registry mirrors or Docker Hub
credentials to mitigate the [Docker pull rate limit](https://docs.docker.com/docker-hub/download-rate-limit/) consequences.

#### Docker Registry Mirror

It is the operator's responsibility to provide a working Docker registry mirror. When it is up and running, its URL can
be provided to the provisioning script.

The first method of providing it is through the `docker` object of a custom
[configuration file](#configuration-file-summary):

```yml
docker:
  registry_mirrors: ['http://example.local:5000']
```

The second method is to use the `--registry-mirror` option of the provisioning script:

```bash
[provisioning system] # ./dek_provision.py -h
[…]
  --registry-mirror URL
                        add the URL to the list of local Docker registry mirrors
[…]
```

If the custom configuration file contains some `registry_mirrors` list items, then the URL specified using the
`--registry-mirror` option will be appended to the end of the list.

It is important to note that the provisioning script won't configure the provisioning system to use the registry
mirrors. It is the operator's responsibility to set it up. The script only takes care of the configuration of the
installer and the provisioned system.

#### Docker Hub Credentials

The provisioning script provides a possibility to specify Docker Hub credentials to be used by the installer when
pulling images from Docker Hub.

The first method of providing it is through the `docker` object of a custom
[configuration file](#configuration-file-summary):

```yml
docker:
  dockerhub:
    username: "<user-name>"
    password: "<user-password>"
```

The second method is to use the Docker Hub credentials options of the provisioning script:

```bash
[provisioning system] # ./dek_provision.py -h
[…]
  --dockerhub-user NAME
                        NAME of the user to authenticate with DockerHub during Live System stage
  --dockerhub-pass VALUE
                        Password used to authenticate with DockerHub during Live System stage
[…]
```

Only the installer will use these credentials. They won't affect the provisioning and the provisioned systems.

### PXE

Smart Edge Open provisioning process supports network boot with PXE protocol. This provisioning scenario does not need any installation media to carry bootable image.

> NOTE: PXE protocol is a DHCP exentsion, using PXE will disrupt current network DHCP server, so it should be done in separated network environment. 

PXE scenario is disabled by default. To enable it, custom configuration file has to be provided and modified like below:
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

If any of above values is not provided explicitly, it will be guessed from current network configuration.

> NOTE: Dnsmasq component provides DHCP and TFTP services, these services are started on all available network interfaces.

PXE provisioning scenario differs from default scenario in following points.

#### Services startup
While starting provisioning service, the script invocation have to include additional parameter to support PXE boot mode `--run-esp-for-pxe-boot`.
```bash
[provisioning system] # ./dek_provision.py --config custom.yml --run-esp-for-pxe-boot
```
#### Installation Media Flashing
PXE scenario does not require installation media.

#### System installation
Reset provisioned system, enter the BIOS to boot from network with PXE.

> NOTE: Known issue: PXE EFI boot does not work with Dell PowerEdge R750, please use BIOS verion instead.

### Secure Boot and TPM
The Developer Experience Kit supports switching on Secure Boot and TPM feature on the reference platform. This can be done with
configuration file. Both features are enabled by default configuration.

Provisioning configuration allows a global definition of these settings. In such a case, the provisioning script will apply them to all provisioned hosts.
```yaml
bios:
  tpm: false
  secure_boot: true
```

Secure Boot and TPM settings can be overridden in the [hosts](#hosts) section for specific hosts and then
overridden in the profiles section again. Profiles settings have top priority and override settings defined in the global section
and hosts section. 
```yml
profiles:
  - name: Smart_Edge_Open_Developer_Experience_Kits
    url: https://github.com/smart-edge-open/profiles.git
    branch: main
    scenario: single-node
    experience_kit:
      url: https://github.com/smart-edge-open/open-developer-experience-kits.git
      branch: main
      deployment: dek
    controlplane_mac: ''
    account:
      username: smartedge-open
      password: smartedge-open

    bios:
      secure_boot: true
      tpm: true
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


### Hosts
Provisioning script configuration can provide a list of hosts to apply individual settings. This list is
defined in ***hosts*** section. Each entry in **hosts** section defines settings which will be applied to given host.
Individual hosts are identified by MAC address of any network card accessible from given host.

The [BMC](#bmc) settings, Secure Boot and TPM settings can be specified for each host in **hosts** section. Individual
settings are overridden one by one so it is possible to override just a single value like an address and keep user and
password unchanged from global configuration.
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

### Changing machine hostname
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

### Docker has to be installed

#### Problem

One of the following error messages may appear when you attempt to run the provisioning script (e.g.,
`dek_provision.py`):

```text
ERROR: Failed to execute 'docker --version':
    [Errno 2] No such file or directory: 'docker'
    See the Troubleshooting section of the Intel® Smart Edge Open Provisioning Process document
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
    See the Troubleshooting section of the Intel® Smart Edge Open Provisioning Process document
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

```Shell.bash
[Provisioning System] # ./dek_provision.py […]
```

If it doesn't help, retry the build one more time with the `--cleanup` flag added. This option will force the complete
rebuild.

```Shell.bash
[Provisioning System] # ./dek_provision.py --cleanup […]
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

```Shell.bash
[Provisioning System] # ./dek_provision.py --cleanup […]
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
