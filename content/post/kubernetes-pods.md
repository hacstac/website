---
title: "Kubernetes Pods Deep Dive"
date: 2020-10-20T15:43:48+08:00
tags: ["Kubernetes", "DevOps"]
categories: ["DevOps"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

### 1.0 Pods

POD: Kubernetes groups multiple containers into a single atomic unit called a Pod

<!--more-->

A pod is a co-located group of containers and represents the basic building block in Kubernetes. Instead of deploying containers individually, we always deploy and operate on a pod of containers. We’re not implying that a pod always includes more than one container—it’s common for pods to contain only a single container. The key thing about pods is that when a pod does have multiple containers, all of them are always run on a single worker node—it never spans multiple worker nodes. A Pod represents a collection of application containers and volumes running in the same execution environment. Pods, not containers, are the smallest deployable arti‐ fact in a Kubernetes cluster. This means all of the containers in a Pod always land on the same machine.

Applications running in the same Pod share the same IP address and port space (net‐ work namespace), have the same hostname (UTS namespace), and can communicate using native interprocess communication channels over System V IPC or POSIX message queues (IPC namespace). However, applications in different Pods are isolated from each other; they have different IP addresses, different hostnames, and more. Containers in different Pods running on the same node might as well be on different servers.

![Pods](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Kubernetes/Pod1.png)

---

### 1.1 Some general FAQs regarding pods

#### 1.1.1 Why we need pods

- **Understanding why multiple containers are better than one container running multiple processes**

Imagine an app consisting of multiple processes that either communicate through
IPC (Inter-Process Communication) or through locally stored files requires them to run on the same machine. Because in Kubernetes, we always run processes in containers, and each container is much like an isolated machine. Containers are designed to run only a single process per container (unless the process itself spawns child processes). If you run multiple unrelated operations in a single container, it is your responsibility to keep all those processes running, manage their logs, and so on. For example, you'd have to include a mechanism for automatically restarting individual processes if they crash. Also, all those processes would log to the same standard output, so you'd have difficulty figuring out what method logged. That's why we need to run each process in its container, and because of this problem, we need a higher-level construct that will allow us to bind containers together and manage them as a single unit. This is the reasoning behind pods.

A pod of containers allows us to run near related processes together and provide them with (almost) the same environment as if they were all running in a single
container, while keeping them somewhat isolated. This way, we can get the best of both
worlds. We can take advantage of all the features containers provide, while at the
same time giving the processes the illusion of running together.

- **Understanding the partial isolation between containers of the same pod**

We want containers inside each group to share certain resources, although not all, so that they’re not fully isolated. Kubernetes achieves this by config- uring Docker to have all containers of a pod share the same set of Linux namespaces instead of each container having its own set. Because all containers of a pod run under the same Network and UTS namespaces (we’re talking about Linux namespaces here), they all share the same hostname and network interfaces. Similarly, all containers of a pod run under the same IPC namespace and can communicate through IPC. In the latest Kubernetes and Docker versions, they can also share the same PID namespace, but that feature isn’t enabled by default. But when it comes to the filesystem, things are a little different. Because most of the container’s filesystem comes from the container image, by default, the filesystem of each container is fully isolated from other containers.

#### 1.1.2 How containers share the same IP and Port space in pods

One thing to stress here is that because containers in a pod run in the same Network namespace, they share the same IP address and port space. This means processes running in containers of the same pod need to take care not to bind to the same port numbers or they’ll run into port conflicts. But this only concerns containers in the same pod. Containers of different pods can never run into port conflicts, because each pod has a separate port space. All the containers in a pod also have the same loopback network interface, so a container can communicate with other containers in the same pod through localhost. All pods in a Kubernetes cluster reside in a single flat, shared, network-address space, which means every pod can access every other pod at the other pod’s IP address. No NAT (Network Address Translation) gateways exist between them. When two pods send network packets between each other, they’ll each see the actual IP address of the other as the source IP in the packet.

Consequently, communication between pods is always simple. It doesn’t matter if two pods are scheduled onto a single or onto different worker nodes; in both cases the containers inside those pods can communicate with each other across the flat NAT- less network, much like computers on a local area network (LAN), regardless of the actual inter-node network topology. Like a computer on a LAN, each pod gets its own IP address and is accessible from all other pods through this network established specifically for pods. This is usually achieved through an additional software-defined net- work layered on top of the actual network.

![Pod Network](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Kubernetes/Pod_Network.png)

#### 1.1.3 How to organizing containers across pods properly

Pods are relatively lightweight, we can have as many as we need without incurring almost any overhead. Instead of stuffing everything into a single pod, we should organize apps into multiple pods, where each one contains only tightly related components or processes. For Ex: a multi-tier application consisting of a frontend
application server and a backend database should be configured as a single pod or as
two pods? Although nothing is stopping us from running both the frontend server and the database in a single pod with two containers, it isn’t the most appropriate way. We’ve said that all containers of the same pod always run co-located, but do the web server and the database really need to run on the same machine? The answer is obviously no.

If both the frontend and backend are in the same pod, then both will always be run on the same machine. If you have a two-node Kubernetes cluster and only this single pod, you’ll only be using a single worker node and not taking advantage of the computational resources (CPU and memory) you have at your disposal on the second node. Splitting the pod into two would allow Kubernetes to schedule the frontend to one node and the backend to the other node, thereby improving the utilization of your infrastructure.

Another reason why you shouldn’t put them both into a single pod is scaling. A pod is also the basic unit of scaling. Kubernetes can’t horizontally scale individual containers; instead, it scales whole pods. If your pod consists of a frontend and a backend container, when you scale up the number of instances of the pod to, let’s say, two, you end up with two frontend containers and two backend containers. Usually, frontend components have completely different scaling requirements
than the backends, so we tend to scale them individually. Not to mention the fact that
backends such as databases are usually much harder to scale compared to (stateless)
frontend web servers.

#### 1.1.4 When to use multiple containers in a pod

The main reason to put multiple containers into a single pod is when the application
consists of one main process and one or more complementary processes. For example, the main container in a pod could be a web server that serves files from a certain file directory, while an additional container (a sidecar container) periodically downloads content from an external source and stores it in the web server’s directory. But always remember you need a good reason to place a two containers in a single pod. ( like another container for gathering logs or to be used for more specific need )

#### 1.1.5 What is pod manifest

Pods are described in a Pod manifest. The Pod manifest is just a text-file representation of the Kubernetes API object. Kubernetes strongly believes in declarative configu‐ ration. Declarative configuration means that you write down the desired state of the world in a configuration and then submit that configuration to a service that takes actions to ensure the desired state becomes the actual state.

The Kubernetes API server accepts and processes Pod manifests before storing them in persistent storage (etcd). The scheduler also uses the Kubernetes API to find Pods that haven’t been scheduled to a node. The scheduler then places the Pods onto nodes depending on the resources and other constraints expressed in the Pod manifests. Multiple Pods can be placed on the same machine as long as there are sufficient resources. However, scheduling multiple replicas of the same application onto the same machine is worse for reliability, since the machine is a single failure domain. Consequently, the Kubernetes scheduler tries to ensure that Pods from the same application are distributed onto different machines for reliability in the presence of such failures. Once scheduled to a node, Pods don’t move and must be explicitly destroyed and rescheduled.

---

### 1.2 Let get into the pods

Pods and other Kubernetes resources are usually created by posting a JSON or YAML
manifest to the Kubernetes REST API endpoint. Also, we can use other, simpler ways
of creating resources, such as the kubectl run command, but this usually allow us to configure a limited set of properties. Additionally, defining all your Kubernetes objects from YAML files makes it possible to store them in a version control system, with all the benefits it brings.

#### 1.2.1 Main parts of pod definition

- The three important sections of any pod defination is:
  - Metadata includes the name, namespace, labels, and other information about the pod.
  - Spec contains the actual description of the pod’s contents, such as the pod’s containers, volumes, and other data.
  - Status contains the current information about the running pod, such as what condition the pod is in, the description and status of each container, and the pod’s internal IP and other basic info. The status part contains read-only runtime data that shows the state of the resource at a given moment. When creating a new pod, you never need to provide the status part.

EX: let’s look at what a YAML definition for one of those pods looks like: I know this looks complicated, but it becomes simple once you understand the basics and know how to distinguish between the important parts and the minor details. Also, you can take comfort in the fact that when creating a new pod, the YAML you need to write is much shorter

```bash
---linkding.yml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cattle.io/timestamp: "2020-09-19T07:58:22Z"
    cni.projectcalico.org/podIP: 10.42.0.66/32
    cni.projectcalico.org/podIPs: 10.42.0.66/32
    field.cattle.io/ports: '[[{"containerPort":9090,"dnsName":"linkding-hostport","hostPort":9090,"kind":"HostPort","name":"port","protocol":"TCP","sourcePort":9090}]]'
  creationTimestamp: null
  generateName: linkding-5c77d5f4cf-
  labels:
    pod-template-hash: 5c77d5f4cf
    workload.user.cattle.io/workloadselector: deployment-default-linkding
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:cattle.io/timestamp: {}
          f:field.cattle.io/ports: {}
        f:generateName: {}
        f:labels:
          .: {}
          f:pod-template-hash: {}
          f:workload.user.cattle.io/workloadselector: {}
        f:ownerReferences:
          .: {}
          k:{"uid":"762057a3-c1d7-4fa1-91ca-01e961fbc72c"}:
            .: {}
            f:apiVersion: {}
            f:blockOwnerDeletion: {}
            f:controller: {}
            f:kind: {}
            f:name: {}
            f:uid: {}
      f:spec:
        f:containers:
          k:{"name":"linkding"}:
            .: {}
            f:image: {}
            f:imagePullPolicy: {}
            f:livenessProbe:
              .: {}
              f:failureThreshold: {}
              f:initialDelaySeconds: {}
              f:periodSeconds: {}
              f:successThreshold: {}
              f:tcpSocket:
                .: {}
                f:port: {}
              f:timeoutSeconds: {}
            f:name: {}
            f:ports:
              .: {}
              k:{"containerPort":9090,"protocol":"TCP"}:
                .: {}
                f:containerPort: {}
                f:hostPort: {}
                f:name: {}
                f:protocol: {}
            f:readinessProbe:
              .: {}
              f:failureThreshold: {}
              f:initialDelaySeconds: {}
              f:periodSeconds: {}
              f:successThreshold: {}
              f:tcpSocket:
                .: {}
                f:port: {}
              f:timeoutSeconds: {}
            f:resources: {}
            f:securityContext:
              .: {}
              f:allowPrivilegeEscalation: {}
              f:capabilities: {}
              f:privileged: {}
              f:readOnlyRootFilesystem: {}
              f:runAsNonRoot: {}
            f:stdin: {}
            f:terminationMessagePath: {}
            f:terminationMessagePolicy: {}
            f:tty: {}
            f:volumeMounts:
              .: {}
              k:{"mountPath":"/etc/linkding/data"}:
                .: {}
                f:mountPath: {}
                f:name: {}
        f:dnsPolicy: {}
        f:enableServiceLinks: {}
        f:restartPolicy: {}
        f:schedulerName: {}
        f:securityContext: {}
        f:terminationGracePeriodSeconds: {}
        f:volumes:
          .: {}
          k:{"name":"data"}:
            .: {}
            f:hostPath:
              .: {}
              f:path: {}
              f:type: {}
            f:name: {}
    manager: kube-controller-manager
    operation: Update
    time: "2020-09-19T07:58:22Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          f:cni.projectcalico.org/podIP: {}
          f:cni.projectcalico.org/podIPs: {}
    manager: calico
    operation: Update
    time: "2020-09-19T07:58:26Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:status:
        f:conditions:
          k:{"type":"ContainersReady"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Initialized"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Ready"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
        f:containerStatuses: {}
        f:hostIP: {}
        f:phase: {}
        f:podIP: {}
        f:podIPs:
          .: {}
          k:{"ip":"10.42.0.66"}:
            .: {}
            f:ip: {}
        f:startTime: {}
    manager: kubelet
    operation: Update
    time: "2020-09-19T07:59:15Z"
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: linkding-5c77d5f4cf
    uid: 762057a3-c1d7-4fa1-91ca-01e961fbc72c
  selfLink: /api/v1/namespaces/default/pods/linkding-5c77d5f4cf-zlz6h
spec:
  containers:
  - image: sissbruecker/linkding:latest
    imagePullPolicy: Always
    livenessProbe:
      failureThreshold: 3
      initialDelaySeconds: 10
      periodSeconds: 2
      successThreshold: 1
      tcpSocket:
        port: 9090
      timeoutSeconds: 2
    name: linkding
    ports:
    - containerPort: 9090
      hostPort: 9090
      name: port
      protocol: TCP
    readinessProbe:
      failureThreshold: 3
      initialDelaySeconds: 10
      periodSeconds: 2
      successThreshold: 2
      tcpSocket:
        port: 9090
      timeoutSeconds: 2
    resources: {}
    securityContext:
      allowPrivilegeEscalation: false
      capabilities: {}
      privileged: false
      readOnlyRootFilesystem: false
      runAsNonRoot: false
    stdin: true
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    tty: true
    volumeMounts:
    - mountPath: /etc/linkding/data
      name: data
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-zh8ch
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: rancherserver
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - hostPath:
      path: /data
      type: ""
    name: data
  - name: default-token-zh8ch
    secret:
      defaultMode: 420
      secretName: default-token-zh8ch
status:
  phase: Pending
  qosClass: BestEffort
---

# This is what we actually write:

# linkding.yaml
apiVersion: v1
kind: Pod
metadata:
  name: linkding
spec:
  containers:
  - image: sissbruecker/linkding:latest
    name: linkding
    ports:
    - containerPort: 9090
      protocol: TCP

```

#### 1.2.2 Lets create a pod

```bash
# ---Creation----
# The simplest way to create a pod is via the imperative kubectl run command.
$ kubectl run linkding --image=sissbruecker/linkding:latest --port=9090

--or--

# To create a pod from file
$ kubectl apply -f ./linkding.yaml

```

#### 1.2.3 Get details of a pod

```bash
# ---Details---
# Listing pods
$ kubectl get pods
If you ran this command immediately after the Pod was created, you might see:
NAME       READY   STATUS   RESTARTS   AGE
linkding   0/1     Pending   0         1s
# The Pending state indicates that the Pod has been submitted but hasn’t been scheduled yet.

# Get small overview
$ kubectl explain pods
$ kubectl explain pod.spec

# Getting the whole defination
$ kubectl get pod linkding -o yaml

# To get pod details
$ kubectl describe pods linkding
Name:         linkding-5c77d5f4cf-zlz6h
Namespace:    default
Priority:     0
Node:         rancherserver/10.0.1.69
Start Time:   Sat, 19 Sep 2020 07:58:23 +0000
Labels:       pod-template-hash=5c77d5f4cf
              workload.user.cattle.io/workloadselector=deployment-default-linkding
Annotations:  cattle.io/timestamp: 2020-09-19T07:58:22Z
              cni.projectcalico.org/podIP: 10.42.0.120/32
              cni.projectcalico.org/podIPs: 10.42.0.120/32
              field.cattle.io/ports:
                [[{"containerPort":9090,"dnsName":"linkding-hostport","hostPort":9090,"kind":"HostPort","name":"port","protocol":"TCP","sourcePort":9090}]...
Status:       Running
IP:           10.42.0.120
IPs:
  IP:           10.42.0.120
Controlled By:  ReplicaSet/linkding-5c77d5f4cf
Containers:
  linkding:
    Container ID:   docker://5afb45048bae434f94571685f822ba8922d86e829d10c0dd4d067cb7f3889ac2
    Image:          sissbruecker/linkding:latest
    Image ID:       docker-pullable://sissbruecker/linkding@sha256:96089547c4829f6578d35ec2ba4dbfb7afced22c971b692c3a7ed0105ca59af8
    Port:           9090/TCP
    Host Port:      9090/TCP
    State:          Running
      Started:      Wed, 23 Sep 2020 08:22:02 +0000
    Last State:     Terminated
      Reason:       Error
      Exit Code:    137
      Started:      Wed, 23 Sep 2020 08:21:04 +0000
      Finished:     Wed, 23 Sep 2020 08:21:50 +0000
    Ready:          True
    Restart Count:  3
    Liveness:       tcp-socket :9090 delay=10s timeout=2s period=2s #success=1 #failure=3
    Readiness:      tcp-socket :9090 delay=10s timeout=2s period=2s #success=2 #failure=3
    Environment:    <none>
    Mounts:
      /etc/linkding/data from data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-zh8ch (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  data:
    Type:          HostPath (bare host directory volume)
    Path:          /data
    HostPathType:
  default-token-zh8ch:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-zh8ch
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason          Age                     From                    Message
  ----     ------          ----                    ----                    -------
  Normal   SandboxChanged  9m1s                    kubelet, rancherserver  Pod sandbox changed, it will be killed and re-created.
  Normal   Pulling         7m12s                   kubelet, rancherserver  Pulling image "sissbruecker/linkding:latest"
  Normal   Pulled          6m55s                   kubelet, rancherserver  Successfully pulled image "sissbruecker/linkding:latest"
  Normal   Created         6m44s                   kubelet, rancherserver  Created container linkding
  Normal   Started         6m33s                   kubelet, rancherserver  Started container linkding

```

#### 1.2.4 Deletion of pods

```bash
## ---Deletion---
# deleting pod by a name
$ kubectl delete pod linkding

# Using the yaml, which is used when creating a pod
$ kubectl delete -f ./linkding.yaml

# delete multiple pods by a name
$ kubectl delete pods pod1 pod2

# delete pods using the label selectors
$ kubectl delete pod -l creation_method=manual

# deleting pods by deleting the whole namespace
$ kubectl delete ns custom-namespace

# delete all pods in a namespace
$ kubectl delete pods --all

# delete all resources in a namespace
$ kubectl delete all --all
```

#### 1.2.5 Accessing the pods

```bash
# ---Accessing the pod---

# Using the port forwarding
$ kubectl port-forward linkding 9090:9090
# A secure tunnel is created from your local machine, through the Kubernetes master, to the instance of the Pod running on one of the worker nodes. As long as the port-forward command is still running, you can access the Pod (in this case the kuard web interface) at http://localhost:9090.


# Get the logs
# When your application needs debugging, it’s helpful to be able to dig deeper than describe to understand what the application is doing. Kubernetes provides two commands for debugging running containers. The kubectl logs command downloads the current logs from the running instance:
$ kubectl logs linkding

# If you are using multicontainer pods ( use container name )
$ kubectl logs linkding -c <container>

# Running commands in container with exec
# Sometimes logs are insufficient, and to truly determine what’s going on you need to execute commands in the context of the container itself.
$ kubectl exec -it linkding date

------------------------------------------------------------------------------------------
# ---Copying files to and from containers---
# At times you may need to copy files from a remote container to a local machine for more in-depth exploration.
$ kubectl cp <pod-name>:/website/main.js ./main.js
$ kubectl cp $HOME/config.txt <pod-name>:/config.txt
# Copying files into a container is an anti-pattern.

```

#### 1.2.6 Health Checks

```bash
# ---Health Checks---
# When you run your application as a container in Kubernetes, it is automatically kept alive for you using a process health check. This health check simply ensures that the main process of your application is always running. If it isn’t, Kubernetes restarts it.

# Add healthcheck
apiVersion: v1
kind: Pod
metadata:
  name: linkding
spec:
  containers:
  - image: sissbruecker/linkding:latest
    name: linkding
    livenessProbe:
      httpGet:
        path: /healthy
        port: 9090
      initialDeleySeconds: 5
      timeoutSeconds: 1
      periodSeconds: 10
      failureThreshold: 3
    ports:
    - containerPort: 9090
      protocol: TCP

# The preceding Pod manifest uses an httpGet probe to perform an HTTP GET request against the /healthy endpoint on port 9090 of the kuard container.
# The probe sets an initialDelaySeconds of 5, and thus will not be called until 5 seconds after all the containers in the Pod are created. The probe must respond within the 1-second time‐out, and the HTTP status code must be equal to or greater than 200 and less than 400 to be considered successful.
# Kubernetes will call the probe every 10 seconds. If more than three consecutive probes fail, the container will fail and restart.

# Types of Health Checks
# In addition to HTTP checks, Kubernetes also supports tcpSocket health checks that open a TCP socket; if the connection is successful, the probe succeeds. This style of probe is useful for non-HTTP applications; for example, databases or other non– HTTP-based APIs.
# Finally, Kubernetes allows exec probes. These execute a script or program in the context of the container. Following typical convention, if this script returns a zero exit code, the probe succeeds; otherwise, it fails. exec scripts are often useful for custom application validation logic that doesn’t fit neatly into an HTTP call.

```

#### 1.2.7 Resource Management

```bash
# ---Resource Management---

## Resource Requests: Minimum Required Resources

# A Pod requests the resources required to run its containers. Kubernetes guarantees that these resources are available to the Pod. The most commonly requested resources are CPU and memory, but Kubernetes has support for other resource types as well, such as GPUs and more

## Resource limits

# Requests are used when scheduling Pods to nodes. The Kubernetes scheduler will ensure that the sum of all requests of all Pods on a node does not exceed the capacity of the node

# Therefore, a Pod is guaranteed to have at least the requested resources when running on the node. Importantly, “request” specifies a minimum. It does not specify a maximum cap on the resources a Pod may use

---
apiVersion: v1
kind: Pod
metadata:
  name: linkding
spec:
  containers:

- image: sissbruecker/linkding:latest
    name: linkding
    resources:
      requests:
        cpu: "500m"
        memory: "128Mi"
      limits:
        cpu: "1000m"
        memory: "256Mi"
    livenessProbe:
      httpGet:
        path: /healthy
        port: 9090
      initialDeleySeconds: 5
      timeoutSeconds: 1
      periodSeconds: 10
      failureThreshold: 3
    ports:
  - containerPort: 9090
      protocol: TCP
```

#### 1.2.8 Persistent Volumes with Pods

```bash
# ---Persisting Data with Volumes---

# When a Pod is deleted or a container restarts, any and all data in the container’s file‐system is also deleted

# This is often a good thing, since you don’t want to leave around cruft that happened to be written by your stateless web application

# In other cases, having access to persistent disk storage is an important part of a healthy application

# Kubernetes models such persistent storage

---
apiVersion: v1
kind: Pod
metadata:
  name: linkding
spec:
  containers:

- image: sissbruecker/linkding:latest
    name: linkding
    resources:
      requests:
        cpu: "500m"
        memory: "128Mi"
      limits:
        cpu: "1000m"
        memory: "256Mi"
    livenessProbe:
      httpGet:
        path: /healthy
        port: 9090
      initialDeleySeconds: 5
      timeoutSeconds: 1
      periodSeconds: 10
      failureThreshold: 3
    volumeMounts:
  - mountPath: "/data"
        name: "linkding-data"
    ports:
  - containerPort: 9090
      protocol: TCP

# Different ways of using volumes with pods

## Communication/Synchronization

# To achieve this, the Pod uses an emptyDir volume. Such a volume is scoped to the Pod’s lifespan, but it can be shared between two containers.

-----------------------------------------------------------------------------------------
## Cache

# An application may use a volume that is valuable for performance, but not required for correct operation of the application

# For example, perhaps the application keeps prerendered thumbnails of larger images. Of course, they can be reconstructed from the original images, but that makes serving the thumbnails more expensive

# You want such a cache to survive a container restart due to a health-check failure, and thus emptyDir works well for the cache use case as well

-----------------------------------------------------------------------------------------
## Persitent Data

# Sometimes you will use a volume for truly persistent data—data that is independent of the lifespan of a particular Pod, and should move between nodes in the cluster if a node fails or a Pod moves to a different machine for some reason

# To achieve this, Kubernetes supports a wide variety of remote network storage volumes, including widely supported protocols like NFS and iSCSI as well as cloud provider network storage like Amazon’s Elastic Block Store, Azure’s Files and Disk Storage, as well as Google’s Persistent Disk

-----------------------------------------------------------------------------------------
## Mounting the host filesystem

# Other applications don’t actually need a persistent volume, but they do need some access to the underlying host filesystem. For example, they may need access to the /dev filesystem in order to perform raw block-level access to a device on the system

# For these cases, Kubernetes supports the hostPath volume, which can mount arbitrary locations on the worker node into the container. The previous example uses the hostPath volume type.

-----------------------------------------------------------------------------------------
## Persisting Data Using Remote Disks

# Oftentimes, you want the data a Pod is using to stay with the Pod, even if it is restarted on a different host machine

# To achieve this, you can mount a remote network storage volume into your Pod. When using network-based storage, Kubernetes automatically mounts and unmounts the appropriate storage whenever a Pod using that volume is scheduled onto a particular machine

# There are numerous methods for mounting volumes over the network

# Kubernetes includes support for standard protocols such as NFS and iSCSI as well as cloud provider–based storage APIs for the major cloud providers (both public and private)

# In many cases, the cloud providers will also create the disk for you if it doesn’t already exist

volumes:

- name: "linkding"
    nfs:
      server: my.nfs.server.local
      path: "/exports"
```

---
