```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```

# Onboarding Applications to an Intel® Smart Edge Open Cluster

## Overview

Intel® Smart Edge Open uses Helm Charts to onboard applications to edge node clusters. 

In this guide, you'll use the Helm command-line interface (CLI) to install an example application to an edge cluster. Once you've installed the application, you'll verify it's running on the cluster and then uninstall it.

Our example application is an NGINX web server. You can use these instructions to onboard any application from a Helm Chart.

These instructions apply to clusters created by deploying the [Developer Experience Kit](/experience-kits/developer-experience-kit.md).

### Requirements
#### Hardware
- An Intel® Smart Edge Open cluster that has been deployed by running the [Developer Experience Kit](/experience-kits/developer-experience-kit.md). 

#### Software
The Helm v3 CLI must be installed on the system hosting the cluster. For instructions, see the [Helm documentation](https://helm.sh/docs/intro/install/)

#### Knowledge
Basic knowlege of Kubernetes and Helm. 
- For a list of commonly used `kubectl` commands and flags, see the [`kubectl` Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- For more information on Helm commands, see the [Helm documentation](https://helm.sh/docs/)

### Install the Application 

#### Prepare the System
##### Verify Pods are Running
Before onboarding an application, check the server deployment status and verify that all pods are running:

```shell
$ kubectl get pods -A
NAMESPACE                NAME                                                         READY   STATUS    RESTARTS   AGE
harbor                   harbor-app-chartmuseum-6b78696d68-qcq7l                      1/1     Running   1          31d
harbor                   harbor-app-core-5ccc7dd7b7-gm7vz                             1/1     Running   4          31d
harbor                   harbor-app-database-0                                        1/1     Running   1          31d
harbor                   harbor-app-jobservice-5dc5c7cd67-b9j2w                       1/1     Running   7          31d
harbor                   harbor-app-nginx-65dfc4d4b9-8ngk8                            1/1     Running   2          31d
harbor                   harbor-app-notary-server-697cf7b747-khnhl                    1/1     Running   1          31d
harbor                   harbor-app-notary-signer-59957c8b64-6t9pt                    1/1     Running   1          31d
harbor                   harbor-app-portal-88895f59f-2srwb                            1/1     Running   1          31d
harbor                   harbor-app-redis-0                                           1/1     Running   1          31d
harbor                   harbor-app-registry-6bf6c79db8-ks7c8                         2/2     Running   2          31d
harbor                   harbor-app-trivy-0                                           1/1     Running   1          31d
kube-system              calico-kube-controllers-6b59cd85f8-jqcz4                     1/1     Running   1          31d
kube-system              calico-node-jxbl8                                            1/1     Running   1          31d
kube-system              coredns-558bd4d5db-8pbd6                                     1/1     Running   1          31d
kube-system              coredns-558bd4d5db-r4khd                                     1/1     Running   1          31d
kube-system              etcd-intel                                                   1/1     Running   1          31d
kube-system              kube-apiserver-intel                                         1/1     Running   1          31d
kube-system              kube-controller-manager-intel                                1/1     Running   1          31d
kube-system              kube-multus-ds-amd64-89dmz                                   1/1     Running   1          30d
kube-system              kube-proxy-f72h9                                             1/1     Running   1          31d
kube-system              kube-scheduler-intel                                         1/1     Running   1          31d
smartedge-system         nfd-release-node-feature-discovery-master-854696fc76-2wwgh   1/1     Running   1          30d
smartedge-system         nfd-release-node-feature-discovery-worker-jx9t8              1/1     Running   1          30d
sriov-network-operator   network-resources-injector-c5cqk                             1/1     Running   1          30d
sriov-network-operator   operator-webhook-flrbx                                       1/1     Running   1          30d
sriov-network-operator   sriov-network-config-daemon-rn9tg                            3/3     Running   3          30d
sriov-network-operator   sriov-network-operator-57f857f94c-b8zdh                      1/1     Running   1          30d
telemetry                cadvisor-vvrk8                                               2/2     Running   2          30d
telemetry                collectd-jjld8                                               3/3     Running   3          30d
telemetry                grafana-756ffcb68c-x62nt                                     3/3     Running   3          30d
telemetry                prometheus-node-exporter-vqhgf                               1/1     Running   1          30d
telemetry                prometheus-server-5b469cc688-ft85j                           3/3     Running   3          30d
telemetry                statsd-exporter-cfbd5bf65-j2ppt                              2/2     Running   2          30d
```

<!-- * Prepare your target system. -->
##### Add the Repository
Log into the server.

```shell
ssh smartedge-open@<server-ip-address>
```
Add the NGINX Helm repository. This will download the Helm Chart from the registry to deploy the example application.
> NOTE: To install an application using a Helm Chart that is not available from the registry, you can untar the chart's tarball locally to use for application deployment.

```shell
$ helm repo add nginx https://helm.nginx.com/stable

# Typical stdout
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
nginx/nginx-ingress     0.10.0          1.12.0          NGINX Ingress Controller
stable/nginx-ingress    1.41.3          v0.34.1         DEPRECATED! An nginx Ingress controller that us...
stable/nginx-lego       0.3.1                           Chart for nginx-ingress-controller and kube-lego
```

##### Install the Helm Chart
<!-- * Check the applications. -->
Use `helm search` to find charts in the repository you added. In the example below, we're searching for the chart called `nginx-ingress`. 

```shell
$ helm search repo nginx-ingress

# Typical stdout
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
nginx/nginx-ingress     0.10.0          1.12.0          NGINX Ingress Controller
stable/nginx-ingress    1.41.3          v0.34.1         DEPRECATED! An nginx Ingress controller that us...
stable/nginx-lego       0.3.1                           Chart for nginx-ingress-controller and kube-lego
```

<!-- * Complete the installation steps. -->
Use `helm install` to install the chart. In the example below, we set the chart's release name to `my-ingress-controller`. Once we set it, we'll be able refer to the chart by its release name at the CLI.

```shell
$ helm install my-ingress-controller nginx/nginx-ingress

#Typical stdout
NAME: my-ingress-controller
LAST DEPLOYED: Tue Sep  7 10:39:24 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The NGINX Ingress Controller has been installed.
```
### Verify the Installation Status
**Option 1:** Use the `helm status` command to check the status of the application installation. From this point on, we refer to the chart by its release name, `my-ingress-controller`.

```shell
$ helm status my-ingress-controller

#Typical stdout
NAME: my-ingress-controller
LAST DEPLOYED: Tue Sep  7 10:39:24 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The NGINX Ingress Controller has been installed.
```
**Option 2:** Alternatively you can use the `kubectl` command to check the deployment of the ingress controller in the default namespace.

```shell
$ kubectl get deployments

#Typical stdout
NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
my-ingress-controller-nginx-ingress   0/1     1            0           7s
```

### Uninstall the Application 

To remove the application, use the `helm uninstall` command. 

```shell
$ helm uninstall my-ingress-controller

# Typical stdout
release "my-ingress-controller" uninstalled
```

### Cleanup
The uninstall steps above will perform the cleanup of the application. If you do not require the Docker container images, you can remove them now.

Below is the command to check if a container image still exists:

```shell
$ docker images | grep nginx-ingress

# Typical stdout
nginx/nginx-ingress                                      1.12.0         411b997b75dd   2 months ago    175MB
docker images | grep nginx-ingress

docker rmi -f <docker-image-id>

# Typical stdout
Untagged: nginx/nginx-ingress:1.12.0
Untagged: nginx/nginx-ingress@sha256:a8a89dea468022c4d6e38c2ddaec649fb2bcd4bdda1ecbcd9d08cc846d7df483
Deleted: sha256:411b997b75ddb834af0c5b8b3a71298df624a9136f7b51e77adfed68c20b3884
Deleted: sha256:de053c75b648cc064ef402f739ce77c5fa6c70643c409290c5a09f4ebb6c6da6
Deleted: sha256:2b87bb2095fa239af067093cf8a7911ff42762a670a6f593af136626586598d8
Deleted: sha256:277bbd45d69b05bc14190d4c426bf7ecea9c61cca3fff92d56e7f1f63dc2f4d6
Deleted: sha256:be1bd68b26a410f136652f2347ed04cf1f38631881b7bbe1a04d5ddf7e589262
Deleted: sha256:2855bbcefcf95050e64049447e99e77efa2bff32374e586982d69be4612467ce
Deleted: sha256:bad169ad8b30eab551acbb8cd8fbdcd824528189e3dd0cc52dd88a37bbf121cd
Deleted: sha256:36d83ebf5fec7ae1be4c431f0945f2dbe6828ecdc936c604daa48f17c0b50ed7
Deleted: sha256:b4c9a251dc81d52dd1cca9b4c69ca9e4db602a9a7974019f212846577f739699
Deleted: sha256:038ca5b801cea48e9f40f6ffb4cda61a2fe0b6b0f378a7434a0d39d2575a4082
Deleted: sha256:764055ebc9a7a290b64d17cf9ea550f1099c202d83795aa967428ebdf335c9f7

# Execute the command again to check if the containers are deleted
$ docker images | grep nginx-ingress

#Typical output is - None
```

### Troubleshooting

<!-- In this section, provide troubleshooting information for any likely issues and a link to support.-->

- Make sure you have an active internet connection during the full installation. If you lose Internet connectivity at any time, the installation might fail.

- Make sure you are using a fresh installation of the Developer Experience Kit. Earlier software, especially Docker* and Docker Compose*, can cause issues. 

- If the server hosting the cluster accesses the Internet from behind a proxy, ensure the following proxy environment variables are set properly set:

```shell
# ensure the correct proxy settings in enviroment file
vi /etc/environment

http_proxy=http://<proxy-server-addres>:<port>
https_proxy=http://<proxy-server-addres>:<port>
ftp_proxy=http://<proxy-server-addres>:<port>
```
- If your pod reports a status of `CrashLoopBackOff`, check your Docker account plan. Docker Hub limits the number of pulls available to users with certain account types. If you are pull limited, consider upgrading your Docker account to increase the number of pulls you are allowed per day.
 
```shell
docker login <docker account>

```

If you're unable to resolve your issues, you can open an [issue](https://github.com/smart-edge-open/open-developer-experience-kits/issues) on GitHub.

### Summary

In this guide, you learned how to install and uninstall an application on a single-node cluster created by deploying the [Developer Experience Kit](/experience-kits/developer-experience-kit.md). 
