---
title: "How To Setup Vagrant and Virtual Box for Ubuntu 20.04!!"
date: 2020-09-22T15:43:48+08:00
draft: false
tags: ["DevOps", "Linux"]
categories: ["Linux"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

## Overview

---

### What is Vagrant??

Vagrant is an open-source software product for building and maintaining portable virtual software development environments, e.g., for Virtual Box, KVM, Hyper-V, Docker containers, VMware, and AWS.

<!--more-->

### What is VirtualBox??

Oracle VM Virtual Box is a free and open-source hosted hypervisor for x86 virtualization, developed by Oracle Corporation. Created by Innotek, it was acquired by Sun Microsystems in 2008, which was, in turn, acquired by Oracle in 2010. Virtual Box can be installed on Windows, Mac-OS, Linux, Solaris, and Open Solaris.

### Virtual Box + Vagrant = 🔥

Vagrant and Virtual Box can stimulate the production environment of your app or website. For example, suppose you’re using Digital Ocean or AWS to run a Virtual Private Server (VPS) in the cloud running Ubuntu, PHP, and MySQL. In that case, you can install your local version to have all the same versions of that software while keeping your own computer’s software untouched. These both can reduce and eliminate bugs and errors that result from trying to develop code for a production server on an environment that does not match. So with these, you can create VMs in a virtual box in just one command with so much customization.

---

## Install Virtual Box

Download Virtual Box 5.2.x ( Latest ) and its Extension Pack from Virtual Box 5.2.x Builds. I am using virtual box 5.2.x because the vagrant gives a problem with 6.x sometimes. That’s why I m using 5.2.x (It’s working like a charm ).

{{< gist AkashRajvanshi 8ff69a4d610b9d43305f595cf5db73db "VirtualBox.sh" >}}

---

## Install Vagrant

{{< gist AkashRajvanshi f127cb2797be36eb2c73a199ef9c8a7b "Vagrant.sh" >}}

---

## Let’s Deploy Some Fun 🔥

Here, we start 2 Ubuntu 18.04 VM’s with vagrant, and we set up the Kubernetes cluster on nodes ( 1 work as master node & other work as worker node ) in just less than a minute. So, let’s start the journey 🎇:

Directory Structure

```
├── files
│   └── daemon.json
├── vars
│   └── default.yml
├── ansible.cfg
├── hosts
├── docker.yml
├── kubernetes.yml
├── portainer.yml
└── Vagrantfile
```

First of all, you need to create VM’s with vagrant. To create VM’s with vagrant, you need to create a file name Vagrantfile.
This given Vagrantfile will set up two Ubuntu 18.04 VM’s in VirtualBox.

{{< gist AkashRajvanshi 48d82a591f9cb7334b0e6e5237376ecd "Vagrant-file.sh" >}}

Deploy the VM’s

```bash
$ vagrant up

$ vagrant status
Current machine states:
master                    running (virtualbox)
worker                    running (virtualbox)
```

We set up the Kubernetes cluster on both nodes ( 1 Master, 1 Worker Node). To do this whole process, we are using ansible.

To setup ansible ( You need to install ansible on your working machine )
Now, you need to create some ansible config files so that Ansible can connect to the nodes.

- hosts

```bash
[cluster]
master ansible_host=127.0.0.1 ansible_port=2222
worker ansible_host=127.0.0.1 ansible_port=2200
```

- ansible.cfg

```bash
[defaults]
inventory = hosts
remote_user = vagrant
private_key_file = /home/${USER}/.vagrant.d/insecure_private_key
host_key_checking = False
```

To check connection to the nodes with ansible:

```bash
$ ansible cluster -i hosts -m ping
worker | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
master | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

---

### To Install Docker

You can install docker on both the nodes in just a 20s.

{{< gist AkashRajvanshi 16034a99747abf5d0908b4b6476e6470 "docker.yaml" >}}

```bash
# Daemon.json

{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "insecure-registries" : ["localhost:32000"]
}
```

```bash
# Run ansible playbook
$ time ansible-playbook -i hosts docker.yml

PLAY [Install Docker with Ansible] *******************************************************************

TASK [Gathering Facts] *******************************************************************************
ok: [worker]
ok: [master]TASK [Updating the system] ***************************************************************************
changed: [master]
changed: [worker]TASK [Upgrading the system] **************************************************************************
changed: [master]
changed: [worker]TASK [Installing aptitude using apt] *****************************************************************
changed: [master]
changed: [worker]TASK [Installing required system packages] ***********************************************************
changed: [master] => (item=apt-transport-https)
changed: [worker] => (item=apt-transport-https)
ok: [master] => (item=ca-certificates)
ok: [worker] => (item=ca-certificates)
ok: [master] => (item=curl)
ok: [worker] => (item=curl)
ok: [master] => (item=software-properties-common)
ok: [worker] => (item=software-properties-common)
changed: [worker] => (item=python3-pip)
changed: [master] => (item=python3-pip)
changed: [worker] => (item=virtualenv)
changed: [master] => (item=virtualenv)
ok: [worker] => (item=python3-setuptools)
ok: [master] => (item=python3-setuptools)
ok: [worker] => (item=python3-apt)
ok: [master] => (item=python3-apt)TASK [Adding Docker GPG apt Key] *********************************************************************
changed: [worker]
changed: [master]TASK [Adding Docker Repository] **********************************************************************
changed: [worker]
changed: [master]TASK [Updating apt and installing docker-ce] *********************************************************
changed: [master]
changed: [worker]TASK [Copying Daemon File] ***************************************************************************
changed: [worker]
changed: [master]TASK [Restarting Docker Service] *********************************************************************
changed: [master]
changed: [worker]TASK [Adding vagrant user to docker] *****************************************************************
changed: [master]
changed: [worker]PLAY RECAP *******************************************************************************************
master                     : ok=11   changed=10   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
worker                     : ok=11   changed=10   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

ansible-playbook -i hosts docker.yml  20.73s user 2.53s system 11% cpu 3:18.11 total
```

---

### To Install Portainer

You can install portainer on both the nodes in just 6s. ( To access the portainer http://NodeIP:9000 )

{{< gist AkashRajvanshi 7ff68b38af98a95e83fddd022ffc64c6 "portainer.yaml" >}}

```yaml
# vars/default.yml

---
container_name: portainer
container_image: portainer/portainer-ce
container_ports: 9000:9000
container_config_volume: /var/run/docker.sock:/var/run/docker.sock
container_data_volume: /home/vagrant/portainer:/data
```

```bash
# Run playbook
$ time ansible-playbook -i hosts portainer.yml

PLAY [Installing Portainer With Ansible] *************************************************************

TASK [Gathering Facts] *******************************************************************************
ok: [master]
ok: [worker]TASK [Installing Docker Module for Python] ***********************************************************
changed: [worker]
changed: [master]TASK [Pulling portainer docker image] ****************************************************************
changed: [worker]
changed: [master]TASK [Creating Portainer Container] ******************************************************************
changed: [worker]
changed: [master]PLAY RECAP *******************************************************************************************
master                     : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
worker                     : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

ansible-playbook -i hosts portainer.yml  6.10s user 0.81s system 16% cpu 42.014 total
```

---

### To Install Kubernetes

You can setup Kubernetes using microk8s in just 11s.

{{< gist AkashRajvanshi 7ea07ebaf4b6c56f568938c72184c816 "kubernetes.yaml" >}}

```bash
# Run playbook
$ time ansible-playbook -i hosts kubernetes.yml

PLAY [Setup Kubernetes with Ansible] *****************************************************************

TASK [Gathering Facts] *******************************************************************************
ok: [worker]
ok: [master]TASK [Updating the system] ***************************************************************************
ok: [master]
ok: [worker]TASK [Installing Snap] *******************************************************************************
ok: [master]
ok: [worker]TASK [Installing MicroK8s Snap Package] *************************************************************
[WARNING]: The value 1.19 (type float) in a string field was converted to '1.19' (type string). If
this does not look like what you expect, quote the entire value to ensure it does not change.
changed: [worker]
changed: [master]TASK [Adding vagrant user to microk8s group] *********************************************************
changed: [master]
changed: [worker]TASK [Giving vagrant user a permission to ~/.kube directory] ******************************
changed: [master]
changed: [worker]TASK [debug] *****************************************************************************************
ok: [master] => {
    "shell_result.stdout_lines": [
        "NAME            STATUS   ROLES    AGE     VERSION",
        "ubuntu-bionic   Ready    <none>   6m40s   v1.19.0-34+09a4aa08bb9e93"
    ]
}
ok: [worker] => {
    "shell_result.stdout_lines": [
        "NAME            STATUS   ROLES    AGE     VERSION",
        "ubuntu-bionic   Ready    <none>   6m48s   v1.19.0-34+09a4aa08bb9e93"
    ]
}PLAY RECAP *******************************************************************************************
master                     : ok=7    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
worker                     : ok=7    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

ansible-playbook -i hosts kubernetes.yml  11.74s user 1.41s system 13% cpu 1:39.82 total
```

---

### Last Step:

```bash
# Adding a node to cluster
# Note: Make sure to change to hostname: ( hostnamectl set-hostname ubuntu-worker/master)# On Master Node
$ microk8s add-node
microk8s join 10.0.1.13:25000/62c7fa47f537f4425f5d4071f0688a71

# On Worker Node
$ microk8s join 10.0.1.13:25000/62c7fa47f537f4425f5d4071f0688a71

# Master Node
$ microk8s kubectl get nodes
NAME            STATUS     ROLES    AGE   VERSION
ubuntu-master   Ready      <none>   3m53s v1.19.0-34+09a4aa08bb9e93
ubuntu-worker   Ready      <none>   3m59s v1.19.0-34+09a4aa08bb9e93
```

Congratulations 🎊, You’ve just set up two ubuntu nodes with docker, portainer & Kubernetes cluster in less than a minute!!

---

## Conclusion

Now, you’ve operational experience using vagrant and the virtual box together and set up both of them easily in any operating system. Also, you can effortlessly create VMs using vagrant to set up things like Docker, Kubernetes, and other more.

---
