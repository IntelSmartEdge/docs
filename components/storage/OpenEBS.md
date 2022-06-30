```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2022 Intel Corporation
```

# OpenEBS

- [OpenEBS](#openebs)
  - [Overview](#overview)
    - [Overview of OpenEBS](#overview-of-openebs)
      - [OpenEBS Local PV](#openebs-local-pv)
    - [Node Disk manager (NDM)](#node-disk-manager-ndm)
  - [OpenEBS configuration and usage](#openebs-configuration-and-usage)
    - [Configuration](#configuration)
      - [Pre-requisites](#pre-requisites)
      - [Verify OpenEBS](#verify-openebs)
      - [PVC creation](#pvc-creation)
  - [Reference](#reference)

## Overview

OpenEBS is an easy to use open-source storage solution for Kubernetes.


### Overview of OpenEBS
OpenEBS easily deploys Kubernetes Stateful Workloads that require fast and highly reliable container attached storage. OpenEBS turns any storage available on the Kubernetes worker nodes into local or distributed Kubernetes Persistent Volumes.

It provides a low overhead solution that Supports local sotrage such as LVM LocalPV, Rawfile LocalPV, Partition Local PV and Hostpath LocalPV, Device LocalPV and ZFS LocalPV. 

OpenEBS itself is deployed as just another container and enables storage services that can be designated on a per pod, application, cluster or container level.

#### OpenEBS Local PV
OpenEBS provides Dynamic PV provisioners for Kubernetes Local Volumes. A local volume implies that storage is available only from a single node. A local volume represents a mounted local storage device such as a disk, partition or directory.

OpenEBS used lvm-localpv for this applicaiton. This is and OpenEBS CSI driver for provisioning Local PVs backed by LVM.

> Note: Local PV LVM is the Default used in in the case of SmartEdge deployment.

### Node Disk manager (NDM)

Node-disk-manager (NDM) aims to make it easy to manage the disks attached to the node. It treats disks as resources that need to be monitored and managed just like other resources like CPU, Memory and Network. It contains a daemon which runs on each node, detects attached disks and loads them as BlockDevice objects (custom resource) into Kubernetes.

NDM has 2 main components:
- `node-disk-manager daemonset`, which runs on each node and is responsible for device detection.
  
- `node-disk-operator deployment`, which acts as an inventory of block devices in the cluster.
  
and 2 optional components:

- `ndm-cluster-exporter deployment`, which fetches block device object from etcd and exposes it as prometheus metrics.
- `ndm-node-exporter daemonset`, which runs on each node, queries the disk for details like SMART and expose it as prometheus metrics.

## OpenEBS configuration and usage
To deploy the OpenEBS operator to the Intel® Smart Edge Open cluster, `openebs_enabled: True` must be set in `/inventory/default/group_vars/all/10-default.yml`, which is disabled by default.

> Note: For OpenEBS to work it is expected that two SSDs are present in the platform.

### Configuration

#### Pre-requisites
OpenEBS reqires two SSDs on the node. The deployment currently runs and wipes the suitable drive that does not contiain the OS. This drive is then selected and the Persistant volume (PV) and Volume group (VG) are created through Ansible. Finally a LVM storage class is created.
 
#### Verify OpenEBS
After deployment, user can run the following commands to verify OpenEBS creation
```shell
[smartedge-open@controller ~]# kubectl get pods -A | grep openebs
openebs                  openebs-localpv-provisioner-b5467787f-2hjn5                  1/1     Running
openebs                  openebs-lvm-localpv-controller-0                             5/5     Running
openebs                  openebs-lvm-localpv-node-rxdbn                               2/2     Running
openebs                  openebs-ndm-cluster-exporter-7498fcc659-84npw                1/1     Running
openebs                  openebs-ndm-node-exporter-7m22t                              1/1     Running
openebs                  openebs-ndm-operator-69b5fcf66b-hzg5d                        1/1     Running
openebs                  openebs-ndm-wwllt                                            1/1     Running
```
 Check that the assigned drive (in this case `sdb`) has `lvmvg` created
```shell
[smartedge-open@controller ~]# sudo pvs
PV         VG                                        Fmt  Attr PSize   PFree
  /dev/sda3  ubuntu-vg                                 lvm2 a--  <1.75t 986.99g
  /dev/sdb   lvmvg                                     lvm2 a--  232.88g 212.88g
```

Check that storage classes for OpenEBS are created.
```shell
[smartedge-open@controller ~]# kubectl get sc
NAME                   PROVISIONER            RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-device         openebs.io/local       Delete          WaitForFirstConsumer   false                  10m
openebs-hostpath       openebs.io/local       Delete          WaitForFirstConsumer   false                  10m
openebs-lvmpv          local.csi.openebs.io   Delete          Immediate              false                  10m
openebs-lvmpv-shared   local.csi.openebs.io   Delete          Immediate              false                  10m
```

> Note: "openebs-lvmpv" is the default storage class for the cluster. This is for one application only to use the PVC
> "openebs-lvmpv-shared" is used for multiple applications that use the same PVC.

#### PVC creation 

"A PersistentVolumeClaim (PVC) is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources. Pods can request specific levels of resources (CPU and Memory). Claims can request specific size and access modes."

Create a PVC `sample-pvc.yaml`

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: csi-lvmpv
  namespace: smartedge-apps
spec:
  storageClassName: openebs-lvmpv
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
>Note: use "openebs-lvmpv-shared" for multiple applications. 

Apply the created PVC 
```shell
[root@controller ~]# kubectl apply -f sample-pvc.yaml
```
Check whether the PVC is created
```shell
[root@controller ~]# kubectl get pvc -A
NAMESPACE          NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
smartedge-apps     csi-lvmpv            Bound    pvc-a6789f30-b765-56c9-b21c-57939cd3f450   1Gi      RWO            openebs-lvmpv   63s
```

Sample PVC Pod  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-device-pod
spec:
  volumes:
  - name: local-storage
    persistentVolumeClaim:
        claimName: csi-lvmpv
  containers:
  - name: hello-container
    image: busybox
    command:
       - sh
       - -c
       - 'while true; do echo "`date` [`hostname`] Hello from OpenEBS Local PV." >> /mnt/store/greet.txt; sleep $(($RANDOM % 5 + 300)); done'
    volumeMounts:
    - mountPath: /mnt/store
      name: local-storage
```
## Reference
For further details:
- OpenEBS Documentation: https://openebs.io/docs
- OpenEBS Github: https://github.com/openebs/openebs
- NDM Github: https://github.com/openebs/node-disk-manager
- OpenEBS lvm-localpv: https://github.com/openebs/lvm-localpv