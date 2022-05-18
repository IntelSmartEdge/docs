```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021-2022 Intel Corporation
```

# KubeVirt

## Overview

KubeVirt is an open-source project extending Kubernetes\* with a Virtual Machine (VM) support via Custom Resource Definitions (CRDs) and easy to deploy KubeVirt agents and controllers. KubeVirt addresses a need to allow non-containerizable applications/workloads inside VMs to be treated as K8s managed workloads. This allows for both VM and container/pod applications to coexist within a shared K8s environment, allowing for communication between the K8s pods, VMs, and services on the same cluster.

## How It Works

KubeVirt provides a command line utility (`virtctl`) which allows for management of a VM (create, start, stop, etc.). Additionally, it provides a Containerized Data Importer (CDI) utility, which allows for loading existing `qcow2` VM images (and other data) into the created VM. Support for the K8s Container Network Interface (CNI) plugin Multus\* and SRIOV is also provided which allows users to attach Network Interface Card (NICs) and Virtual Functions (VFs) to a deployed VM. More information about KubeVirt can be found on the [official website](https://kubevirt.io/) and [GitHub\*](https://github.com/kubevirt/kubevirt).

### Stateless vs Stateful VMs

The types of VM deployments can be split into two categories based on the storage required by the workload.
* **Stateless** VMs are backed by ephemeral storage, meaning that the data will disappear with VM restart/reboot.
* **Stateful** VMs are backed by persistent storage, meaning that data will persist after VM restart/reboot.
The type of storage required will be determined based on the use case. In Smart Edge Open, support for both types of VM is available with the aid of KubeVirt.

#### VMs with ephemeral storage

These are VMs with ephemeral storage, such as ContainerDisk storage that would be similarly deployed to ordinary container pods. The data contained in the VM would be erased on each VM deletion/restart. Thus, it is suitable for stateless applications running inside the pods. A better fit for such an application would be running the workload in a container pod but for various reasons (e.g., a legacy application), users may not want to containerize their workload. From an K8s/Smart Edge Open perspective, the advantage of this deployment is no additional storage configuration; users only need to have a cluster with KubeVirt deployed and a dockerized version of a VM image to spin up the VM.

#### VMs with persistent Local Storage

Some VMs require persistent storage, and the data for this kind of VM stays persistent between restarts of the VM.

In the case of persistent storage, it is recommended to use storage provided by Rook-Ceph (detail description is provided in [Rook-Ceph component description document](../storage/rook-ceph.md)). A DataVolume must be created by a user before creating VM. As a result of creating DataVolume Persistent Volume Claim (PVC) is created and Persistent Volume (PV) is populated with data sources.

## How To

### Enable Kubevirt in Smart Edge Open

The KubeVirt role responsible for bringing up KubeVirt components is enabled by default in the Smart Edge via Ansible\* automation. The following is a complete list of steps to bring up all components related to VM support. VM support also requires Virtualization and VT-d to be enabled in the BIOS of the Edge Node.

KubeVirt is deployed by default both in Intel® Smart Edge Node software v6 and Secure Access Service Edge (SASE). Default SR-IOV interfaces are created and bound to vfio-pci by [SR-IOV Network Operator](../networking/sriov-network-operator.md). Storage provisioning is done by [Rook-Ceph](../storage/rook-ceph.md).

### Validate Kubevirt in Smart Edge Open cluster

On successful deployment, the following pods will be in a running state:
```shell
    # kubectl get pods -n kubevirt

    kubevirt      virt-api-684fdfbd57-9zwt4                 1/1     Running
    kubevirt      virt-api-684fdfbd57-nctfx                 1/1     Running
    kubevirt      virt-controller-64db8cd74c-cn44r          1/1     Running
    kubevirt      virt-controller-64db8cd74c-dbslw          1/1     Running
    kubevirt      virt-handler-jdsdx                        1/1     Running
    kubevirt      virt-operator-c5cbfb9ff-9957z             1/1     Running
    kubevirt      virt-operator-c5cbfb9ff-dj5zj             1/1     Running

    [root@controller ~]# kubectl get pods -n cdi

    cdi           cdi-apiserver-5f6457f4cb-8m9cb            1/1     Running
    cdi           cdi-deployment-db8c54f8d-t4559            1/1     Running
    cdi           cdi-operator-7796c886c5-sfmsb             1/1     Running
    cdi           cdi-uploadproxy-556bf8d455-f8hn4          1/1     Running
```

### Create Docker image for stateless VM

Create a `Dockerfile` and place the VM image in the same directory and then build the Docker image per the example below:

```yaml
#Dockerfile
FROM scratch
ADD CentOS-7-x86_64-GenericCloud.qcow2 /disk/
```
```shell
docker build -t centosimage:1.0 .
```

As a result the VM image will be wrapped inside the Docker image.

### Deploy stateless VM on the cluster

Prepare file: statelessVM.yaml
```yaml
apiVersion: kubevirt.io/v1alpha3
kind: VirtualMachine
metadata:
  name: cirros-stateless-vm
spec:
  running: false
  template:
    metadata:
      labels:
        kubevirt.io/domain: cirros
    spec:
      domain:
        cpu:
          cores: 2
        devices:
          disks:
          - disk:
              bus: virtio
            name: rootfs
          - disk:
              bus: virtio
            name: cloudinit
          interfaces:
          - name: default
            bridge: {}
        resources:
          requests:
            memory: 2056M
      networks:
      - name: default
        pod: {}
      volumes:
      - name: rootfs
        containerDisk:
          image: kubevirt/cirros-registry-disk-demo
      - name: cloudinit
        cloudInitNoCloud:
          userDataBase64: SGkuXG4=
```

Execute following steps:

  1. Deploy the VM:
      ```shell
      # kubectl create -f statelessVM.yaml
      ```
  2. Start the VM:
      ```shell
      # kubectl virt start cirros-stateless-vm
      ```
  3. Check that the VM pod got deployed and the VM is deployed:
      ```shell
      # kubectl get pods | grep launcher
      # kubectl get vms
      ```
  4. Execute into the VM (login/pass cirros/gocubsgo):
      ```shell
      # kubectl virt console cirros-stateless-vm
      ```
  5. Check that IP address of Smart Edge Open/K8s overlay network is available in the VM:
      ```shell
      [root@vm ~]# ip addr
      ```

### Deploy stateful VM

To deploy a sample stateful VM with persistent storage refer to [Rook-Ceph example deployment of VM application](../storage/rook-ceph.md#pvc-vm-application).

#### Uploading QCOW2 image to Persistent Volume through CDI on the Cluster:

### Stateful VM deployment
To deploy a sample stateful VM locally through CDI, with persistent storage and additionally use a Generic Cloud CentOS\* image, which requires users to initially log in with ssh key instead of login/password over ssh:

  1. Download the Generic Cloud qcow image for CentOS 7:
      ```shell
      # wget https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud-2003.qcow2
      ```

  2. Get the address of the CDI upload proxy:
      ```shell
      # kubectl get services -A | grep cdi-uploadproxy
      ```
      
  3. Create and upload the image to PVC via CDI:
      ```shell
      [root@controller ~]# kubectl virt image-upload dv centos-dv --image-path=./CentOS-7-x86_64-GenericCloud-2003.qcow2 --insecure --size=15Gi --storage-class=rook-ceph-block --uploadproxy-url=https://<cdi-proxy-ip>:443

      DataVolume default/centos-dv created
      Waiting for PVC centos-dv upload pod to be ready...
      Pod now ready
      Uploading data to https://<cdi-proxy-ip>:443

      898.75 MiB / 898.75 MiB [======================================================================================================================================================================================] 100.00% 15s

      Uploading data completed successfully, waiting for processing to complete, you can hit ctrl-c without interrupting the progress
      Processing completed successfully
      Uploading /root/kubevirt/CentOS-7-x86_64-GenericCloud-2003.qcow2 completed successfully
      ```

  4. Check that DV, and PVC are correctly created:
      ```shell
      [root@controller ~]# kubectl get dv
      centos-dv   Succeeded              105s
      [root@controller ~]# kubectl get pvc
      centos-dv   Bound    kv-pv0      15Gi       RWO            rook-ceph-block   109s
      ```

  5. Create the ssh key:
      ```shell
      [root@controller ~]# ssh-keygen
      ```

  6. Get the controllers public key:
      ```shell
      [root@controller ~]# cat ~/.ssh/id_rsa.pub
      ```

  7. Edit the cdi-centos-vm.yaml file for the VM with the updated public key:
      ```yaml
      apiVersion: kubevirt.io/v1alpha3
      kind: VirtualMachine
      metadata:
        name: centos-vm
      spec:
        running: false
        template:
          metadata:
            labels:
              kubevirt.io/domain: centos
          spec:
            domain:
              cpu:
                cores: 2
              devices:
                disks:
                - disk:
                    bus: virtio
                  name: rootfs
                - disk:
                    bus: virtio
                  name: cloudinit
                interfaces:
                - name: default
                  bridge: {}
              resources:
                requests:
                  memory: 2056M
            networks:
            - name: default
              pod: {}
            volumes:
              - name: rootfs
                persistentVolumeClaim:
                  claimName: centos-dv
              - name: cloudinit
                cloudInitNoCloud:
                  userData: |-
                    #cloud-config
                    package_upgrade: true
                    packages:
                      - git
                      - net-tools
                      - tcpdump
                      - vim
                    write_files:
                      - content: |
                        owner: root:root
                        path: /etc/sysconfig/64bit_strstr_via_64bit_strstr_sse2_unaligned
                    users:
                      - name: root
                        password: root
                        sudo: ALL=(ALL) NOPASSWD:ALL
                        ssh_authorized_keys:
                          - ssh-rsa <controller-public-key> <user>@<node>
      ```

  8.  Deploy the VM:
      ```shell
      [root@controller ~]# kubectl apply -f cdi-centos-vm.yaml
      ```

  9.  Start the VM:
      ```shell
      [root@controller ~]# kubectl virt start centos-vm
      ```

  10. Check that the VM container has deployed:
      ```shell
      [root@controller ~]# kubectl get pods | grep virt-launcher-centos
      ```

  11. Get the IP address of the VM:
      ```shell
      [root@controller ~]# kubectl get vmi
      ```

>**NOTE**: The user should verify that there is no K8s network policy that would block the traffic to the VM (ie. `block-all-ingress policy`). If such policy exists it should be either removed or a new policy should be created to allow traffic. To check current network policies run: `kubectl get networkpolicy -A`. See K8s [documentation for more information on network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/).

  13. SSH into the VM:
      ```shell
      [root@controller ~]# ssh <vm_ip>
      ```

### Deploy VM with SRIOV NIC support

To deploy a VM requesting SRIOV VF of NIC:

  1. Check that the SRIOV VF bound to vfio-pci is available as an allocatable resource for DP:
     ```shell
     # kubectl get node <node-name> -o json | jq '.status.allocatable'
     {
      "cpu": "46",
      "devices.kubevirt.io/kvm": "110",
      "devices.kubevirt.io/tun": "110",
      "devices.kubevirt.io/vhost-net": "110",
      "ephemeral-storage": "189274027310",
      "hugepages-1Gi": "0",
      "hugepages-2Mi": "0",
      "intel.com/sriov-netdev-net-c0p0": "4",
      "intel.com/sriov-vfiopci-net-c0p1": "4", <- This interface
      "intel.com/sriov-netdev-net-c1p0": "4",
      "intel.com/sriov_vfiopci_net_c1p1": "4",
      "memory": "65502980Ki",
      "pods": "110"
     }
     ```

  2. Check that the SRIOV Network Attachement Definition for this interface is available:
     ```shell
     # kubectl get net-attach-def -n smartedge-apps
     sriov-vfio-network-c0p1
     ```
    
  3. Deploy the VM requesting the SRIOV device (if a smaller amount is available on the platform, adjust the number of HugePages required in the .yaml file).

     Create sriov-vm.yml file:

     ```yaml
      apiVersion: kubevirt.io/v1alpha3
      kind: VirtualMachine
      metadata:
        name: testvmsriov1
        namespace: smartedge-apps
      spec:
        running: true
        template:
          metadata:
            labels:
              kubevirt.io/size: small
              kubevirt.io/domain: testvmsriov1
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
                - name: sriov1
                  sriov: {}
              resources:
                requests:
                  memory: 2056M
            networks:
            - name: default
              pod: {}
            - multus:
                networkName: sriov-vfio-network-c0p1
              name: sriov1
            volumes:
              - name: containervolume
                containerDisk:
                  image: tedezed/ubuntu-container-disk:20.0
              - name: cloudinitvolume
                cloudInitNoCloud:
                  userData: |-
                    #cloud-config
                    users:
                      - name: root
                    password: toor
                    chpasswd:
                      expire: False
                      list: |-
                        root:toor
     ```

     Deploy using:

     ```shell
     # kubectl apply -f sriov-vm.yml
     ```

  4. Execute into the VM (login/pass root/toor)(It may take few minutes for the VM to start up):
     ```shell
     [root@controller ~]# kubectl virt console testvmsriov1 -n smartedge-apps
     ```
  5. Fix the networking in the VM for Eth1:
     ```shell
     [root@vm ~]# vim /etc/netplan/50-cloud-init.yaml
        network:
           ethernets:
               enp1s0:
                   dhcp4: true
                   match:
                       macaddress: 72:7f:6f:60:f5:54
                   set-name: enp1s0
               enp6s0:
                   dhcp4: no
                   dhcp6: no
                   addresses: [192.10.10.10/24]
                   gateway4: 192.10.10.1
                   nameservers:
                       addresses: [192.10.10.1, 1.1.1.1]
       
           version: 2

     #restart networking service
     [root@vm ~]# netplan generate
     [root@vm ~]# netplan apply
     ```

  6.  Check that the SRIOV interface has an assigned IP address:
      ```shell
      root@testvmsriov1:~# ip addr
      1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
          inet 127.0.0.1/8 scope host lo
            valid_lft forever preferred_lft forever
          inet6 ::1/128 scope host
            valid_lft forever preferred_lft forever
      2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1480 qdisc fq_codel state UP group default qlen 1000
          link/ether 72:7f:6f:60:f5:54 brd ff:ff:ff:ff:ff:ff
          inet 10.245.127.108/32 scope global dynamic enp1s0
            valid_lft 86313599sec preferred_lft 86313599sec
          inet6 fe80::707f:6fff:fe60:f554/64 scope link
            valid_lft forever preferred_lft forever
      3: enp6s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
          link/ether 72:f5:6c:c0:98:80 brd ff:ff:ff:ff:ff:ff
          inet 192.10.10.10/24 brd 192.10.10.255 scope global enp6s0
            valid_lft forever preferred_lft forever
          inet6 fe80::70f5:6cff:fec0:9880/64 scope link tentative
            valid_lft forever preferred_lft forever
      ```

## Useful Commands and Troubleshooting

### Commands

```
kubectl get pv                  # get Persistent Volumes
kubectl get pvc                 # get Persistent Volume Claims
kubectl get dv                  # get Data Volume
kubectl get sc                  # get Storage Classes
kubectl get vms                 # get VM state
kubectl get vmi                 # get VM IP
kubectl virt start <vm_name>    # start VM
kubectl virt restart <vm_name>  # restart VM
kubectl virt stop <vm_name>     # stop VM
kubectl virt pause <vm_name>    # pause VM
kubectl virt console <vm_name>  # Get console connection to VM
kubectl virt help               # See info about rest of virtctl commands
```

### Troubleshooting

1. The PVC image is not being uploaded through CDI.
Check that the IP address of the `cdi-upload-proxy` is correct and that the Network Traffic policy for CDI is applied:
   ```shell
   kubectl get services -A | grep cdi-uploadproxy
   kubectl get networkpolicy | grep cdi-upload-proxy-policy
   ```

2. Cannot SSH to stateful VM with Cloud Generic Image due to the public key being denied.
Confirm that the public key provided in VM yaml file is valid and in a correct format. Example of a correct format:
   ```yaml
   users:
         - name: root
           password: root
           sudo: ALL=(ALL) NOPASSWD:ALL
           ssh_authorized_keys:
             - ssh-rsa Askdfjdiisd?-SomeLongSHAkey-?dishdxxxx root@controller
   ```
3. Completely deleting a stateful VM.
Delete VM, DV, PV, PVC, and the Virtual Disk related to VM from the Edge Node: 
<!-- Update for storage usage needed -->
   ```shell
   # kubectl delete vm <vm_name>
   # kubectl delete dv <dv_name>
   # kubectl delete pv <pv_names>
   ```

## References

- [KubeVirt](https://kubevirt.io/)
- [KubeVirt components](https://github.com/kubevirt/kubevirt/blob/master/docs/components.md)
- [Containerized Data Importer](https://github.com/kubevirt/containerized-data-importer/blob/master/README.md)
