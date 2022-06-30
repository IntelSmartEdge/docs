```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2022 Intel Corporation
```

# SR-IOV Detection Application

- [SR-IOV Detection Application](#sr-iov-detection-application)
  - [Overview: SR-IOV Requirements](#overview-sr-iov-requirements)
  - [Overview: SR-IOV Detection Application](#overview-sr-iov-detection-application)
  - [Usage](#usage)
    - [Application input parameters](#application-input-parameters)
      - [How to use the input parameters](#how-to-use-the-input-parameters)
    - [Capabilities supported by the application](#capabilities-supported-by-the-application)
    - [Application stages and printings in debug mode for each stage](#application-stages-and-printings-in-debug-mode-for-each-stage)
      - [1) Parsing the configuration YAML file](#1-parsing-the-configuration-yaml-file)
      - [2) Detecting the network devices in the Linux](#2-detecting-the-network-devices-in-the-linux)
      - [3) Checking the detected devices against the configured criteria lists](#3-checking-the-detected-devices-against-the-configured-criteria-lists)
    - [How to edit the configuration YAML file](#how-to-edit-the-configuration-yaml-file)
      - [Strict rules for the YAML file](#strict-rules-for-the-yaml-file)
      - [Example of configurations YAML file with card affinity couples and special capability](#example-of-configurations-yaml-file-with-card-affinity-couples-and-special-capability)
      - [Rules for configuring card affinity couples in the YAML file](#rules-for-configuring-card-affinity-couples-in-the-yaml-file)
      - [Configuring PTP Support in the YAML file](#configuring-ptp-support-in-the-yaml-file)
    - [Known issues](#known-issues)

## Overview: SR-IOV Requirements

The Single Root I/O Virtualization (SR-IOV) feature provides the ability to partition a single physical PCI resource into virtual PCI functions that can be allocated to application and network function pods.
Previous to this detection application, the user had to configure the relevant NIC for SR-IOV.
The objective of this application is to detect automatically the physical NICs, on the node, for use with SR-IOV according to input based on:

1) Priority of PF (Physical Function) detection.
2) Criteria list of capabilities requested for each PF.
This application will be executed from an Ansible role during the provisioning process in deployment of the experience kit.

## Overview: SR-IOV Detection Application

A) The application was implemented in GOLANG.

B) This application can run properly in Linux OS ONLY.

C) The user configures in a configuration YAML file:

   1) How many NICs are needed for SR-IOV.
   2) What is the priority of each NIC: According to the order of the PFs in the YAML file.
   3) In each NIC - which capabilities are needed.

D) The goal is to find the right NICs on the platform to use with SR-IOV.

E) The default YAML file is:
[sriov_detection_configuration.yml](<https://github.com/smart-edge-open/open-developer-experience-kits/blob/sriov_detection_app/roles/infrastructure/device_sriov_detection/files/sriov_detection_configuration.yml>)
  
F) This application will:

  1) Create a capabilities criteria list according to the configuration YAML file.
  2) Detect available Intels NICs with device id, bus info and support for SR-IOV.
  3) Find capabilities for each NIC.
  4) Comparing those NICs capabilities to the capabilities in the criteria list.
  5) If enough NICs will pass this filtering, print their interface.
  6) Upon completion it output the print in 5) to the relevant ansible variables. The final result is using the detected NICs during the provisioning to configure the SR-IOV.

## Usage

### Application input parameters

1) Name of cofigurations YAML file:

- Only the name of the file and not a path.
- Optional parameter.
- With no use of this parameter the default file will be:
  sriov_detection_configuration.yml.

2) A flag for debug mode: "debug_mode"

- Optional parameter.
- The use of this flag will cause the application to print debug information.
- With no use of this flag the application will print only the final result or unrecoverable errors that will cause the application to exit.

#### How to use the input parameters

Input arguments MUST be one of those options ONLY:

- "go run sriov_detection.go"
- "go run sriov_detection.go debug_mode"
- "go run sriov_detection.go" [CONFIG YAML FILE]
- "go run sriov_detection.go [CONFIG YAML FILE] debug_mode"

### Capabilities supported by the application

Basic capabilities:

1) Device Id
2) Bus Information
3) Driver
4) Number of minimum VFs
5) NUMA location
6) Link state
7) Specific port number
8) Link Speed

Special capabilities:

1) Card affinity capability: supports of PF's couples.
2) PTP support capability.

### Application stages and printings in debug mode for each stage

#### 1) Parsing the configuration YAML file

The printings: `"The criteria list for this EK is:"`

#### 2) Detecting the network devices in the Linux

The application will filter the NICs on the platform according to a list of basic capabilities:
a) Vendor of Intel
b) Have no IPV4 address
c) Have PCI Bus info
d) Suitable for SR-IOV: Support minimum of 1 VF.
e) Have a device Id.
The printing in the top of the stage log will be:

