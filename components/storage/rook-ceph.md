```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2019-2021 Intel Corporation
```
# Rook-Ceph

- [Rook-Ceph](#Rook-Ceph)
  - [Overview](#overview)
    - [Overview of Ceph](#overview-of-ceph)
    - [Overview of Rook operator and CSI](#overview-of-rook-operator-and-csi)
  - [Rook-Ceph configuration and usage](#rook-ceph-configuration-and-usage)
    - [Configuration](#configuration)
      - [Pre-requisites](#pre-requisites)
      - [Ceph Configuration](#ceph-configuration)
      - [Ceph Resource Restraint](#ceph-resource-restraint)
    - [Verify the Ceph OSD creation on the node](#verify-the-ceph-osd-creation-on-the-node)
    - [Usage](#usage)
      - [PVC VM application](#pvc-vm-application)
  - [Limitations](#limitations)
  - [Reference](#reference)

## Overview
Edge applications based on Kubernetes* need to store persistent data, Kubernetes provides a persistent volume provisioning based on local disk directly attached to a single node.
This solution has several shortcomings:

1. Dynamic provisioning is not supported. Dynamic provisioning allows to automatically provision storage when it is requested by users.
2. No data reliability. Only one copy of user data is stored in local node, this causes a single point of failure issue.

Ceph storage is introduced to resolve the problems, to integrate Ceph storage into Kubernetes cluster, we propose Rook-Ceph as storage orchestration solution.

### Overview of Ceph
Ceph is an open-source, highly scalable, distributed storage solution for block storage, shared filesystems, and object storage with years of production deployments. It was born in 2003 as an outcome of Sage Weil’s doctoral dissertation and then released in 2006 under the LGPL license.

Ceph is a prime example of distributed storage, a distributed storage system is composed of many server nodes interconnected through a network to provide storage services externally as a whole.
The benefit of distributed storage are:

- Large capacity and easy expansion
- High reliability. No single point of failure, data security and business continuity are guaranteed.
- High performance. Data shared and enable concurrent access.
- Data management and more features support. The storage cluster can be managed and configured easily, also provide functions such as data encryption/decryption and data deduplication, etc.

Ceph consists of several components:

- MON (Ceph Monitors) are responsible for forming cluster quorums. All the cluster nodes report to MON and share information about every change in their state.
- OSD (Ceph Object Store Devices) are responsible for storing objects and providing access to them over the network.
- MGR (Ceph Manager) provides additional monitoring and interfaces to external management systems.
- RADOS (Reliable Autonomic Distributed Object Stores) is the core of Ceph cluster. RADOS ensures that stored data always remains consistent with data replication, failure detection, and recovery among others.
- LibRADOS is the library used to gain access to RADOS. With support for several programing languages, LibRADOS provides a native interface for RADOS as well as a base for other high-level services, such as RBD, RGW, and CephFS.
- RBD (RADOS Block Device), which are now known as the Ceph block device, provide persistent block storage, which is thin-provisioned, resizable, and stores data striped over multiple OSDs.
- RGW (RADOS Gateway) is an interface that provides object storage service. It uses libRGW (the RGW library) and libRADOS to establish connections with the Ceph object storage among applications. RGW provides RESTful APIs that are compatible with Amazon* S3 and OpenStack* Swift.
- CephFS is the Ceph Filesystem that provides a POSIX-compliant filesystem. CephFS uses the Ceph cluster to store user data.
- MDS (Ceph Metadata Server) keeps track of file hierarchy and stores metadata only for CephFS.

### Overview of Rook operator and CSI
Rook operator is an open-source cloud-native storage orchestrator that transforms storage software into self-managing, self-scaling, and self-healing storage services.

Rook-Ceph is the Rook operator for Ceph, it starts and monitors Ceph daemon pods, such as MONs, OSDs, MGR, and others, Rook-Ceph also monitors the daemon to ensure the cluster is healthy. Ceph MONs are started or failed over when necessary. Other adjustments are made as the cluster grows or shrinks.

Just like native Ceph, Rook-Ceph provides block, filesystem, and object storage for applications.

- Ceph CSI (Container Storage Interface) is a standard for exposing arbitrary block and file storage systems to containerized workloads on Container Orchestration Systems like Kubernetes. Ceph CSI is integrated with Rook and enables two scenarios:
	- RBD (block storage): This driver is optimized for RWO pod access where only one pod may access the storage.
	- CephFS (filesystem): This driver allows for RWX with one or more pods accessing the same storage.
- For object storage, Rook supports the creation of new buckets and access to existing buckets via two custom resources: Object Bucket Claim (OBC) and Object Bucket (OB). Applications can access the objects via RGW.

## Rook-Ceph configuration and usage

To deploy the Rook-Ceph operator and Ceph to the Intel® Smart Edge Open cluster, `rook_ceph_enabled: True` must be set in `/inventory/default/group_vars/all/10-default.yml`, which is disabled by default. This will perform Rook operator and Ceph daemons install and deploy, images for Rook operator and Ceph daemon with CSI plugins are download from public docker repository.

### Configuration

Rook-Ceph operator provide custom resource for CephCluster to configure Ceph parameters by user.

#### Pre-requisites
Currently, the deployment searches for any available raw drive in the node and then consumes it for Ceph. If there are 2 drives, 1 with OS and another drive with
partitions then the second drive with partitions will be wiped out and consumed by Ceph. Also the deployment will wipeout if a Ceph OSD exists and will create a new
Ceph OSD in the 2nd drive. Note that, creating paritions on the drive after deploying Ceph has to be avoided.
If the user wants to manually prepare the drive then he can follow
the steps to reset the drive to a usable state as described in : https://rook.io/docs/rook/v1.7/ceph-teardown.html#zapping-devices
	
#### Ceph Configuration
Specifies Ceph configuration in `inventory/default/group_vars/all/10-default.yml`:

```yaml
rook_ceph_enabled: True

rook_ceph:
  mon_count: 1
  host_name: "{{ hostvars[groups['controller_group'][0]]['ansible_nodename'] }}"
  osds_per_device: "1"
  replica_pool_size: 1
```

#### Ceph Resource Restraint
Since Rook-Ceph daemons deploy in Edge Node, considering that the resource consumption of the ceph daemon may affect the operation of the edge application, the default value of resource restraint for Ceph daemons is defined in `roles/kubernetes/rook_ceph/defaults/main.yml`.

```yaml
_rook_ceph_limits:
  api:
    requests:
      cpu: "500m"
      memory: "512Mi"
    limits:
      cpu: "500m"
      memory: "512Mi"
  mgr:
    requests:
      cpu: "500m"
      memory: "512Mi"
    limits:
      cpu: "500m"
      memory: "512Mi"
  mon:
    requests:
      cpu: "500m"
      memory: "512Mi"
    limits:
      cpu: "500m"
      memory: "512Mi"
  osd:
    requests:
      cpu: "500m"
      memory: "4Gi"
    limits:
      cpu: "500m"
      memory: "4Gi"
```

### Verify the Ceph OSD creation on the node
The user can do the following to verify the Ceph creation on the node by running
   ```bash
	kubectl get pods -n rook-ceph
	NAME                                                       READY   STATUS      RESTARTS      AGE
    csi-cephfsplugin-provisioner-689686b44-rwzpc               6/6     Running     0             40m
	csi-cephfsplugin-zv7hk                                     3/3     Running     0             40m
	csi-rbdplugin-fn56f                                        3/3     Running     0             40m
	csi-rbdplugin-provisioner-5775fb866b-dpph4                 6/6     Running     0             40m
	rook-ceph-crashcollector-silpixa00401187-f98464b49-4pvk4   1/1     Running     0             40m
	rook-ceph-mgr-a-7cb77f9c6c-lx7tl                           1/1     Running     0             40m
	rook-ceph-mon-a-6d647c9546-bpp9r                           1/1     Running     0             40m
	rook-ceph-operator-54655cf4cd-kjbm8                        1/1     Running     0             40m
	rook-ceph-osd-0-7b45dfcc77-5nc95                           1/1     Running     0             40m
	rook-ceph-osd-prepare-silpixa00401187--1-5rs2b             0/1     Completed   0             40m
	rook-ceph-tools-54474cfc96-xrvzw                           1/1     Running     0             40m
   ```

### Usage
After deployment of Rook operator and Ceph daemons, there is an example for VM application using PVC (Ceph block storage) provision by Ceph-CSI.

#### PVC VM application

Create a PVC using the following yaml file
1. The PVC yaml file sample-pvc.yaml contains
```yaml
   apiVersion: cdi.kubevirt.io/v1beta1
   kind: DataVolume
   metadata:
     name: ubuntu-reg-dv
   spec:
     source:
       registry:
         url: "docker://tedezed/ubuntu-container-disk:20.0"
     pvc:
       storageClassName: rook-ceph-block
       accessModes:
         - ReadWriteOnce
       resources:
         requests:
           storage: 10Gi
```

2. Apply the PVC
```bash
   kubectl apply -f sample-pvc.yaml
```

3. Check that the PVC exists - it will take some time to import the image
```bash
   kubectl get pvc
   NAME                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
   ubuntu-reg-dv           Bound    pvc-86050fce-e695-4586-b800-7bc477d9eb03   10Gi       RWO            rook-ceph-block   9s
   ubuntu-reg-dv-scratch   Bound    pvc-702e606b-3c0e-4508-b925-412aee318414   10Gi       RWO            rook-ceph-block   9s
   # kubectl get pods
   NAME                                      READY   STATUS    RESTARTS   AGE
   importer-ubuntu-reg-dv                    1/1     Running   0          54s
   # kubectl get pvc # ubuntu-reg-dv-scratch goes away after it completes image upload
   NAME                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
   ubuntu-reg-dv           Bound    pvc-86050fce-e695-4586-b800-7bc477d9eb03   10Gi       RWO            rook-ceph-block   9s
```

4. Create a yaml file for deploying the VM
```yaml
   apiVersion: kubevirt.io/v1alpha3
   kind: VirtualMachine
   metadata:
     name: testvmceph
   spec:
     running: true
     template:
       metadata:
         labels:
           kubevirt.io/size: small
           kubevirt.io/domain: testvmceph
       spec:
         domain:
           cpu:
             cores: 2
           devices:
             disks:
             - disk:
                 bus: virtio
               name: containervolume
             - disk:
                  bus: virtio
               name: cloudinitvolume
             interfaces:
             - name: default
               bridge: {}
           resources:
             requests:
               memory: 4096M
         networks:
         - name: default
           pod: {}
         volumes:
           - name: containervolume
             persistentVolumeClaim:
               claimName: ubuntu-reg-dv
           - name: cloudinitvolume
             cloudInitNoCloud:
               userData: |-
                 #cloud-config
                 chpasswd:
                   list: |
                     debian:debian
                     root:toor
                   expire: False 
```

5. Deploy the VM
```bash
   kubectl apply -f sample-vm.yaml
```

6. Check that the VM is running
```bash
   # kubectl get vm
   testvmceph            3s    Starting   False
   # kubectl get vmi
   testvmceph            45s   Running   10.245.225.34   silpixa00400489   True
```

7. Log in to the VM and create a file
```bash
   # kubectl virt console  testvmceph (root toor)
   # touch myfileishere.txt   
```

8. Remove the VM
```bash
   # kubectl delete vm testvmceph
```

9. Re-deploy the VM
```bash
   # kubectl apply -f vm.yaml
```
   
10. Log in to the VM and check if the file exists
```bash
	# kubectl console vm testvmceph (root toor)
    # ls
```

## Limitations
There is a limitation for Rook-Ceph ansible role implementation in Smart Edge Open cluster, currently there is only support for single node and single disk deployment for Ceph. So by default, you don't need to set replica_pool_size > 1.

## Reference
For further details:
- Rook github: https://github.com/rook/rook
- Rook Introduction: https://01.org/kubernetes/blogs/tingjie/2020/introduction-cloud-native-storage-orchestrator-rook
- Kubernetes CSI: https://kubernetes-csi.github.io/docs/
