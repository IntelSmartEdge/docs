```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2022 Intel Corporation
```

# Intel® Smart Edge Open Developer Experience Kit Default Installation - Prepare the Provisioning System 

Before you can install the Intel® Smart Edge Open Developer Experience Kit, you will need to first prepare the provisioning system by installing the following software:

- Git
- Docker
- Docker Compose
- Python

> **Note:** You must be logged in as a root user on the provisioning system to complete the following steps. Run the following command on the provisioning system to login as a root user:\

  ```Shell.bash
    $ sudo su -
  ```

### 1. Install Git

Use the following command to install Git to the target system:

```Shell.bash
# apt-get install git
```

### 2. Install Docker and Docker Compose

Next, you will need to install Docker, configure it to use a proxy server, and install Docker Compose. 

**i.** Install Docker by following the installation instructions in the [Docker repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository). Installation by package manager is not supported and will result in errors.

**ii.** Configure Docker to use a Proxy Server by running the commands below to create or edit the Docker configuration file.

```Shell.bash
#mkdir ~/.docker
```

```Shell.bash
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

**iii.** Set your environment variables. Replace `myproxy.hostname` in the example with the information for your own proxy.

```Shell.bash
# mkdir -p /etc/systemd/system/docker.service.d
```
```Shell.bash
# vi /etc/systemd/system/docker.service.d/proxy.conf
[Service]
Environment="HTTP_PROXY=http://myproxy.hostname:8080"
Environment="HTTPS_PROXY=https://myproxy.hostname:8080/"
Environment="NO_PROXY="localhost,127.0.0.1,::1"
```

**iv.** Reload the file and restart the Docker service.

```Shell.bash
# systemctl daemon-reload
# systemctl restart docker.service
```

**v.** Install Docker Compose

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

### 4. Create AWS t2.medium Instance (Advanced installation only)

If you plan to install the Intel® Smart Edge Open Developer Experience Kit with security features enabled then you will need to create an AWS t2.medium instance by following the instructions below:

1. Create EC2 t2.medium instance (at least) with Ubuntu 20.04 OS image in any geo location.
2. If the region in which instance is being created does not have default VPC configured, Create VPC and subnet with IPV4 CIDR.
3. While creating instance, chose the VPC created in step 2, if no default VPC available.
4. Create Internet gateway and attach to your VPC to allow traffic from internet to reach your instance.
5. Create custom route table for the above created VPC and associate to the subnet created in step 2.
6. At the end of instance creation steps, you will be prompted to generate new key pair or import existing key pairs. Generate key pair and download private key and use it to make SSH connection to the AWS instance.
7. Assign static ip address to AWS instance to prevent disconnect(due to reboot) while deploying ISecL controller.
> Create elastic ip address from Amazon public IPv4 pool.
> Associate elastic ip to your newly created AWS instance. 
8. While connecting to instance from terminals(PuTTY, Xterm, etc), use instance public IP(elastic IP), private key(ppk) and socks5 proxy.
9. ISecL uses following ports to enable inter service communication. All these ports need to be added to security groups of the EC2 instance.
> CMS - 30445
> HVS - 30443
> AAS - 30444
> NATS - 30222
> NFS port - 2049
> SGX PCCS service - 32666
> KMRA AppHSM service - 30500

Review the office [Get started with AWS instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html) documentation for more information about creating your instance.


### 5. Clone the Developer Experience Kit Repository

Clone the [Developer Experience Kit repo](https://github.com/smart-edge-open/open-developer-experience-kits) to the provisioning system:

```Shell.bash
# git clone https://github.com/smart-edge-open/open-developer-experience-kits.git --branch=smart-edge-open-21.12 ~/dek
# cd ~/dek
```

### 6. Generate the Configuration File
Run the following command to generate the `custom.yml` file. It is recommended you generate a new congfiguraion file instead of using the default `default_config.yml` file provided.

```Shell.bash
[Provisioning System] # sudo ./dek_provision.py --init-config > custom.yml
```

### Next

After preparing the provisioning system, you will create the installation image. This is covered in the next section. 