`"STARTING TO DETECT THE NETWORK DEVICES ON THE LINUX: "`

The stage will finish with printing all the detected devices.
The format of the printings in this section:
Date, Time, NIC name at the platform, PCI Bus Information, Device Id, Driver, Link Speed, Number of VFs, Link Sate, NUMA Location, DDP Support, PTP Support, RRU Connection, Specific Port Number, Card Affinity.

Example:

`2022/03/23 18:53:12 THE DEVICES SLICE:`

`2022/03/23 18:53:12 {ens7f0 0000:cc:00.0 0d58 i40e 40000 64 1 1  false false false 0 0000:cc:00.x false false}`

`2022/03/23 18:53:12 {eth2 0000:cc:00.1 0d58 i40e 40000 64 1 1  false false false 1 0000:cc:00.x false false}`

`2022/03/23 18:53:12 {eth3 0000:d0:00.0 0d58 i40e 40000 64 1 1  false false false 0 0000:d0:00.x false false}`

`2022/03/23 18:53:12 {ens7f1 0000:d0:00.1 0d58 i40e 40000 64 1 1  false false false 1 0000:d0:00.x false false}`

`2022/03/23 18:53:12 {eno12399 0000:31:00.0 159b ice 65535 128 0 0  false false false 0 0000:31:00.x false false}`

`2022/03/23 18:53:12 {eno12409 0000:31:00.1 159b ice 65535 128 0 0  false false false 1 0000:31:00.x false false}`

`2022/03/23 18:53:12 {ens5f0 0000:b1:00.0 159b ice 25000 128 1 1  false false false 0 0000:b1:00.x false false}`

`2022/03/23 18:53:12 {ens5f1 0000:b1:00.1 159b ice 25000 128 0 1  false false false 1 0000:b1:00.x false false}`


Notes:

- The Connector Type: This capability is a string and not supported. Since currently its empty string, it can not be seen at the example above.

- Specific Port Number capability: The physical port from the PCI bus info.
  
- The last 2 booleans in this example use for the purpose of debugging only. 

#### 3) Checking the detected devices against the configured criteria lists

There will be no relevant printings at this stage.

### How to edit the configuration YAML file

The file has 2 parts:

1) The criteria list for the detected NICs for SR-IOV.
2) General configuarions.

The deafult YAML file is a basic example: without neither card affinity couples nor special capabilities:

```yml

criteriaLists:
  pf1:
    functionality: APP                     
    deviceId: [158a,0d58,1593,159b,1592,188a]
    numVFSupp: 64
  pf2:
    functionality: APP                      
    deviceId: [158a,0d58,1593,159b,1592,188a]
    numVFSupp: 64
  pf3:
    functionality: APP                     
    deviceId: [158a,0d58,1593,159b,1592,188a]
    numVFSupp: 64
  pf4:
    functionality: APP                     
    deviceId: [158a,0d58,1593,159b,1592,188a]
    numVFSupp: 64

configurations:
  timeoutForFindPTPMaster: 1
```

After parsing this file the application will print:

`2022/03/14 19:50:36 The criteria list for this EK is:`

`2022/03/14 19:50:36 pf1: {APP [158a 0d58 1593 159b 1592 188a] [] 0 64 2 -1 [] false false false -1 }`

`2022/03/14 19:50:36 pf2: {APP [158a 0d58 1593 159b 1592 188a] [] 0 64 2 -1 [] false false false -1 }`

`2022/03/14 19:50:36 pf3: {APP [158a 0d58 1593 159b 1592 188a] [] 0 64 2 -1 [] false false false -1 }`

`2022/03/14 19:50:36 pf4: {APP [158a 0d58 1593 159b 1592 188a] [] 0 64 2 -1 [] false false false -1 }`


#### Strict rules for the YAML file

1) In the criteria lists section each PF must be unique.
2) In the criteria lists section each PF must contain at least 1 device id.
3) 2 sections ONLY: criteriaLists and configurations.
4) In the criteria lists section the fields MUST be ONLY from this list with those types:

```shell
type LinkStateType uint8
const(
LINK_STATE_DOWN LinkStateType = iota // 0
LINK_STATE_UP                        // 1
LINK_STATE_INVALID_VALUE             // 2
)

```

| Type        | Name in the YAML file           | Status / Notes  |
| :-------------: |:-------------:| :-----|
| string      | "functionality" |  |
| Array of string      | "deviceId"      |    |
| Array of string |  "driver"      |     |
| uint64  | "linkSpeed"  | Units: Mb/s  |
| uint32  | "numVFSupp"  |   |
| LinkStateType  | "linkState"  | Invalid value: 2  |
| int8  | "numaLocation"  | Invalid value: -1  |
| Array of string  | "connectorType"  | Currently not supported  |
| bool  | "ddpSupport"  | Currently not supported   |
| bool  | "ptpSupport"  | Depends on the 'configuration' section. Please read further instructions.  |
| bool  | "rruConnection"  | Currently not supported  |
| int8  | "specificPortNumber"  | Invalid value: -1  |
| string  | "cardAffinityPfCouple"  | Have a major affect on the application. Please read further instructions.  |

