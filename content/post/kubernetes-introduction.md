---
title: "Kubernetes 101"
date: 2020-10-17T15:43:48+08:00
tags: ["Kubernetes", "DevOps"]
categories: ["DevOps"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

### 1.0 What is Kubernetes

Kubernetes is an open-source platform/tool created by Google. It is written in GO-Lang. So currently Kubernetes is an open-source project under Apache 2.0 license. Sometimes in the industry, Kubernetes is also known as “K8s”. With Kubernetes, you can run any Linux container across private, public, and hybrid cloud environments. Kubernetes provides some edge functions, such as Loadbalancer, Service discovery, and Roled Based Access Control(RBAC).

<!--more-->

Basically, kubernetes is a software that allows us to deploy, manage and scale applications. The applications will be packed in containers and kubernetes groups them into units. It allows us to span our application over thousands of servers while looking like one single unit.

- Key Features of Kubernetes
  - Horizontal Scaling
  - Auto Scaling
  - Health check & Self-healing
  - Load Balancer
  - Service Discovery
  - Automated rollbacks & rollouts
  - Canary Deployment

---

## 2.0 Why we need Kubernetes

First, we need to familier with few terms:

**Monolithic Applications:** Years ago, most software applications were big monoliths, running either as a single process or as a small number of processes spread across a handful of servers. These legacy systems are still widespread today. They have slow release cycles and are updated relatively infrequently. At the end of every release cycle, developers pack- age up the whole system and hand it over to the ops team, who then deploys and monitors it. In case of hardware failures, the ops team manually migrates it to the remaining healthy servers. So these components that are all tightly coupled together and have to be developed, deployed, and managed as one entity, because they all run as a single OS process.

- Problems with Monolithic Applications: Change of one part of the application require a redeployment of the whole application. Requires powerful servers, Uses Vertical Scaling ( which is Very Expensive ) and If one components creates problem, the whole application becomes unscalable.

**Microservices Applications:** Today, these big monolithic legacy applications are slowly being broken down into smaller, independently running components called microservices. Each microservice runs as an independent process and communicates with other microservices through simple, well defined interfaces ( APIs ). Microservices communicate through synchronous protocols such as HTTP, over which they usually expose RESTful (REpresentational State Transfer) APIs, or through asyn- chronous protocols such as AMQP (Advanced Message Queueing Protocol).

Microservices are decoupled from each other, they can be developed, deployed, updated,
and scaled individually. This enables you to change components quickly and as often as
necessary to keep up with today’s rapidly changing business requirements.

- Problems with Microservices Based Applications: The bigger numbers of deployable components and increasingly larger datacenters, it becomes increasingly difficult to configure, manage, and keep the whole system running smoothly. It’s much harder to figure out where to put each of those components to achieve high resource utilization and thereby keep the hardware costs down and the one of biggest problem with scaling microservices is multiple apps that are running on same host mey have conflicting dependencies. One solution of this is applications could run in the exact same environment during development and in production so they have the exact same operating system, libraries, system configuration, networking environment, and everything else but this will not solve the conflicting, versions of libraries or different environment requirements to solve this issue we have great technology called containers. ( You are also provide Dedicated VM to particular service but it is not a ideal solution because it increases hardware cost and management)

Doing all this manually is hard work. We need automation, which includes automatic scheduling of those components to our servers, automatic configuration, supervision, and failure-handling. This is where Kubernetes comes in.

![Microservices](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Kubernetes/MicroServices.png)

Kubernetes provides you with a framework to run distributed systems resiliently. It takes care of scaling and failover for your application, provides deployment patterns, and more. For example, Kubernetes can easily manage a canary deployment for your system.

Before used Kubernetes, you need to prepare your infrastructure to deploy a new microservice. I believe it cost you a few days or weeks. Without Kubernetes, large teams would have to manually script the deployment workflows. With Kubernetes, you don’t need to create your deployment script manually and it will reduce the amount of time and resources spent on DevOps.

### 2.1 What kubernetes provides

- Service discovery and load balancing: Kubernetes can expose a container using the DNS name or using their own IP address. If traffic to a container is high, Kubernetes is able to load balance and distribute the network traffic so that the deployment is stable.
- Storage orchestration: Kubernetes allows you to automatically mount a storage system of your choice, such as local storages, public cloud providers, and more.
Automated rollouts and rollbacks You can describe the desired state for your deployed containers using Kubernetes, and it can change the actual state to the desired state at a controlled rate. For example, you can automate Kubernetes to create new containers for your deployment, remove existing containers and adopt all their resources to the new container.
- Automatic bin packing: You provide Kubernetes with a cluster of nodes that it can use to run containerized tasks. You tell Kubernetes how much CPU and memory (RAM) each container needs. Kubernetes can fit containers onto your nodes to make the best use of your resources.
- Self-healing: Kubernetes restarts containers that fail, replaces containers, kills containers that don't respond to your user-defined health check, and doesn't advertise them to clients until they are ready to serve.
- Secret and configuration management: Kubernetes lets you store and manage sensitive information, such as passwords, OAuth tokens, and SSH keys. You can deploy and update secrets and application configuration without rebuilding your container images, and without exposing secrets in your stack configuration.

### 2.2 What kubernetes not provides

Kubernetes is a lot of good things, I hope to have been clear exposing all Kubernetes benefits. The main problem from Kubernetes newbie is that they discover it is not a PaaS (Platform as a Service) system like they suppose. Kubernetes is a lot of things but not an “all included” service. It is great and reduces the amount of work, especially on the sysadmin side, but doesn’t offer you any apart from the infrastructure.

Said that most of the things you are looking for in a fully-managed system are there: simplified deployments, scaling, load balancing, logging, and monitoring. Usually, you get a standard configuration from your hosting but you can theoretically customize it if you really need it.

- It does not limit the types of applications supported. Everything is written into the container so every container application, no matter on technology, can be run. The counterpart is that you still have to define the container by hand.
- It doesn’t offer an automated deployment. You just have to push to a docker repository your built images, no more. This is quite easy if you already work in a process with Continuous Integration, Delivery, and Deployment (CI/CD), but consider that without it will be quite tricky.
- It does not provide any application-level services, just infrastructure. If you need a database, you have to buy a service or run it into a dedicated container. Of course, taking charges of backups and so on.
- Most of the interactions with the system are by a command line that wraps API. That’s very good because it allows automating each setting. Commands syntax is very simple but, if you are looking to a system that is managed fully by a UI you are looking to the worst place.

---

## 3.0 How does Kubernetes work

The system is composed of a master node and any number of worker nodes. When the developer submits a list of apps to the master, Kubernetes deploys them to the cluster of worker nodes. What node a component lands on doesn’t (and shouldn’t) matter—neither to the developer nor to the system administrator.

![Kubernetes Master](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Kubernetes/Kubernetes_Master.png)

Kubernetes will run your containerized app somewhere in the cluster, provide information to its components on how to find each other, and keep all of them running.
Because your application doesn’t care which node it’s running on, Kubernetes can
relocate the app at any time, and by mixing and matching apps, achieve far better
resource utilization than is possible with manual scheduling.

### 3.1 Architecture of Kubernetes Cluster

We’ve seen a bird’s-eye view of Kubernetes’ architecture. Now let’s take a closer look at what a Kubernetes cluster is composed of. At the hardware level, a Kubernetes cluster is composed of many nodes, which can be split into two types:

- The master node, which hosts the Kubernetes Control Plane that controls and manages the whole Kubernetes system
- Worker nodes that run the actual applications you deploy

![Kubernetes Architecture](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Kubernetes/Kubernetes_arch1.png)

#### 3.1.1 Master ( Control Plane )

The Control Plane is what controls the cluster and makes it function. It consists of
multiple components that can run on a single master node or be split across multiple
nodes and replicated to ensure high availability. These components are

- **Kubernetes API Server**: The application that serves Kubernetes functionality through a RESTful interface and stores the state of the cluster ( which admin and other control plane communicated with ).

- **Scheduler**: Scheduler watches API server for new Pod requests. It communicates with Nodes to create new pods and to assign work to nodes while allocating resources or imposing constraints.

- **Controller Manager**: Component on the master that runs controllers. Includes Node controller, Endpoint Controller, Namespace Controller, etc. ( Performs cluster-level functions, such as replicating components, keeping track of worker nodes, handling node failures, and so on )

  - Node Controller: Responsible for noticing and responding when nodes go down.
  - Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
  - Endpoints Controller: Populates the Endpoints object (that is, it joins Services and Pods).
  - Service Account and Token Controllers: Create default accounts and API access tokens for new namespaces.
  - Cloud-Controller-Manager: Cloud-controller-manager runs controllers that interact with the underlying cloud providers. The cloud-controller-manager binary is an alpha feature introduced in Kubernetes release 1.6. Cloud-controller-manager runs cloud-provider-specific controller loops only. You must disable these controller loops in the Kube-controller-manager. You can disable the controller loops by setting the --cloud-provider flag to external when starting the Kube-controller-manager.
  - Node Controller: For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding.
  - Route Controller: For setting up routes in the underlying cloud infrastructure.
  - Service Controller: For creating, updating, and deleting cloud provider load balancers.
  - Volume Controller: For creating, attaching, and mounting volumes, and interacting with the cloud provider to orchestrate volumes.

- **etcd**: A reliable distributed data store that persistently stores the cluster
configuration.

The components of the Control Plane hold and control the state of the cluster, but
they don’t run your applications. This is done by the (worker) nodes.

#### 3.1.2 Slave ( Worker Nodes )

The worker nodes are the machines that run your containerized applications. The
task of running, monitoring, and providing services to your applications is done by
the following components:

- Docker, rkt, or another container runtime, which runs your containers

- **Kubelet**: Kubectl registering the nodes with the cluster, watches for work assignments from the scheduler, instantiate new Pods, report back to the master.
Container Engine: Responsible for managing containers, image pulling, stopping the container, starting the container, destroying the container, etc.

- **Kube Proxy**: Responsible for forwarding app user requests to the right pod (load-balances network traffic between application components).

- The sequence of deployment: `DevOps -> API Server -> Scheduler -> Cluster ->Nodes -> Kubelet -> Container Engine -> Spawn Container in Pod`

- The sequence of App user request: `App user -> Kube proxy -> Pod -> Container(Your app is run here)`

### 3.2 The Six Layers of K8s

![Kubernetes Abstrction](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Kubernetes/Kubernetes_Abstraction.png)

Deployments create and manage ReplicaSets, which create and manage Pods, which run on Nodes, which have a container runtime, which run the app code you put in your Docker image.

The levels shaded blue are higher-level K8s abstractions. The green levels represent Nodes and Node subprocess that you should be aware of, but may not touch.

#### 3.2.1 Deployments

Although pods are the basic unit of computation in Kubernetes, they are not typically directly launched on a cluster. Instead, pods are usually managed by one more layer of abstraction: the deployment. A deployment’s primary purpose is to declare how many replicas of a pod should be running at a time. When a deployment is added to the cluster, it will automatically spin up the requested number of pods, and then monitor them. If a pod dies, the deployment will automatically re-create it. Using a deployment, you don’t have to deal with pods manually. You can just declare the desired state of the system, and it will be managed for you automatically.

#### 3.2.2 ReplicaSet

The Deployment creates a ReplicaSet that will ensure your app has the desired number of Pods. ReplicaSets will create and scale Pods based on the triggers you specify in your Deployment. Replication Controllers perform the same function as ReplicaSets, but Replication Controllers are old school. ReplicaSets are the smart way to manage replicated Pods in 2019.

#### 3.2.3 Pods

All containers will run in a pod. Pods abstract the network and storage away from the underlying containers. Your app will run here. Each pod has one unique IP address assigned which means one pod can communicate with each other like a traditional container in a docker environment. Each container inside the pod can reach all other pods into the virtual network, but cannot deep to the other container on other pods. That’s important to guarantee the pod abstraction: nobody has to know how the Pods are composed internally. Moreover, the IP assigned is volatile so you must use always the service name (resolved to the right IP directly). Pods are used as the unit of replication in Kubernetes. If your application becomes too popular and a single pod instance can’t carry the load, Kubernetes can be configured to deploy new replicas of your pod to the cluster as necessary. Even when not under heavy load, it is standard to have multiple copies of a pod running at any time in a production system to allow load balancing and failure resistance. Pods handle Volumes, Secrets, and configuration for containers. Pods are ephemeral. They are intended to be restarted automatically when they die.

Note: Worker Node is already explained in above section

### 3.3 Key Terms used with Kubernetes

#### 3.3.1 Services

The name “service” in informatics science is overused. In Kubernetes scope think to a service like something you want to serve. A Kubernetes service involves a set of pods and may offer a complex feature or just expose a single Pod with a single container. So you can have a service that provides a CMS feature, with database and web server inside, or two different services, one for the database and one for the webserver. That’s up to you. Basically it is abstraction layer on top of a set of ephemeral pods (think of this as the ‘face’ of a set of pods)

#### 3.3.2 Ingress

By default, Kubernetes provides isolation between pods and the outside world. If you want to communicate with a service running in a pod, you have to open up a channel for communication. This is referred to as ingress. To do this there is an ingress controller that does something similar to a load balancer. It virtually forwards traffic from outside to the services. During this step, based on the ingress implementation you have chosen, you can add HTTPS encryption, route traffic based on the hostname or URL segments. The only service can be linked to the ingress controller, not Pods. There are multiple ways to add ingress to your cluster. The most common ways are by adding either an Ingress controller, or a LoadBalancer.

#### 3.3.3 Volume

By default Pod storage is volatile. This is important to know because at the first restart you will lose everything. Kubernetes volumes allow mapping some part of the hard drive of containers to a safe place. That space can be shared between containers. The mount point can be any part of the container, but a volume cannot be mount into another one.

**PersistentVolumes and PersistentVolumeClaims**: To help abstract away infrastructure specifics, K8s developed PersistentVolumes and PersistentVolumeClaims. Unfortunately the names are a bit misleading, because vanilla Volumes can have persistent storage, as well. PersisententVolumes (PV) and PersisentVolumeClaims (PVC) add complexity compared to using Volumes alone. However, PVs are useful for managing storage resources for large projects. With PVs, a K8s user still ends up using a Volume, but two steps are required first.

1. A PersistentVolume is provisioned by a Cluster Administrator (or it’s provisioned dynamically).
2. An individual Cluster user who needs storage for a Pod creates PersistentVolumeClaim manifest. It specifies how much and what type of storage they need. K8s then finds and reserves the storage needed.

The user then creates a Pod with a Volume that uses the PVC. PersistentVolumes have lifecycles independent of any Pod. In fact, the Pod doesn’t even know about the PV, just the PVC. PVCs consume PV resources, analogously to how Pods consume Node resources.

#### 3.3.4 Namespaces

Think to namespace like the feature that makes Kubernetes multitenant. The namespace is the tenant level. Each namespace can partitioning resources to isolate services, ingress, and many other things. This feature is good to have a strong separation between application, delegate safely to different teams, and have separated environments in a single infrastructure. It is a virtual cluster on top of an underlying physical cluster

#### 3.3.5 Labels and Selectors

Labels are key-value pairs used to tag objects. Objects are the items that you create in a Kubernetes cluster (pods, deployments, replica sets, services, volumes etc). Selectors are used to collect objects based on tags.

#### 3.3.6 StatefulSets

As we know, a ReplicaSet creates and manages Pods. If a Pod shuts down because a Node fails, a ReplicaSet can automatically replace the Pod on another Node. You should generally create a ReplicaSet through a Deployment rather than creating it directly, because it’s easier to update your app with a Deployment.

![States](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Kubernetes/Kubernetes_States.png)

Sometimes your app will need to keep information about its state. You can think of state as the current status of your user’s interaction with your app. So in a video game it’s all the unique aspects of the user’s character at a point in time. What do you do when your app has state you need to keep track of? Use a StatefulSet.

Like a ReplicaSet, a StatefulSet manages deployment and scaling of a group of Pods based on a container spec. Unlike a Deployment, a StatefulSet’s Pods are not interchangeable. Each Pod has a unique, persistent identifier that the controller maintains over any rescheduling. StatefulSets for good for persistent, stateful backends like databases. The state information for the Pod is held in a Volume associated with the StatefulSet.

#### 3.3.7 DaemonSets

DaemonSets are for continuous process. They run one Pod per Node. Each new Node added to the cluster automatically gets a Pod started by the DaemonSet. DaemonSets are useful for ongoing background tasks such as monitoring and log collection.
StatefulSets and DaemonSets are not controlled by a Deployment. Although they are at the same level of abstraction as a ReplicaSet, there is not a higher level of abstraction for them in the current API.

#### 3.3.8 ETCD

It stores the configuration information which can be used by each of the nodes in the cluster. It is a high availability key-value store that can be distributed among multiple nodes. It is accessible only by Kubernetes API server as it may have some sensitive information. It is a distributed key-value store which is accessible to all.
ETCD is a distributed reliable key-value store used by Kubernetes to store all data used to manage the cluster. Think of it this way, when you have multiple nodes and multiple masters in your cluster, etcd stores all that information on all the nodes in the cluster in a distributed manner. ETCD is responsible for implementing locks within the cluster to ensure there are no conflicts between the Masters.

#### 3.3.9 Cluster IP

Only has a Virtual IP (also called Cluster IP). This service can be used within a cluster only Acts like a traffic router to your Pods inside the cluster. Each port exposed by a pod will need a service if you want a client to talk to it via that port. By default, the service port is the same as the port exposed by the Pod. Different methods to access this service: From any node inside the cluster: `<cluster IP>:<service port>` From the node where a replica of the Pod is running: `<pod IP>:<pod port>`.

#### 3.3.10 Node Port

For this service, a physical port on the node is mapped to the service port and the service connects to the Pod via the pod port. Remember, this will also have a Virtual IP (i.e. cluster IP). Different methods to access this service: From outside the cluster, if any node in the cluster has a public IP: `<node public IP>:<node port>` From any node inside the cluster: `<cluster IP>:<service port>` From the node where a replica of the Pod is running: `<pod IP>:<pod port>`.

#### 3.3.11 Load Balancer

Giving public access to a node in your cluster is not a recommended method, When you want to give public access to a service, create a load balancer service (not getting into ingress at this stage). For each service that you want to give public access, you need to create a load balancer service, This type of service will generally work only in a cloud environment. Remember, this will also have a Virtual IP (i.e. cluster IP) This service will have a node port too Different methods to access this service: From outside the cluster: `<load balancer dns>:<service port>` From outside the cluster, if any node in the cluster has a public IP: `<node public IP>:<node port>` From any node inside the cluster: `<cluster IP>:<service port>` From the node where a replica of the Pod is running: `<pod IP>:<pod port>`.

#### 3.3.12 External Name

This maps a service to endpoints completely outside of the cluster.

---

## 4.0 Befits of using kubernetes

### 4.1 Kubernetes will keeping your containers running

Once the application is running, Kubernetes continuously makes sure that the deployed
state of the application always matches the description you provided. For example, if
you specify that you always want five instances of a web server running, Kubernetes will always keep exactly five instances running. If one of those instances stops working properly, like when its process crashes or when it stops responding, Kubernetes will restart it automatically. Similarly, if a whole worker node dies or becomes inaccessible, Kubernetes will select new nodes for all the containers that were running on the node and run them on the newly selected nodes.

### 4.2 Scaling the number of copies

While the application is running, you can decide you want to increase or decrease the number of copies, and Kubernetes will spin up additional ones or stop the excess ones, respectively. You can even leave the job of deciding the optimal number of cop- ies to Kubernetes. It can automatically keep adjusting the number, based on real-time metrics, such as CPU load, memory consumption, queries per second, or any other metric your app exposes.

### 4.3 Hitting a moving target

We’ve said that Kubernetes may need to move your containers around the cluster This can occur when the node they were running on has failed or because they were evicted from a node to make room for other containers. If the container is providing a service to external clients or other containers running in the cluster, how can they use the container properly if it’s constantly moving around the cluster? And how can clients connect to containers providing a service when those containers are replicated and spread across the whole cluster? To allow clients to easily find containers that provide a specific service, you can tell Kubernetes which containers provide the same service and Kubernetes will expose all of them at a single static IP address and expose that address to all applications run- ning in the cluster. This is done through environment variables, but clients can also look up the service IP through good old DNS. The kube-proxy will make sure connec- tions to the service are load balanced across all the containers that provide the service. The IP address of the service stays constant, so clients can always connect to its con- tainers, even when they’re moved around the cluster.

### 4.4 Simplifying application deployment

Kubernetes exposes all its worker nodes as a single deployment platform,
application developers can start deploying applications on their own and don’t need
to know anything about the servers that make up the cluster. In essence, all the nodes are now a single bunch of computational resources that are waiting for applications to consume them. A developer doesn’t usually care what kind of server the application is running on, as long as the server can provide the application with adequate system resources. Certain cases do exist where the developer does care what kind of hardware the application should run on. ( Like some app needs SSD deployment instead of HDD )

### 4.5 Achieving better utilization of hardware

By setting up Kubernetes on your servers and using it to run your apps instead of running them manually, you’ve decoupled your app from the infrastructure. When you tell Kubernetes to run your application, you’re letting it choose the most appropriate node to run your application on based on the description of the application’s resource requirements and the available resources on each node. By using containers and not tying the app down to a specific node in your cluster, you’re allowing the app to freely move around the cluster at any time, so the different app components running on the cluster can be mixed and matched to be packed tightly onto the cluster nodes. This ensures the node’s hardware resources are utilized as best as possible.

The ability to move applications around the cluster at any time allows Kubernetes
to utilize the infrastructure much better than what you can achieve manually.

### 4.6 Health checking and self-healing

Having a system that allows moving an application across the cluster at any time is also valuable in the event of server failures. As your cluster size increases, you’ll deal with failing computer components ever more frequently. Kubernetes monitors your app components and the nodes they run on and auto- matically reschedules them to other nodes in the event of a node failure. This frees the ops team from having to migrate app components manually and allows the team to immediately focus on fixing the node itself and returning it to the pool of available hardware resources instead of focusing on relocating the app.

### 4.7 Automatic scaling

Using Kubernetes to manage your deployed applications also means the ops team
doesn’t need to constantly monitor the load of individual applications to react to sudden load spikes. As previously mentioned, Kubernetes can be told to monitor the resources used by each application and to keep adjusting the number of running instances of each application. If Kubernetes is running on cloud infrastructure, where adding additional nodes is as easy as requesting them through the cloud provider’s API, Kubernetes can even automatically scale the whole cluster size up or down based on the needs of the deployed applications.

### 4.8 Simplifying application development

Then there’s the fact that developers don’t need to implement features that they would usually implement. This includes discovery of services and/or peers in a clustered application. Kubernetes does this instead of the app. Usually, the app only needs to look up certain environment variables or perform a DNS lookup. If that’s not enough, the application can query the Kubernetes API server directly to get that and/or other information. Querying the Kubernetes API server like that can even save developers from having to implement complicated mechanisms such as leader election.

### 4.9 Storage orchestration

Consequently, mount local or public cloud or network storage.

### 4.10 Secret and configuration management

Create and update secrets and configs without rebuilding your image

### 4.11 Horizontal Scaling

first, we need to know what are the horizontal and vertical scalings are:

- Vertically Scaling : Adding more CPUs, memory, and other server components
- Horizontal Scaling : Setting up additional Servers and running multiple copies

Kubernetes uses horizontal scaling, which provides greater flexibilities to the companies

---

## 5.0 Ways to spin up kubernetes cluster

### 5.1 Kubeadm

The official CNCF tool for provisioning Kubernetes clusters in a variety of shapes and forms (e.g. single-node, multi-node, HA, self-hosted)

### 5.2 Minikube

Minikube can run on Windows and MacOS, because it relies on virtualization (e.g. Virtualbox) to deploy a kubernetes cluster in a Linux VM. You can also run minikube directly on linux with or without virtualization. It also has some developer-friendly features, like add-ons.

From a user perspective minikube is a very beginner friendly tool. You start the cluster using `minikube start`, wait a few minutes and your `kubectl` is ready to go. To specify a Kubernetes version you can use the `--kubernetes-version` flag.

If you are new to Kubernetes the first class support for its dashboard that minikube offers may help you. With a simple minikube dashboard the application will open up giving you a nice overview of everything that is going on in your cluster. This is being achieved by minikube’s addon system that helps you integrating things like, Helm, Nvidia GPUs and an image registry with your cluster.

### 5.3 MicroK8s

MicroK8s is a Kubernetes distribution from Canonical that is designed for fast and simple deployment, which makes it a good choice to run Kubernetes locally.

Microk8s is similar to minikube in that it spins up a single-node Kubernetes cluster with its own set of add-ons. If MicroK8s runs on Linux, it also offers the advantage of not requiring VMs. On Windows and macOS, MicroK8s uses a VM framework called Multipass to create VMs for the Kubernetes cluster.

Like minikube, microk8s is limited to a single-node Kubernetes cluster, with the added limitation of only running on Linux and only on Linux where snap is installed.

### 5.4 K3s

K3s is a minified version of Kubernetes developed by Rancher Labs. By removing dispensable features (legacy, alpha, non-default, in-tree plugins) and using lightweight components (e.g. sqlite3 instead of etcd3) they achieved a significant downsizing. This results in a single binary with a size of around 60 MB.

K3s runs on any Linux distribution without any additional external dependencies or tools. It is marketed by Rancher as a lightweight Kubernetes offering suitable for edge environments, IoT devices, CI pipelines, and even ARM devices, like Raspberry Pi's. K3s achieves its lightweight goal by stripping a bunch of features out of the Kubernetes binaries (e.g. legacy, alpha, and cloud-provider-specific features), replacing docker with containerd, and using sqlite3 as the default DB (instead of etcd). As a result, this lightweight Kubernetes only consumes 512 MB of RAM and 200 MB of disk space. K3s has some nice features, like Helm Chart support out-of-the-box.

Unlike the previous two offerings, K3s can do multiple node Kubernetes cluster. However, due to technical limitations of SQLite, K3s currently does not support High Availability (HA), as in running multiple master nodes.

One feature that stands out is called auto deployment. It allows you to deploy your Kubernetes manifests and Helm charts by putting them in a specific directory. K3s watches for changes and takes care of applying them without any further interaction. This is especially useful for CI pipelines and IoT devices (both target use cases of K3s). Just create/update your configuration and K3s makes sure to keep your deployments up to date.

### 5.5 Kind

Kind (Kubernetes-in-Docker), as the name implies, runs Kubernetes clusters in Docker containers. This is the official tool used by Kubernetes maintainers for Kubernetes v1.11+ conformance testing. It supports multi-node clusters as well as HA clusters. Because it runs K8s in Docker, kind can run on Windows, Mac, and Linux.

Kind is optimized first and foremost for CI pipelines, so it may not have some of the developer-friendly features of other offerings. Kind leads to a significantly faster startup speed compared to spawning VM.

Creating a cluster is very similar to minikube’s approach. Executing kind create cluster, playing the waiting game and afterwards you are good to go. By using different names (--name) kind allows you to create multiple instances in parallel.

If you are looking for a way to programmatically create a Kubernetes cluster, kind kindly (you have been for waiting for this, don’t you ) publishes its Go packages that are used under the hood.

### 5.6 K3d

A new project that aims to bring K3s-in-Docker (similar to kind).

---
