---
title: "Kubernetes Controllers"
date: 2020-10-22T15:43:48+08:00
tags: ["Kubernetes", "DevOps"]
categories: ["DevOps"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

### 1.0 Replication Controller [ Deprecated ]

A ReplicationController is a Kubernetes resource that ensures its pods are always kept running. If the pod disappears for any reason, such as in the event of a node disappearing from the cluster or because the pod was evicted from the node, the ReplicationController notices the missing pod and creates a replacement pod.

<!--more-->

![Replication Controller](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Kubernetes/ReplicationController.png)

In the figure we saw that if a node goes down and takes two pods with it. Pod A was created directly and is therefore an unmanaged pod, while pod B is managed by a ReplicationController. After the node fails, the ReplicationController creates a
new pod (pod B2) to replace the missing pod B, whereas pod A is lost completely—
nothing will ever recreate it.

### 1.1 The opration of a Replication Controller

A ReplicationController constantly monitors the list of running pods and makes sure the actual number of pods of a “type” always matches the desired number. If too few such pods are running, it creates new replicas from a pod template. If too many such pods are running, it removes the excess replicas.

### 1.2 Three main parts of Replication Controller

- A label selector, which determines what pods are in the ReplicationController’s scope
- A replica count, which specifies the desired number of pods that should be running
- A pod template, which is used when creating new pod replicas

### 1.3 Controller Reconciliation Loop

A ReplicationController’s job is to make sure that an exact number of pods always
matches its label selector. If it doesn’t, the ReplicationController takes the appropriate action to reconcile the actual with the desired number.

![Reconciliation](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Kubernetes/ReplicationController_Reconciliation.png)

### 1.4 Effect of changing the controller's label selector or pod template

Changes to the label selector and the pod template have no effect on existing pods. Changing the label selector makes the existing pods fall out of the scope of the ReplicationController, so the controller stops caring about them. ReplicationCon- trollers also don’t care about the actual “contents” of its pods (the container images, environment variables, and other things) after they create the pod. The template therefore only affects new pods created by this ReplicationController.

### 1.5 Benefits of Replication Controller

- It makes sure a pod (or multiple pod replicas) is always running by starting a
new pod when an existing one goes missing.
- When a cluster node fails, it creates replacement replicas for all the pods that
were running on the failed node (those that were under the Replication-
Controller’s control).
- It enables easy horizontal scaling of pods—both manual and automatic

### 1.6 Lets get into the Replication Controller

```bash
# Create a YAML file to create a Replication Controller

---cyberchef_rc.yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: cyberchef
spec:
  replicas: 3
  selector:
    app: cyberchef
  template:
    metadata:
      labels:
        app: cyberchef
    spec:
      containers:
        - name: cyberchef
          image: mpepping/cyberchef
          ports:
            - containerPort: 8000

$ kubectl create -f ./cyberchef_rc.yml

------------------------------------------------------------------------------------------
# Get Pods
$ kubectl get pods
NAME              READY   STATUS    RESTARTS   AGE
cyberchef-gzg6n   1/1     Running   0          5m11s
cyberchef-h4crn   1/1     Running   0          5m11s
cyberchef-vlp7f   1/1     Running   0          5m11s

# Trying to delete the pod
$ kubectl delete pod cyberchef-vlp7f

# Replication Controller Immediately spins up a new pod
$ kubectl get pods
NAME              READY   STATUS    RESTARTS   AGE
cyberchef-gzg6n   1/1     Running   0          6m30s
cyberchef-h4crn   1/1     Running   0          6m30s
cyberchef-hs9cp   1/1     Running   0          9s

------------------------------------------------------------------------------------------
# Getting details of replication controller
$ kubectl get rc
NAME        DESIRED   CURRENT   READY   AGE
cyberchef   3         3         3       8m16s

------------------------------------------------------------------------------------------
# Getting additional details about rc
$ kubectl describe rc
Name:         cyberchef
Namespace:    default
Selector:     app=cyberchef
Labels:       app=cyberchef
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=cyberchef
  Containers:
   cyberchef:
    Image:        mpepping/cyberchef
    Port:         8000/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                    Message
  ----    ------            ----   ----                    -------
  Normal  SuccessfulCreate  9m33s  replication-controller  Created pod: cyberchef-vlp7f
  Normal  SuccessfulCreate  9m33s  replication-controller  Created pod: cyberchef-h4crn
  Normal  SuccessfulCreate  9m33s  replication-controller  Created pod: cyberchef-gzg6n
  Normal  SuccessfulCreate  3m12s  replication-controller  Created pod: cyberchef-hs9cp
```

### 1.7 Moving pods in and out of the scope of a ReplicationController

Pods created by a ReplicationController aren’t tied to the ReplicationController in any way. At any moment, a ReplicationController manages pods that match its label selector. By changing a pod’s labels, it can be removed from or added to the scope of a ReplicationController. It can even be moved from one ReplicationController to another.

If you change a pod’s labels so they no longer match a ReplicationController’s label selector, the pod becomes like any other manually created pod. It’s no longer managed by anything. If the node running the pod fails, the pod is obviously not rescheduled. But keep in mind that when you changed the pod’s labels, the replication controller noticed one pod was missing and spun up a new pod to replace it.

There in below example, we now have four pods altogether: one that isn’t managed by our ReplicationController and three that are. Among them is the newly created pod. After we change the pod’s label from app=cyberchef to app=testing, the ReplicationController no longer cares about the pod. Because the controller’s replica count is set to 3 and only two pods match the label selector so replication controller will spin a new pod and this changed lebal pod is keep running but no one can manage this pod, so we can test this and perform operation on this.

```bash
$ kubectl get pods
NAME              READY   STATUS    RESTARTS   AGE
cyberchef-gbrgh   1/1     Running   0          12h
cyberchef-h728j   1/1     Running   0          12h
cyberchef-npql7   1/1     Running   0          12h

$ kubectl label pod cyberchef-h728j type=vulnerable

$ kubectl get pods --show-labels
NAME              READY   STATUS    RESTARTS   AGE   LABELS
cyberchef-gbrgh   1/1     Running   0          12h   app=cyberchef
cyberchef-h728j   1/1     Running   0          12h   app=cyberchef,type=vulnerable
cyberchef-npql7   1/1     Running   0          12h   app=cyberchef

$ kubectl label pod cyberchef-h728j app=testing --overwrite

$ kubectl get pods -L app

NAME              READY   STATUS    RESTARTS   AGE     APP
cyberchef-gbrgh   1/1     Running   0          13h     cyberchef
cyberchef-h728j   1/1     Running   0          13h     testing
cyberchef-kwn7r   1/1     Running   0          2m44s   cyberchef
cyberchef-npql7   1/1     Running   0          13h     cyberchef

$ kubectl get rc
NAME        DESIRED   CURRENT   READY   AGE
cyberchef   3         3         3       13h
```

### 1.8 Changing pod template

A ReplicationController’s pod template can be modified at any time. Changing the pod
template is like replacing a cookie cutter with another one. It will only affect the cookies you cut out afterward and will have no effect on the ones you’ve already cut. To modify the old pods, you’d need to delete them and let the ReplicationController replace them with new ones based on the new template.

Editing a ReplicationController like this to change the container image in the pod
template, deleting the existing pods, and letting them be replaced with new ones from
the new template could be used for upgrading pods

```bash
# This will open the ReplicationController’s YAML definition in your default text editor.
# Find the pod template section and add an additional label to the metadata
#  After you save your changes and exit the editor, kubectl will update the ReplicationController and print the following message:  replicationcontroller/cyberchef edited
$ kubectl edit rc cyberchef
```

### 1.9 Horizontally scaling pods

Scaling the number of pods up or down is as easy as changing the value of the rep- licas field in the ReplicationController resource. After the change, the Replication- Controller will either see too many pods exist (when scaling down) and delete part of them, or see too few of them (when scaling up) and create additional pods.

```bash
# By kubectl command
$ kubectl scale rc cyberchef --replicas=4

# By changing Yaml
$ kubeclt edit rc cyberchef
# Edit replicas section

$ kubectl get rc
NAME        DESIRED   CURRENT   READY   AGE
cyberchef   4         4         4       2m
```

### 1.10 Deleting a Replication Controller

```bash
# This will delete rc as well all the pods
$ kubectl delete rc cyberchef

# If you want to keep running your pods and delete the replication controller
$ kubectl delete rc cyberchef --cascade=false
# Now pods are running their own
```

---

## 2.0 ReplicaSets

A ReplicaSet acts as a cluster-wide Pod manager, ensuring that the right types and number of Pods are running at all times.

Initially, ReplicationControllers were the only Kubernetes component for replicating
pods and rescheduling them when nodes failed. Later, a similar resource called a
ReplicaSet was introduced. It’s a new generation of ReplicationController and
replaces it completely (ReplicationControllers will eventually be deprecated). ReplicaSet also use same policy as replicationController, ReplicaSets create and manage Pods, they do not own the Pods they create. ReplicaSets use label queries to identify the set of Pods they should be managing.

If we going in deep detail, Despite the value placed on declarative configuration of software, there are times when it is easier to build something up imperatively. In particular, early on you may be simply deploying a single Pod with a container image without a ReplicaSet manag‐ ing it. But at some point you may want to expand your singleton container into a replicated service and create and manage an array of similar containers. You may have even defined a load balancer that is serving traffic to that single Pod. If ReplicaSets owned the Pods they created, then the only way to start replicating your Pod would be to delete it and then relaunch it via a ReplicaSet. This might be disruptive, as there would be a moment in time when there would be no copies of your container running. However, because ReplicaSets are decoupled from the Pods they manage, you can simply create a ReplicaSet that will “adopt” the existing Pod, and scale out additional copies of those containers. In this way, you can seamlessly move from a single imperative Pod to a replicated set of Pods managed by a ReplicaSet.

### 2.1 Comparing a ReplicaSet to a ReplicationController

A ReplicaSet behaves exactly like a ReplicationController, but it has more expressive pod selectors. Whereas a ReplicationController’s label selector only allows matching pods that include a certain label, a ReplicaSet’s selector also allows matching pods that lack a certain label or pods that include a certain label key, regardless of its value. Also, for example, a single ReplicationController can’t match pods with the label env=production and those with the label env=devel at the same time. It can only match either pods with the env=production label or pods with the env=devel label. But a sin- gle ReplicaSet can match both sets of pods and treat them as a single group. Similarly, a ReplicationController can’t match pods based merely on the presence of a label key, regardless of its value, whereas a ReplicaSet can. For example, a Replica- Set can match all pods that include a label with the key env, whatever its actual value is (you can think of it as env=*).

### 2.2 Reconciliation Loops

The central concept behind a reconciliation loop is the notion of desired state versus observed or current state. Desired state is the state you want. With a ReplicaSet, it is the desired number of replicas and the definition of the Pod to replicate. For example, “the desired state is that there are three replicas of a Pod running the cyberchef”.

In contrast, the current state is the currently observed state of the system. For example, “there are only two cyberchef Pods currently running.” The reconciliation loop is constantly running, observing the current state of the world and taking action to try to make the observed state match the desired state. For instance, with the previous examples, the reconciliation loop would create a new cyberchef Pod in an effort to make the observed state match the desired state of three replicas.

There are many benefits to the reconciliation loop approach to managing state. It is
an inherently goal-driven, self-healing system, yet it can often be easily expressed in a few lines of code. As a concrete example of this, note that the reconciliation loop for ReplicaSets is a single loop, yet it handles user actions to scale up or scale down the ReplicaSet as well as node failures or nodes rejoining the cluster after being absent.

### 2.3 Quarantining Containers

Oftentimes, when a server misbehaves, Pod-level health checks will automatically restart that Pod. But if your health checks are incomplete, a Pod can be misbehaving but still be part of the replicated set. In these situations, while it would work to simply kill the Pod, that would leave your developers with only logs to debug the problem. Instead, you can modify the set of labels on the sick Pod. Doing so will disassociate it from the ReplicaSet (and service) so that you can debug the Pod. The ReplicaSet con‐ troller will notice that a Pod is missing and create a new copy, but because the Pod is still running it is available to developers for interactive debugging, which is significantly more valuable than debugging from logs.

### 2.4 DeepDive with ReplicaSet

You’re creating a resource of type ReplicaSet which has much the same contents as the ReplicationController you created earlier. The only difference is in the selector. Instead of listing labels the pods need to have directly under the selector property, you’re specifying them under selector .matchLabels. This is the simpler (and less expressive) way of defining label selectors in a ReplicaSet.

```bash
# Creating a YAML for ReplicaSet:
---cyberchef_rs.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: cyberchef
  labels:
    app: cyberchef
spec:
  replicas: 3
  selector:
    matchLabels:
      app: cyberchef
  template:
    metadata:
      labels:
        app: cyberchef
    spec:
      containers:
        - name: cyberchef
          image: mpepping/cyberchef:testing
          ports:
            - containerPort: 8000

# To submit the cyberChef replicaSet to the kubernetes API:
$ kubectl apply -f ./cyberchef_rs.yml

------------------------------------------------------------------------------------------
$ kubectl get rs
NAME        DESIRED   CURRENT   READY   AGE
cyberchef   3         3         3       40m

------------------------------------------------------------------------------------------
# To get further details:
$ kubectl describe rs
Name:         cyberchef
Namespace:    default
Selector:     app=cyberchef
Labels:       app=cyberchef
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=cyberchef
  Containers:
   cyberchef:
    Image:        mpepping/cyberchef:testing
    Port:         8000/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  41m   replicaset-controller  Created pod: cyberchef-l68b2
  Normal  SuccessfulCreate  41m   replicaset-controller  Created pod: cyberchef-8hg5p
  Normal  SuccessfulCreate  41m   replicaset-controller  Created pod: cyberchef-qfqrk

------------------------------------------------------------------------------------------
# Finding a replicaSet from a pod
# Method1:  Check for annotation : kubernetes.io/created-by
$ kubectl get pods <pod-name> -o yaml

------------------------------------------------------------------------------------------
# Method2: Check for the kind
$ kubectl get pods cyberchef-qm28m -o yaml | grep kind
kind: ReplicaSet

------------------------------------------------------------------------------------------
# Finding a Set of pods for a replicaSet
# use the selector to filter out the pods that a replicaSet is using
$ kubectl get pods -l app=cyberchef
NAME              READY   STATUS    RESTARTS   AGE
cyberchef-qm28m   1/1     Running   0          15m
cyberchef-r6fff   1/1     Running   0          15m
cyberchef-scjlr   1/1     Running   0          15m

------------------------------------------------------------------------------------------
## Scaling ReplicaSets
# ReplicaSets are scaled up or down by updating the spec.replicas key on the ReplicaSet object stored in Kubernetes.
# Easiest way to achieve this using scale command
$ kubectl scale replicasets cyberchef --replicas=4
# Note : change ReplicaSet configuration file count as best practices, so use this imperative scale option only on emergency sitautions otherwise update the replicas from the files.

------------------------------------------------------------------------------------------
# Autoscaling a ReplicaSet
# If we understand this with a scenario, if the application uses the high cpu and memory kubernetes will automatically scale the application.
# This scaling is based on responses of some custom application metrics.
# Kubernetes can handle all of these scenarios via Horizontal Pod Autoscaling (HPA).

## Autoscaling based on cpu
$ kubectl autoscale rs cyberchef --min=2 --max=5 --cpu-percent=80
# This command creates an autoscaler that scales between two and five replicas with a CPU threshold of 80%.
# To view, modify, or delete this resource you can use the standard kubectl commands and the horizontalpodautoscalers resource.
# horizontalpodautoscalers is quite a bit to type, but it can be shortened to hpa:
$ kubectl get hpa

------------------------------------------------------------------------------------------
# To Delete ReplicaSets
$ kubectl delete rs cyberchef

# If you don’t want to delete the Pods that are being managed by the ReplicaSet, you can set the --cascade flag to false to ensure only the ReplicaSet object is deleted and not the Pods:
$ kubectl delete rs kuard --cascade=false
```

### 2.5 Using the ReplicaSet more expressive label selectors

The main improvements of ReplicaSets over ReplicationControllers are their more expressive label selectors. You intentionally used the simpler matchLabels selector in the first ReplicaSet example to see that ReplicaSets are no different from ReplicationControllers. Now, you’ll rewrite the selector to use the more powerful matchExpressions property.

```yaml
selector:
  matchExpressions:
    - key: app
      operator: In
      values:
        - cyberchef
```

You can add additional expressions to the selector. As in the example, each expression
must contain a key, an operator, and possibly (depending on the operator) a list of
values. You’ll see four valid operators:

- In—Label’s value must match one of the specified values.
- NotIn—Label’s value must not match any of the specified values.
- Exists—Pod must include a label with the specified key (the value isn’t important). When using this operator, you shouldn’t specify the values field.
- DoesNotExist—Pod must not include a label with the specified key. The values
property must not be specified.

If you specify multiple expressions, all those expressions must evaluate to true for the selector to match a pod. If you specify both matchLabels and matchExpressions, all the labels must match and all the expressions must evaluate to true for the pod to match the selector.

---

## 3.0 DaemonSets

DaemonSet:  Running exactly one pod on each node

To run a pod on all cluster nodes, you create a DaemonSet object, which is much like a ReplicationController or a ReplicaSet, except that pods created by a DaemonSet already have a target node specified and skip the Kubernetes Scheduler. Whereas a ReplicaSet (or ReplicationController) makes sure that a desired number of pod replicas exist in the cluster, a DaemonSet doesn’t have any notion of a desired replica count. It doesn’t need it because its job is to ensure that a pod matching its pod selector is running on each node.

A DaemonSet ensures a copy of a Pod is running across a set of nodes in a Kuber‐ netes cluster. DaemonSets are used to deploy system daemons such as log collectors and monitoring agents, which typically must run on every node. DaemonSets share similar functionality with ReplicaSets; both create Pods that are expected to be long-running services and ensure that the desired state and the observed state of the cluster match.

If a node goes down, the DaemonSet doesn’t cause the pod to be created else-
where. But when a new node is added to the cluster, the DaemonSet immediately
deploys a new pod instance to it. It also does the same if someone inadvertently
deletes one of the pods, leaving the node without the DaemonSet’s pod. Like a Replica-
Set, a DaemonSet creates the pod from the pod template configured in it.

You can use labels to run DaemonSet Pods on specific nodes; for example, you may want to run specialized intrusion-detection software on nodes that are exposed to the edge network.

### 3.1 DaemonSet Scheduler

By default a DaemonSet will create a copy of a Pod on every node unless a node selec‐
tor is used, which will limit eligible nodes to those with a matching set of labels. DaemonSets determine which node a Pod will run on at Pod creation time by specifying the nodeName field in the Pod spec. As a result, Pods created by DaemonSets are ignored by the Kubernetes scheduler.

Like ReplicaSets, DaemonSets are managed by a reconciliation control loop that measures the desired state (a Pod is present on all nodes) with the observed state (is the Pod present on a particular node?). Given this information, the DaemonSet controller creates a Pod on each node that doesn’t currently have a matching Pod.

If a new node is added to the cluster, then the DaemonSet controller notices that it is missing a Pod and adds the Pod to the new node.

### 3.2 DeepDive with DaemonSet

DaemonSets are created by submitting a DaemonSet configuration to the Kubernetes API server.

```bash
# we can only run these pods on selected labels, like if some nodes have ssd's and we want to run those pods only on ssd based nodes.
# Like in below example we use node selector to choose only those pods that have ssd's.

--cyberchef_daemonSet.yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: cyberchef
  labels:
    app: cyberchef
spec:
  selector:
    matchLabels:
      app: cyberchef
  template:
    metadata:
      labels:
        app: cyberchef
    spec:
      nodeSelector:
        disk: ssd
      containers:
        - name: cyberchef
          image: mpepping/cyberchef
          ports:
            - containerPort: 8000

# To Supply to Kubernetes API
$ kubectl apply -f ./cyberchef_daemonSet.yml

------------------------------------------------------------------------------------
# Get the DaemonSet
$ kubectl get ds
kubectl get daemonset
NAME        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
cyberchef   2         2         0       2            0           <none>          4s

------------------------------------------------------------------------------------
# Get the pods
$ kubectl get pods
NAME              READY   STATUS    RESTARTS   AGE
cyberchef-dnrx9   1/1     Running   0          53s
cyberchef-gfhcl   1/1     Running   0          53s

------------------------------------------------------------------------------------
# Get further details
$ kubectl describe daemonset cyberchef
Name:           cyberchef
Selector:       app=cyberchef
Node-Selector:  <none>
Labels:         app=cyberchef
Annotations:    deprecated.daemonset.template.generation: 1
Desired Number of Nodes Scheduled: 2
Current Number of Nodes Scheduled: 2
Number of Nodes Scheduled with Up-to-date Pods: 2
Number of Nodes Scheduled with Available Pods: 2
Number of Nodes Misscheduled: 0
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=cyberchef
  Containers:
   cyberchef:
    Image:        mpepping/cyberchef
    Port:         8000/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                  Message
  ----    ------            ----  ----                  -------
  Normal  SuccessfulCreate  111s  daemonset-controller  Created pod: cyberchef-gfhcl
  Normal  SuccessfulCreate  111s  daemonset-controller  Created pod: cyberchef-dnrx9


------------------------------------------------------------------------------------
# Get the nodes
$ kubectl get nodes
NAME           STATUS   ROLES    AGE     VERSION
minikube       Ready    master   2d17h   v1.19.2
minikube-m02   Ready    <none>   2d16h   v1.19.2
minikube-m03   Ready    <none>   5m      v1.19.2

# To check pod running on which node
$ kubectl get pods -o wide
NAME              READY   STATUS    RESTARTS   AGE    IP           NODE           NOMINATED NODE   READINESS GATES
cyberchef-dnrx9   1/1     Running   0          4m5s   172.17.0.2   minikube-m02   <none>           <none>
cyberchef-gfhcl   1/1     Running   0          4m5s   172.17.0.5   minikube       <none>           <none>

------------------------------------------------------------------------------------
# To limiting daemonSets to specific nodes
# Change the labels of the node
$ kubectl label node minikube-m03 disk=hdd --overwrite

# Show Labels
$ kubectl get nodes --show-labels
NAME           STATUS   ROLES    AGE    VERSION   LABELS
minikube       Ready    master   2d17h   v1.19.2  beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disk=ssd ...
minikube-m03   Ready    master   2d17h   v1.19.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disk=hdd ...
minikube-m02   Ready    master   2d17h   v1.19.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disk=ssd ...
```

### 3.3 Limiting DaemonSets to Specific Nodes

The most common use case for DaemonSets is to run a Pod across every node in a Kubernetes cluster. However, there are some cases where you want to deploy a Pod to only a subset of nodes. For example, maybe you have a workload that requires a GPU or access to fast storage only available on a subset of nodes in your cluster. In cases like these, node labels can be used to tag specific nodes that meet workload requirements.

- Adding Lables to Nodes

DaemonSets to specific nodes is to add the desired set of labels to a subset of nodes. This can be achieved using the kubectl label command.

```bash
kubectl label nodes minikube ssd=true

kubectl get nodes --selector ssd=true
NAME           STATUS   AGE     VERSION
minikube       Ready    2d17h   v1.19.2
minikube-m02   Ready    2d17h   v1.19.2
```

- NodeSelector

Node selectors can be used to limit what nodes a Pod can run on in a given Kuber‐
netes cluster. Node selectors are defined as part of the Pod spec when creating a DaemonSet.

```bash
# I specified this on previous cyberchef daemon example
---x.yml
apiVersion:
kind:
metadata:
spec:
  template:
    metadata:
    spec:
      nodeSelector:
        ssd: "true"
```

### 3.4 Updating a DaemonSet

DaemonSets are great for deploying services across an entire cluster, but what about
upgrades? Prior to Kubernetes 1.6, the only way to update Pods managed by a DaemonSet was to update the DaemonSet and then manually delete each Pod that was managed by the DaemonSet so that it would be re-created with the new configuration.

DaemonSets can be rolled out using the same RollingUpdate strategy that deployments use. You can configure the update strategy using the spec.updateStrategy.type field, which should have the value RollingUpdate. When a DaemonSet has an update strategy of RollingUpdate, any change to the spec.template field (or subfields) in the DaemonSet will initiate a rolling update.

The RollingUpdate strategy gradually updates members of a DaemonSet until all of the Pods are running the new configuration. There are two parameters that control the rolling update of a DaemonSet:

- spec.minReadySeconds, which determines how long a Pod must be “ready” before the rolling update proceeds to upgrade subsequent Pods
- spec.updateStrategy.rollingUpdate.maxUnavailable, which indicates how many Pods may be simultaneously updated by the rolling update

Once a rolling update has started, you can use the kubectl rollout commands to
see the current status of a DaemonSet rollout. For example, kubectl rollout status daemonSets cyberchef will show the current rollout status of a DaemonSet named cyberchef.

### 3.5 Deleting a DaemonSet

Deleting a DaemonSet is pretty straightforward using the kubectl delete command. Just be sure to supply the correct name of the DaemonSet you would like to delete:

```bash
# Delete using specification
$ kubectl delete -f ./cyberchef_daemon.yaml

# Delete using daemonSet
$ kubectl delete ds cyberchef
```

---

## 4.0 Running pods that perform a single completable task

ReplicationControllers, ReplicaSets, and DaemonSets run continuous tasks that are never considered completed. Processes in such pods are restarted when they exit. But in a completable task, after its process terminates, it should not be restarted again.

In the event of a node failure, the pods on that node that are managed by a Job will be rescheduled to other nodes the way ReplicaSet pods are. In the event of a failure of the process itself (when the process returns an error exit code), the Job can be configured to either restart the container or not.

For example, Jobs are useful for ad hoc tasks, where it’s crucial that the task finishes properly. You could run the task in an unmanaged pod and wait for it to finish, but in the event of a node failing or the pod being evicted from the node while it is performing its task, you’d need to manually recreate it. Doing this manually doesn’t make sense—especially if the job takes hours to complete.

An example

```bash

---batch_job.yml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  template:
    metadata:
      labels:
        app: batch-job
    spec:
      restartPolicy: Never
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(50)"]

$ kubectl apply -f ./batch_job.yml

# Get the jobs
$ kubectl get jobs
NAME        COMPLETIONS   DURATION   AGE
batch-job   1/1           44s        46s

# Get the pods
$ kubectl get pods
NAME              READY   STATUS      RESTARTS   AGE
batch-job-8tb2j   1/1     Running     0          1m

# Showing the running job
$ kubectl logs batch-job-8tb2j
3.1415926535897932384626433832795028841971693993751

# After completing the job
NAME              READY   STATUS      RESTARTS   AGE
batch-job-8tb2j   0/1     Completed   0          2m

--------------------------------------------------------------------------------------
## If you want to run multiple pods instances in a job
# Jobs may be configured to create more than one pod instance and run them in parallel or sequentially.
# This is done by setting the completions and the parallelism properties in the Job spec
---
apiVersion: batch/v1
kind: Job
metadata:
  name: multi-completion-batch-job
spec:
  completions: 5
  template:
    <template is the same as in listing 4.11>

# This Job will run five pods one after the other. It initially creates one pod, and when the pod’s container finishes, it creates the second pod, and so on, until five pods complete successfully. If one of the pods fails, the Job creates a new pod, so the Job may create more than five pods overall.

--------------------------------------------------------------------------------------
# Instead of running single Job pods one after the other, you can also make the Job run multiple pods in parallel. You specify how many pods are allowed to run in parallel with the parallelism Job spec property, as shown in the following listing.

---
apiVersion: batch/v1
kind: Job
metadata:
  name: multi-completion-batch-job
spec:
  completions: 5
  parallelism: 2
  template:
    <same as in listing 4.11>

--------------------------------------------------------------------------------------
# Scaling a Job : You can even change a Job’s parallelism property while the Job is running.
# This is similar to scaling a ReplicaSet or ReplicationController, and can be done with the kubectl scale command:
$ kubectl scale job batch-job --replicas 3
```

---

## 5.0  Scheduling Jobs to run periodically or once in the future

Job resources run their pods immediately when you create the Job resource. But many
batch jobs need to be run at a specific time in the future or repeatedly in the specified interval. In Linux and UNIX-like operating systems, these jobs are better known as cron jobs. Kubernetes supports them, too.

A cron job in Kubernetes is configured by creating a CronJob resource. The
schedule for running the job is specified in the well-known cron format, so if you’re
familiar with regular cron jobs, you’ll understand Kubernetes’ CronJobs in a matter
of seconds.

At the configured time, Kubernetes will create a Job resource according to the Job template configured in the CronJob object. When the Job resource is created, one or more pod replicas will be created and started according to the Job’s pod template, as you learned in the previous section. There’s nothing more to it.

### 5.1 Creating a Cronjob

```bash

---cronjob.yml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

### 5.2 Understanding how scheduled jobs are run

Job resources will be created from the CronJob resource at approximately the scheduled time. The Job then creates the pods.

It may happen that the Job or pod is created and run relatively late. You may have a hard requirement for the job to not be started too far over the scheduled time. In that case, you can specify a deadline by specifying the startingDeadlineSeconds field in the CronJob.

```bash
# In thi example one of the times the job is supposed to run is 10:30:00.
# If it doesn’t start by 10:30:15 for whatever reason, the job will not run and will be shown as Failed.

---
apiVersion: batch/v1beta1
kind: CronJob
spec:
  schedule: "0,15,30,45 ****"
  startingDeadlineSeconds: 15
```

---
