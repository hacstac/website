---
title: "Kubernetes Volumes"
date: 2020-10-25T15:43:48+08:00
tags: ["Kubernetes", "DevOps"]
categories: ["DevOps"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

## 1.0 Kubernetes Volumes: Attaching disk storage to containers

Kubernetes volumes are a component of a pod and are thus defined in the pod’s specification much like containers. They aren’t a standalone Kubernetes object and cannot be created or deleted on their own. A volume is available to all containers in the pod, but it must be mounted in each container that needs to access it. In each container, you can mount the volume in any location of its filesystem

---

### 1.1 Why we need persistant volumes with pods

In kubernetes each container in a pod has its own isolated filesystem, because the filesystem comes from the container’s image. Every new container starts off with the exact set of files that was added to the image at build time. Combine this with the fact that containers in a pod get restarted (either because the process died or because the liveness probe signaled to Kubernetes that the container wasn’t healthy anymore) and you’ll realize that the new container will not see anything that was written to the filesystem by the previous container, even though the newly started container runs in the same pod.

In certain scenarios you want the new container to continue where the last one finished, such as when restarting a process on a physical machine. You may not need (or want) the whole filesystem to be persisted, but you do want to preserve the directories that hold actual data.

Kubernetes provides this by defining storage volumes. They aren’t top-level resources like pods, but are instead defined as a part of a pod and share the same lifecycle as the pod. This means a volume is created when the pod is started and is destroyed when the pod is deleted. Because of this, a volume’s contents will persist across container restarts. After a container is restarted, the new container can see all the files that were written to the volume by the previous container. Also, if a pod contains multiple containers, the volume can be used by all of them at once.

---

### 1.2 Volume Types

A wide variety of volume types is available. Several are generic, while others are specific to the actual storage technologies used underneath. Here’s a list of
several of the available volume types:

- `emptyDir` — A simple empty directory used for storing transient data.
- `hostPath` — Used for mounting directories from the worker node’s filesystem
into the pod.
- `gitRepo` — A volume initialized by checking out the contents of a Git repository.
- `nfs` — An NFS share mounted into the pod.
- `gcePersistentDisk` (Google Compute Engine Persistent Disk), `awsElasticBlockStore` (Amazon Web Services Elastic Block Store Volume), `azureDisk` (Microsoft Azure Disk Volume)—Used for mounting cloud provider-specific storage.
- `cinder`, `cephfs`, `iscsi`, `flocker`, `glusterfs`, `quobyte`, `rbd`, `flexVolume`, `vsphere-Volume`, `photonPersistentDisk`, scaleIO—Used for mounting other types of network storage.
- `configMap`, `secret`, `downwardAPI` — Special types of volumes used to expose certain Kubernetes resources and cluster information to the pod.
- `persistentVolumeClaim` — A way to use a pre- or dynamically provisioned per`sistent storage.

These volume types serve various purposes. You’ll learn about some of them in the
following sections. Special types of volumes ( secret, downwardAPI, configMap) are covered in the next phases, because they aren’t used for storing data, but for exposing Kubernetes metadata to apps running in the pod.

#### 1.2.1 emptyDir Volume

The simplest volume type is the emptyDir volume. The app running inside the pod can then write any files it needs to it. Because the volume’s lifetime is tied to that of the pod, the volume’s contents are lost when the pod is deleted. An emptyDir volume is especially useful for sharing files between containers running in the same pod. But it can also be used by a single container for when a container needs to write data to disk temporarily, such as when performing a sort operation on a large dataset, which can’t fit into the available memory. The data could also be written to the container’s filesystem itself (remember the top read-write layer in a container?), but subtle differences exist between the two options. A container’s filesystem may not even be writable.

- Ex : The pod contains two containers and a single volume that’s mounted in both of them, yet at different paths. When the html-generator container starts, it starts writing the output of the fortune command to the /var/htdocs/index.html file every 10 seconds. Because the volume is mounted at /var/htdocs, the index.html file is writ- ten to the volume instead of the container’s top layer. As soon as the web-server container starts, it starts serving whatever HTML files are in the /usr/share/nginx/html directory (this is the default directory Nginx serves files from). Because you mounted the volume in that exact location, Nginx will serve the index.html file written there by the container running the fortune loop. The end effect is that a client sending an HTTP request to the pod on port 80 will receive the current fortune message as the response.

The emptyDir you used as the volume was created on the actual disk of the worker node hosting your pod, so its performance depends on the type of the node’s disks.

```bash
---yaml
apiVersion: v1
kind: Pod
metadata:
  name: fortune
spec:
  containers:
  - image: luksa/fortune
    name: html-generator
    volumeMounts:
    - name: html
      mountPath: /var/htdocs
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
      readOnly: true
    ports:
    - containerPort: 80
      protocol: TCP
    volumes:
    - name: html
    emptyDir: {}
---

$ kubectl apply -f ./fortune.yaml
$ kubectl port-forward fortune 8080:80

$ curl http://localhost:8080
Beware of a tall blond man with one black shoe.
# If you wait a few seconds and send another request, you should receive a different message.

# To use emptyDir on tmpfs ( on ram instead of disk )
volumes:
  - name: html
    emptyDir:
      medium: Memory
```

#### 1.2.2 Git Repo

A gitRepo volume is basically an emptyDir volume that gets populated by cloning a
Git repository and checking out a specific revision when the pod is starting up (but
before its containers are created).

For example, you can use a Git repository to store static HTML files of your website
and create a pod containing a web server container and a gitRepo volume. Every time
the pod is created, it pulls the latest version of your website and starts serving it. The only drawback to this is that you need to delete the pod every time you push changes to the gitRepo and want to start serving the new version of the website.

Note: You need a github repo for the website/App

When you create the pod, the volume is first initialized as an empty directory and then the specified Git repository is cloned into it. If you hadn’t set the directory to . (dot), the repository would have been cloned into the kubia-website-example subdirectory, which isn’t what you want. You want the repo to be cloned into the root directory of your volume. Along with the repository, you also specified you want Kubernetes to check out whatever revision the master branch is pointing to at the time the volume is created.

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: gitrepo-volume-pod
spec:
  containers:
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
      readOnly: true
    ports:
    - containerPort: 80
      protocol: TCP
  volumes:
  - name: html
    gitRepo:
      repository: https://github.com/luksa/kubia-website-example.git
      revision: master
      directory: .
```

To keep your app sync with github, you'll need a sidecar container because git sync process shouldn’t run in the same container as the Nginx web server, but in a second container: a sidecar container. A sidecar container is a container that augments the operation of the main container of the pod. You add a sidecar to a pod so you can use an existing container image instead of cramming additional logic into the main app’s code, which would make it overly complex and less reusable. To find an existing container image, which keeps a local directory synchronized with a Git repository, go to Docker Hub and search for “git sync.” You’ll find many images that do that. Then use the image in a new container in the pod from the previous example, mount the pod’s existing gitRepo volume in the new container, and configure the Git sync container to keep the files in sync with your Git repo. If you set everything up correctly, you should see that the files the web server is serving are kept in sync with your GitHub repo.

A gitRepo volume, like the emptyDir volume, is basically a dedicated directory created specifically for, and used exclusively by, the pod that contains the volume. When the pod is deleted, the volume and its contents are deleted. Other types of volumes, however, don’t create a new directory, but instead mount an existing external directory into the pod’s container’s filesystem. The contents of that volume can survive multiple pod instantiations.

#### 1.2.3 hostPath Volume

Most pods should be oblivious of their host node, so they shouldn’t access any files on the node’s filesystem. But certain system-level pods (remember, these will usually be managed by a DaemonSet) do need to either read the node’s files or use the node’s
filesystem to access the node’s devices through the filesystem. Kubernetes makes this
possible through a hostPath volume. A hostPath volume points to a specific file or directory on the node’s filesystem.

hostPath volumes are the first type of persistent storage we’re introducing, because both the gitRepo and emptyDir volumes’ contents get deleted when a pod is torn down, whereas a hostPath volume’s contents don’t. If a pod is deleted and the next pod uses a hostPath volume pointing to the same path on the host, the new pod will see whatever was left behind by the previous pod, but only if it’s scheduled to the same node as the first pod.

If you’re thinking of using a hostPath volume as the place to store a database’s data directory, think again. Because the volume’s contents are stored on a specific node’s filesystem, when the database pod gets rescheduled to another node, it will no longer see the data. This explains why it’s not a good idea to use a hostPath volume for regular pods, because it makes the pod sensitive to what node it’s scheduled to.

```bash
# To check existing pods using the hostPath or not
$ kubectl describe pod kubia-x-x --namespace kube-system
```

#### 1.2.4 Persistent Storage

When an application running in a pod needs to persist data to disk and have that same data available even when the pod is rescheduled to another node, you can’t use any of the volume types we’ve mentioned so far. Because this data needs to be accessible from any cluster node, it must be stored on some type of network-attached storage (NAS).

To learn about volumes that allow persisting data, you’ll create a pod that will run the MongoDB document-oriented NoSQL database. Running a database pod without a volume or with a non-persistent volume doesn’t make sense, except for testing purposes, so you’ll add an appropriate type of volume to the pod and mount it in the MongoDB container.

for example : we can use google kubernetes engine, where we can use GCE persistent disk as underlying storage mechanism.

Ex : Creating a GCE Persistent Disk in a pod volume

```bash
# Create a cluster
$ gcloud compute disks create --size=1GB --zone=southeast-asia1-a mongodb

# Create a pod using gcePersistentDisk
---yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  volumes:
  - name: mongodb-data
    gcePersistentDisk:
    pdName: mongodb
    fsType: ext4
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - name: mongodb-data
      mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP

# Spin up the pod
$ kubectl apply -f ./mongodb.yaml

# Entering the mongoDB Shell
$ kubectl exec -it mongodb mongo
```

If you delete the pod and recreate it. Mongo uses exact same data, because it uses the persistent disk. This is an example of GCE, If your Kubernetes cluster is running on Amazon’s AWS EC2, for example, you can use an awsElasticBlockStore volume to provide persistent storage for your pods. If your cluster runs on Microsoft Azure, you can use the azureFile or the azureDisk volume.

#### 1.2.5 NFS Storage

If your cluster is running on your own set of servers, you have a vast array of other sup- ported options for mounting external storage inside your volume. For example, to mount a simple NFS share, you only need to specify the NFS server and the path exported by the server.

```yaml
volumes:
  - name: mongodb-data
    nfs:
      server: 11.151.150.124
      path: /some/path
```

#### 1.2.6 Other Storage Technologies

Other supported options include iscsi for mounting an ISCSI disk resource, glusterfs for a GlusterFS mount, rbd for a RADOS Block Device, flexVolume , cinder, cephfs, flocker, fc (Fibre Channel), and others. You don’t need to know all of them if you’re not using them. They’re mentioned here to show you that Kubernetes supports a broad range of storage technologies and you can use whichever you prefer and are used to.

To see details on what properties you need to set for each of these volume types,
you can either turn to the Kubernetes API definitions in the Kubernetes API refer-
ence or look up the information through kubectl explain.

---

### 1.3 DeepDive with Kubernetes Storage Technology

To enable apps to request storage in a Kubernetes cluster without having to deal with infrastructure specifics, two new resources were introduced. They are PersistentVolumes and PersistentVolumeClaims. The names may be a bit misleading, because as you’ve seen in the previous few sections, even regular Kubernetes volumes can be used to store persistent data.

Using a PersistentVolume inside a pod is a little more complex than using a regular
pod volume, so let’s illustrate how pods, PersistentVolumeClaims, PersistentVolumes,
and the actual underlying storage relate to each other.

![PersistentVolume](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Kubernetes/PersistentStorage.png)

**Explaination**: Instead of the developer adding a technology-specific volume to their pod, it’s the cluster administrator who sets up the underlying storage and then registers it in Kubernetes by creating a PersistentVolume resource through the Kubernetes API server. When creating the PersistentVolume, the admin specifies its size and the access modes it supports. When a cluster user needs to use persistent storage in one of their pods, they first create a PersistentVolumeClaim manifest, specifying the minimum size and the access mode they require. The user then submits the PersistentVolumeClaim manifest to the Kubernetes API server, and Kubernetes finds the appropriate PersistentVolume and binds the volume to the claim. The PersistentVolumeClaim can then be used as one of the volumes inside a pod. Other users cannot use the same PersistentVolume until it has been released by deleting the bound PersistentVolumeClaim.

#### 1.3.1 Creating a PersistentVolume

When creating a PersistentVolume, the administrator needs to tell Kubernetes what its capacity is and whether it can be read from and/or written to by a single node or by multiple nodes at the same time. They also need to tell Kubernetes what to do with the PersistentVolume when it’s released (when the PersistentVolumeClaim it’s bound to is deleted. And last, but certainly not least, they need to specify the type, location, and other properties of the actual storage this PersistentVolume is backed by.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  gcePersistentDisk:
    pdName: monodb
    fsType: ext4
```

Note: PersistentVolumes don't belong to any namespaces

#### 1.3.2 PersistentVolumeClaim

Now let’s lay down our admin hats and put our developer hats back on. Say you need to deploy a pod that requires persistent storage. You’ll use the PersistentVolume you created earlier. But you can’t use it directly in the pod. You need to claim it first. Claiming a PersistentVolume is a completely separate process from creating a pod, because you want the same PersistentVolumeClaim to stay available even if the pod is rescheduled (remember, rescheduling means the previous pod is deleted and a new one is created).

As soon as you create the claim, Kubernetes finds the appropriate PersistentVolume and binds it to the claim. The PersistentVolume’s capacity must be large enough to accommodate what the claim requests. Additionally, the volume’s access modes must include the access modes requested by the claim. In your case, the claim requests 1 GiB of storage and a ReadWriteOnce access mode. The PersistentVolume you created earlier matches those two requirements so it is bound to your claim.

The claim is shown as Bound to PersistentVolume mongodb-pv. Note the abbreviations
used for the access modes:

- RWO—ReadWriteOnce—Only a single node can mount the volume for reading and writing.
- ROX—ReadOnlyMany—Multiple nodes can mount the volume for reading.
- RWX—ReadWriteMany—Multiple nodes can mount the volume for both reading and writing.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  resources:
    requests:
      storage: 1Gi
  accessModes:
  - ReadWriteOnce
  storageClassName:""
```

#### 1.3.3 Create a Pod using PVC

The PersistentVolume is now yours to use. Nobody else can claim the same volume
until you release it. To use it inside a pod, you need to reference the Persistent-
VolumeClaim by name inside the pod’s volume (yes, the PersistentVolumeClaim, not
the PersistentVolume directly!).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - name: mongodb-data
      mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP
  volumes:
  - name: mongodb-data
    persistentVolumeClaim:
      claimName: mongodb-pvc
```

```bash
# List out the PersistentVolumes
$ kubectl get pv

# List out the PersistentVolumesClaims
$ kubectl get pvc
```

#### 1.3.4 Recycling PersistentVolumes

If you delete the pod and the pvc and create a new PVC on it, It will show a pending state because The STATUS column shows the PersistentVolume as Released, not Available like before. Because you’ve already used the volume, it may contain data and shouldn’t be bound to a completely new claim without giving the cluster admin a chance to clean it up. Without this, a new pod using the same PersistentVolume could read the data stored there by the previous pod, even if the claim and pod were created in a different namespace (and thus likely belong to a different cluster tenant).

To reclaim the pv, there are two policies exist: Recycle and Delete. The first one deletes
the volume’s contents and makes the volume available to be claimed again. This way, the PersistentVolume can be reused multiple times by different PersistentVolumeClaims and different pods. The Delete policy, on the other hand, deletes the underlying storage. Note that the Recycle option is currently not available for GCE Persistent Disks.

A PersistentVolume only supports the Retain or Delete policies. Other PersistentVolume types may or may not support each of these options, so before creating your own PersistentVolume, be sure to check what reclaim policies are supported for the specific underlying storage you’ll use in the volume.

#### 1.3.5 Dynamic provisioning of PersistentVolumes

You’ve seen how using PersistentVolumes and PersistentVolumeClaims makes it easy to obtain persistent storage without the developer having to deal with the actual stor- age technology used underneath. But this still requires a cluster administrator to pro- vision the actual storage up front. Luckily, Kubernetes can also perform this job automatically through dynamic provisioning of PersistentVolumes. The cluster admin, instead of creating PersistentVolumes, can deploy a Persistent- Volume provisioner and define one or more StorageClass objects to let users choose what type of PersistentVolume they want. The users can refer to the StorageClass in their PersistentVolumeClaims and the provisioner will take that into account when provisioning the persistent storage.

![dynamicPeristentVolume](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Kubernetes/dynamicPV.png)

```bash
---storageClass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  zone: asia-southest1-b

---PersistentVolume.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  storageClassName: fast
  resources:
    requests:
      storage: 100Mi
  accessModes:
    - ReadWriteOnce

$ kubectl get sc
NAME                     TYPE
fast                     kubernetes.io/gce-pd
standard (default)       kubernetes.io/gce-pd

$ kubectl get pv
NAME          CAPACITY      ACCESSMODES      RECLAIMPOLICY      STATUS      STORAGECLASS
mongodb-pv    1Gi           RWO,ROX          Retain             Released
pvc-1e6bc048  1Gi           RWO              Delete             Bound       fast

$ kubectl get pvc mongodb-pvc
NAME         STATUS      VOLUME       CAPACITY      ACCESSMODES      STORAGECLASS
mongodb-pvc  Bound      pvc-1e6bc048  1Gi           RWO              fast
```

---