5) In the configurations section: Currently the application support 1 field ONLY:

| Type  | Name in the YAML file  | Status / Notes  |
| ------------- |:-------------:| :-----|
| int  | "timeoutForFindPTPMaster"  | In milli seconds. The value in production for deterministic success is 21000 = 21 seconds.  |

#### Example of configurations YAML file with card affinity couples and special capability

```yaml

criteriaLists:
  pf1:
    functionality: APP
    deviceId: [158a,0d58,1593,159b,1592,188a]
    numVFSupp: 100
  pf2:
    functionality: APP
    deviceId: [158a,0d58,1593,159b,1592,188a]
    numVFSupp: 100
  pf3:
    functionality: APP
    deviceId: [158a,0d58,1593,159b,1592,188a]
    numVFSupp: 64
  pf4:
    functionality: APP
    deviceId: [158a,0d58,1593,159b,1592,188a]
    numVFSupp: 64
  pf5:
    functionality: APP
    deviceId: [158a,0d58,1593,159b,1592,188a]
    numVFSupp: 64
  pf6:
    functionality: APP
    deviceId: [158a,0d58,1593,159b,1592,188a]
    numVFSupp: 64
    cardAffinityPfCouple: pf3
  pf7:
    functionality: APP
    deviceId: [158a,0d58,1593,159b,1592,188a]
    numVFSupp: 64
    ptpSupport: true
  pf8:
    functionality: APP
    deviceId: [158a,0d58,1593,159b,1592,188a]
    numVFSupp: 64
    cardAffinityPfCouple: pf7

configurations:
  timeoutForFindPTPMaster: 21000

```  

The parsing of the YAML file above by the application will be:

`2022/03/23 14:10:33 THE CRITERIA LISTS ARE:`

`2022/03/23 14:10:33 pf7: {APP [158a 0d58 1593 159b 1592 188a] [] 0 64 2 -1 [] false true false -1 pf8 }`

`2022/03/23 14:10:33 pf8: {APP [158a 0d58 1593 159b 1592 188a] [] 0 64 2 -1 [] false false false -1 pf7 }`

`2022/03/23 14:10:33 pf1: {APP [158a 0d58 1593 159b 1592 188a] [] 0 100 2 -1 [] false false false -1  }`

`2022/03/23 14:10:33 pf2: {APP [158a 0d58 1593 159b 1592 188a] [] 0 100 2 -1 [] false false false -1  }`

`2022/03/23 14:10:33 pf3: {APP [158a 0d58 1593 159b 1592 188a] [] 0 64 2 -1 [] false false false -1 pf6 }`

`2022/03/23 14:10:33 pf4: {APP [158a 0d58 1593 159b 1592 188a] [] 0 64 2 -1 [] false false false -1  }`

`2022/03/23 14:10:33 pf5: {APP [158a 0d58 1593 159b 1592 188a] [] 0 64 2 -1 [] false false false -1  }`

`2022/03/23 14:10:33 pf6: {APP [158a 0d58 1593 159b 1592 188a] [] 0 64 2 -1 [] false false false -1 pf3 }`


#### Rules for configuring card affinity couples in the YAML file

1) The user can not configure the PF to be a couple to itself.
2) The application supports card affinity of ONLY 2 devices.
    Meaning: The user of the application can configure ONLY COUPLES of PFs.
3) The order of the PFs matters: PF1 has the highest priority while PFn has the lowest.

#### Configuring PTP Support in the YAML file

1) The user will need to configure the timeout for this capability in the "configuration" section. The value will be in milli-seconds.
2) The configured timeout that guarantee to detect connection between PTP master to PTP client observed to be 21 seconds (21000 milli-seconds).
Meaning: This action will have significant effect on the application performance. For each NIC detection the application will wait 21 seconds for this capability.  

### Known issues

1) In case of NO use of card affinity couples - the results are deterministic. Meaning: All the runs should have the same NICs detection and selection.
2) In case of using card affinity couples:

- the results may be non deterministic: In different runs there may be different selection of devices.
- There are 2 numeric capabilities that the user configure independently:
     a) Number of VFs.
     b) Link Speed.
  In case of using card affinity couples, the user SHOULD configures those 2 capabilities in the HIGHER priorities PFs.
  Explanation: In order to give the application the highest chance to find devices with the configured requirements, while it has additional requirements for couples constrains from the card affinity configurations.

3) In case that many devices would be detected as PTP supported and there will a general requirement for many NICs (a corner case):
There will be a possiblity that the application will not find enough NICs and return failure.
Explanation: After the application has checked the detected devices against the configured criteria, It will not check again with the devices that were detected to be PTP support (Those devices were saved at differet data structure in the code and had not checked in some cases).
  