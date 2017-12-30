yoshz.kubernetes
================

This repository holds an Ansible role to install and configure Kubernetes.
Installation steps are based on Kelsey Hightower's [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way/)
but includes extra steps for non GCE/AWS cloud solutions.

The role is mainly created to learn how to install Kubernetes step by step.
Its purpose is not to replace Kubeadm, Kubespray or any other tool.

Kubernetes is installed and configured with the following choices. (See [Usage](#Usage) for the right installation order)

- [X] Install `etcd`, `kube-apiserver`, `kube-controller-manager`, `kube-scheduler` as systemd service on each master node in high availabity mode. 
- [X] Install `kube-proxy` and `kubelet` as systemd service on each worker node.
- [X] Configure ufw rules to allow traffic between in each node
- [X] Installs `flannel` as network overlay
- [X] Installs cluster add-ons: `kube-dns`, `heapster`, `dashboard`

Todo's
------

- [ ] Replace Docker with [cri-containerd](https://github.com/kubernetes-incubator/cri-containerd) runtime
- [ ] Configure Webhook authentication for Kubelet
- [ ] Configure VPN connection between each node
- [ ] Install ingress controller

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
    k8s_certs_src: ../certs
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

Usage
=====

Each installation has a different tag which makes it possible to provision your nodes step by step.

### certs-generate

Locally generates the private keys and certificates.
These files will be saved in the `k8s_certs_src` directory.

### certs-install

Installs the private keys and certificates on each node.

### etcd

Installs etcd on each master and initialise cluster.

### kube-apiserver

Installs kube-apiserver on each master as systemd service.

### kube-controller-manager

Installs kube-controller-manager on each master as systemd service. 

### kube-scheduler

Installs kube-scheduler on each master as systemd service.

### cni

Installs cni configuration file needed before starting Kubelet.

### kubelet

Installs kubelet as systemd service on each worker node.

### flannel

Installs flannel as DaemonSet.

### kube-dns

Installs kube-dns as Deployment.

### heapster

Installs heapster in standalone mode as Deployment.

### dashboard

Installs dashboard as Deployment.

License
=======

MIT

Author Information
==================

Yosh de Vos <yosh@yoshz.nl>
