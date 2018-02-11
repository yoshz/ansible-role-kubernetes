yoshz.kubernetes
================

This Ansible role installs and configures Kubernetes on Ubuntu 16.04.
With small modifications it is also possible to use this role in CentOS or other distributions.

Installation steps are based on Kelsey Hightower's [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way/)
but doesn't require GCE and adds missing functionality.

The role is mainly created to learn how to install Kubernetes step by step.
Its purpose is not to replace Kubeadm, Kubespray or any other tool.

Kubernetes is installed and configured with the following choices. (See [Usage](#Usage) for the right installation order)

- [X] Installs `etcd`, `kube-apiserver`, `kube-controller-manager`, `kube-scheduler` as systemd service on each master node in high availabity mode
- [X] Installs `kube-proxy` and `kubelet` as systemd service on each worker node
- [X] Uses a bootstrap token to automatically generate client certificates for kubelet
- [X] Configures ufw rules to allow traffic between in each node
- [X] Installs `flannel` as network overlay
- [X] Installs cluster add-ons: `kube-dns`, `heapster` and `kubernetes-dashboard`
- [X] Installs Traefik as ingress controller
- [X] Configures default ingress rules with basic authentication

Todo's
------

- [ ] Configure apiserver load balancing for kubelet and kube-proxy
- [ ] Install logging aggregation
- [ ] Replace Docker with [cri-containerd](https://github.com/kubernetes-incubator/cri-containerd) runtime
- [ ] Configure VPN connection between each node
- [ ] Configure [data encryption](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)

Installation
============

Requirements
------------

Make sure you have installed all dependencies locally:

* Ansible >= 2.4 (`pip install ansible`)
* Python openssl module (`pip install pyOpenSSL`)
* [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* Openssl cli

Install role
------------

Add this role to your project:
```bash
ansible-galaxy install yoshz.kubernetes
```

Or as submodule
```bash
git submodule add git@github.com:yoshz/ansible-role-kubernetes.git roles/yoshz.kubernetes
```

Inventory
---------

Create an inventory file with all your nodes.

```ini
node1 ansible_host=X.X.X.X
node2 ansible_host=X.X.X.X
node3 ansible_host=X.X.X.X

[k8s-node]
node1
node2
node3

[k8s-master]
node1
node2
node3

[etcd]
node1
node2
node3

[kubernetes:children]
k8s-node
k8s-master
```

Put all nodes that should be master in the `k8s-master` group and workers in the `k8s-node` group.

Playbook
--------

Create a playbook with the following content:
```yaml
- hosts: kubernetes
  roles:
  - role: yoshz.kubernetes
    k8s_version: 1.9.0
    # Local location to store generated certificates 
    k8s_certs_src: ../certs
    # Public DNS used as suffix for ingress rules
    k8s_public_dns: example.org
    # Users that can access the dashboards 
    k8s_basic_auth_users:
    - name: admin
      password: admin
```

Docker
------

Make also sure you have a role taking care of installing Docker for example:
```bash
ansible-galaxy install yoshz.docker
```

Or as submodule
```bash
git submodule add git@github.com:yoshz/ansible-role-docker.git roles/yoshz.docker
```

And prepend the role to the playbook:

```yaml
- hosts: k8s-node
  roles:
  - role: yoshz.docker
    docker_version: 1.12.6
    docker_options: --iptables=false --ip-masq=false
```

Bootstrap token
---------------

A bootstrap token is required to for kubelet to join the cluster.

The following command will generate the values `k8s_token_id` and `k8s_token_secret` variables
that you will need to add to your playbook or vars file.

```bash
# k8s_token_id
openssl rand -hex 3
# k8s_token_secret
openssl rand -hex 8
```

Usage
=====

Installation steps
------------------

Each installation step has a different tag which makes it possible to provision your nodes step by step.

To run a specific step:
```bash
ansible-playbook -i [inventory] [playbook] --tags [tag]
```

#### certs-generate
Locally generates the private keys and certificates.
These files will be saved in the `k8s_certs_src` directory.

#### certs-install
Installs the private keys and certificates on each node.

#### kubectl
Installs kubectl and configures kubeconfig for root.

#### etcd
Installs etcd on each master and initialise cluster.

##### kube-apiserver
Installs kube-apiserver on each master as systemd service.

#### kube-controller-manager
Installs kube-controller-manager on each master as systemd service. 

#### kube-scheduler
Installs kube-scheduler on each master as systemd service.

#### bootstrap-token
Installs bootstrap token secret and enable auto approval of certificate signing requests.

#### cni
Installs cni configuration file needed before starting Kubelet.

#### kubelet
Installs kubelet as systemd service on each worker node.

#### kube-proxy
Installs kube-proxy on each node as systemd service.

#### flannel
Installs flannel as DaemonSet.

#### kube-dns
Installs kube-dns as Deployment.

#### heapster
Installs heapster in standalone mode as Deployment.

#### basic-auth
Installs basic-auth secret which is used for basic authentication for Dashboard and Traefik.

#### dashboard
Installs Kubernetes dashboard


Etcd
----

To list all etcd member run the following command on one of the nodes:
```bash
ETCDCTL_API=3 etcdctl member list
```

If the cluster is already initialized and you want to add additional members run:
Make sure you add one member at a time.
```bash
ETCDCTL_API=3 etcdctl member add <node> --peer-urls=https://<ip>:2380 --endpoints=https://<ip>:2380
```

License
=======

MIT

Author Information
==================

Yosh de Vos <yosh@elzorro.nl>
