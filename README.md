[![License Apache 2.0](https://img.shields.io/github/license/containerum/element.svg)](https://github.com/containerum/element/blob/master/LICENSE)
[![Twitter Follow](https://img.shields.io/twitter/follow/containerumcom.svg?style=social&label=Follow)](https://twitter.com/Containerumcom)

![Containerum logo](logo.svg)
## What is Containerum Element?
Containerum Element is a set of Ansible scripts for bootstrapping a **minimum** stable Kubernetes cluster. It is based on the certified Kubernetes Distribution by Containerum (read more about KDC at [Containerum website](https://en.containerum.com)).

Containerum Elements installs a Kubernetes cluster with the following components:
 - **[Kubernetes Distribution by Containerum](https://en.containerum.com)** 1.11.6
 - **Cri-o** as container runtime
 - **CoreDNS** as DNS and Service Discovery
 - **Calico** as container network interface
 - **Bird** BGP for routing Internet Protocol packets
 - **Helm** package manager (optional)


Containerum Element installs the latest stable versions of the aforementioned components. KDC version is currently 1.11.6.

## Requirements:
**Local machine**
- Ansible >=2.7
- [Cert Machine](https://github.com/nkmazur/cert_machine)

**Nodes:**
- CentOS 7
- File system should support OverlayFS

Each node should have `centos` user accessible via ssh with permission to execute sudo with no password.

## Quick start
### Set variables

Fork or download the repo and edit ```/ansible/group_vars/all.yaml``` as follows:
```
cloud_external_ip: xxx.xxx.xxx.xxx #external IP of the master node. In case of using several master nodes a floating IP address must be specified.
k8s_api_external_port: 56443 #port to access the Kubernetes API
cloud_internal_ip: 10.16.0.30 #internal IP address of the master node. In case of using several master nodes, a floating IP address must be specified.
SERVICE_CLUSTER_IP_RANGE: 10.96.0.0/16 #standard Kubernetes subnet. Do not change unless VERY necessary.
SERVICE_NODE_PORT_RANGE: "30000-32767" #port range for Kubernetes services
CLUSTER_CIDR: 192.168.0.0/16 #change only if it overlaps with existing services
KUBELET_RUNTIME: remote
KUBELET_RUNTIME_ENDPOINT: "unix:///var/run/crio/crio.sock"
IP_AUTODETECTION_METHOD_NIC: ens160 #name of the internal network interface on virtual machines
ansible_user: centos #user that has access to all machines and can execute sudo with no password

## settings fs
#dev_master_log: /dev/sdb #name of the disk on the master node for storing logs

#dev_slave_log: /dev/sdb #name of the disk on worker nodes for storing logs
#dev_slave_containers: /dev/sdd #name of the disk on worker nodes for storing container temporary data and images

#dev_etcd_log: /dev/sdb #name of the disk on etcd nodes for storing logs
#dev_etcd_data_etcd: /dev/sdc #name of the disk on etcd nodes for storing etcd data


# CA cert env
cert_ca_country: "EU"
cert_ca_organization: "DEMO_ORG"
cert_ca_organization_unit: "DEMO_UNIT"
cert_ca_locality: "DEMO_LOC"
cert_ca_validity_days: 1024
cert_ca_key_size: 4096

#cert env
cert_cluster_name: "Containerum"
cert_validity_days: 365
cert_key_size: 2048

helm_version: 2.12.1 #Helm Version

```


Edit ```/ansible/group_vars/inventory``` as follows:
```
[masters]
demo-m1 ansible_host=192.0.2.2 - master node's hostname and IP address. Here and below nodes' current hostnames will be overridden with the ones specified in this config file.
#etc

[slaves]
demo-s1 ansible_host=192.0.2.3 - worker1 node's hostname and IP address
demo-s2 ansible_host=192.0.2.4 - worker2 node's hostname and IP address
#etc

[etcd]
demo-m1 ansible_host=192.0.2.2 - etcd1 node's hostname and IP address(same as master)
demo-s1 ansible_host=192.0.2.3 - etcd2 node's hostname and IP address (same as same as worker1)
demo-s2 ansible_host=192.0.2.4 - etcd3 node's hostname and IP address (same as same as worker1)
#etc

[local]
localhost ansible_connection=local
```
### Launch installer
After the variables are set, run:

```
ansible-playbook -i inventory element.yaml
```
Done!

To manage the cluster remotely, install ```kubectl``` locally and copy ```.kube/config``` file from the master node to ```.kube/config``` on your local machine.

### Helm installation
To install Helm, run:

```
ansible-playbook -i inventory deploy-app.yaml --tags "app-helm"
```
To manage the cluster remotely, install ```helm``` locally and copy ```.kube/helm``` from the master node to ```.kube/helm``` on your local machine. Then run helm commands with --tls flag.

## Additional notes
During installation Containerum Elements overrides the following config files on each node:

**/etc/resolv.conf:**

```
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1
```

**/etc/hosts:**
```
192.0.2.2 demo-m1
192.0.2.3 demo-s1
192.0.2.4 demo-s2
```
The parameters here are drawn from the ```inventory``` file.

**/etc/hostname:**
```
hostname_from_the_inventory_file
```

## Issues and support
Containerum Element has been tested with:
 - vCloud

If you have found a bug, please open an issue. If you have questions about Containerum Element or Kubernetes Distribution by Containerum, you can join us on [Telegram](https://t.me/containerum).

You will also find lots of useful information on Kubernetes on our [Medium blog](https://medium.com/containerum).


## License
Copyright (c) 2019 Containerum.

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
