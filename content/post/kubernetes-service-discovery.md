---
title: "Kubernetes Service Discovery"
date: 2020-10-23T15:43:48+08:00
tags: ["Kubernetes", "DevOps"]
categories: ["DevOps"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

### 1.1 What is Service Discovery

A Kubernetes Service is a resource you create to make a single, constant point of entry to a group of pods providing the same service. Each service has an IP address and port that never change while the service exists.

<!--more-->

Clients can open connections to that IP and port, and those connections are then routed to one of the pods backing that service. This way, clients of a service don’t need to know the location of individual pods providing the service, allowing those pods to be moved around the cluster at any time.

Service-discovery is tool that help to solve the problem of finding which processes are listening at which addresses for which services. A good service-discovery system will enable users to resolve this information quickly and reliably. A good system is also low-latency; clients are updated soon after the information associated with a service changes.

If we understand this with an example: Where you have a frontend web server and a backend data base server. There may be multiple pods that all act as the frontend, but there may only be a single backend database pod. You need to solve two problems to make the system function:

- External clients need to connect to the frontend pods without caring if there’s
only a single web server or hundreds.
- The frontend pods need to connect to the backend database. Because the database runs inside a pod, it may be moved around the cluster over time, causing its IP address to change. You don’t want to reconfigure the frontend pods every time the backend database is moved.

By creating a service for the frontend pods and configuring it to be accessible from outside the cluster, you expose a single, constant IP address through which external clients can connect to the pods. Similarly, by also creating a service for the backend pod, you create a stable address for the backend pod. The service address doesn’t change even if the pod’s IP address changes. Additionally, by creating the service, you also enable the frontend pods to easily find the backend service by its name through either environment variables or DNS.

![Service Discovery](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Kubernetes/Service_Discovery.png)

---

## 1.2 Why we need service discovery

To solve the problem of multi node networking, where each pods share same private network and all those pods should be accessible through a single IP address. To access these pods outside a private network, we need a Kubernetes Services.

If we consider a scenario, where in non kubernetes env where a sysadmin would configure each client app by specifying the exact IP address or hostname of the server providing the service in the client’s configuration files, doing the same in Kubernetes wouldn’t work, because:

- Pods are ephemeral—They may come and go at any time, whether it’s because a pod is removed from a node to make room for other pods, because someone scaled down the number of pods, or because a cluster node has failed.
- Kubernetes assigns an IP address to a pod after the pod has been scheduled to a node and before it’s started —Clients thus can’t know the IP address of the server pod up front.
- Horizontal scaling means multiple pods may provide the same service—Each of those pods has its own IP address. Clients shouldn’t care how many pods are backing the service and what their IPs are. They shouldn’t have to keep a list of all the individual IPs of pods.

That's why we need Kubernetes Services.

---

## 1.3  The Service Object

Real service discovery in Kubernetes starts with a Service object. A Service object is a way to create a named label selector. As we will see, the Service object does some other nice things for us, too.

Just as the kubectl run command is an easy way to create a Kubernetes deployment,
we can use kubectl expose to create a service. Let’s create some deployments and
services so we can see how they work:

```bash
## 1.0 Create a Deployment

# Boot the deployment directly by the kubectl command like ( given belowe ) or use a YAML file to setup a deployment.
# Boot up the Deployment
kubectl run cyberchef --image=mpepping/cyberchef --replicas=3 --port=8000 --labels="ver=1, env=dev"

--------------------------------------------------------------------------------------
## 2.0 Expose a Deployment

# Creating the service using the YAML

---cyberchef_service.yml
apiVersion: v1
kind: Service
metadata:
  name: cyberchef
spec:
  ports:
  - port: 80 # Host Port
    targetPort: 8000 # Container Port
  selector:
    app: cyberchef

$ kubectl create -f ./cyberchef_service.yml

--------------------------------------------------------------------------------------
## 3.0 Get the Services

# Get the clusterIP using getting the service
$ kubectl get svc -o wide
NAME         CLUSTER-IP         PORT(S)           SELECTOR
cyberchef    10.115.242.13      80/TCP            ver=1, env=dev
```

The list shows that the IP address assigned to the service is 10.115.242.13. Because this is the cluster IP, it’s only accessible from inside the cluster. The primary purpose of services is exposing groups of pods to other pods in the cluster, but you’ll usually also want to expose services externally.

### 1.3.1 Testing your service from within the cluster

You can send requests to your service from within the cluster in a few ways:

- The obvious way is to create a pod that will send the request to the service’s cluster IP and log the response. You can then examine the pod’s log to see what the service’s response was.
- You can ssh into one of the Kubernetes nodes and use the curl command.
- You can execute the curl command inside one of your existing pods through the kubectl exec command.

#### Remotely executing commands in running containers

The kubectl exec command allows you to remotely run arbitrary commands inside an existing container of a pod. This comes in handy when you want to examine the contents, state, and/or environment of a container. List the pods with the kubectl get pods command and choose one as your target for the exec command.

```bash
$ kubectl exec cyberchef-d987s -- curl -s http://10.115.242.13
<!--
    CyberChef - The Cyber Swiss Army Knife
.......
.......
-->
< Full Front Page HTML >
```

### 1.3.2 Configuring session affinity on the service

If you execute the same command a few more times, you should hit a different pod with every invocation, because the service proxy normally forwards each connection to a randomly selected backing pod, even if the connections are coming from the same client. If, on the other hand, you want all requests made by a certain client to be redirected to the same pod every time, you can set the service’s sessionAffinity property to ClientIP

```yaml
apiVersion: v1
kind: Service
spec:
  sessionAffinity: ClientIP
  ...
```

This makes the service proxy redirect all requests originating from the same client IP to the same pod. As an exercise, you can create an additional service with session affinity set to ClientIP and try sending requests to it.

### 1.3.3 Exposing multiple ports in the same service

Your service exposes only a single port, but services can also support multiple ports. For example, if your pods listened on two ports—let’s say 8080 for HTTP and 8443 for HTTPS—you could use a single service to forward both port 80 and 443 to the pod’s ports 8080 and 8443. You don’t need to create two different services in such cases. Using a single, multi-port service exposes all the service’s ports through a single cluster IP.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cyberchef
spec:
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: https
    port: 443
    targetPort: 8443
  selector:
    app: cyberchef
```

### 1.3.4 Discovering services

#### Method 1: Discovering services through env variables

When a pod is started, Kubernetes initializes a set of environment variables pointing to each service that exists at that moment. If you create the service before creating the client pods, processes in those pods can get the IP address and port of the service by inspecting their environment variables.

```bash
# To check out the env variables
$ kubectl exec cyberchef-d978l2 env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=cyberchef-d978l2
KUBERNETES_SERVICE_HOST=10.115.115.10
KUBERNETES_SERVICE_PORT=80
```

Environment variables are one way of looking up the IP and port of a service, but isn’t this usually the domain of DNS. Let look at the DNS way

#### Method 2: Discovering services through DNS

In the kubernetes env, One of the pod called as kube-dns. The kube-system namespace also includes a corresponding service with the same name. As the name suggests, the pod runs a DNS server, which all other pods running in the cluster are automatically configured to use (Kubernetes does that by modifying each container’s /etc/resolv.conf file). Any DNS query performed by a process running in a pod will be handled by Kubernetes’ own DNS server, which knows all the services running in your system.

Each service gets a DNS entry in the internal DNS server, and client pods that know the name of the service can access it through its fully qualified domain name (FQDN) instead of resorting to environment variables. Now we connect to the service through its FQDN

```bash
# Ex frontend.default.svc.cluster.local

# frontend = Service Name
# default = NameSpace
# svc.cluster.local = Configurable cluster domain suffix

$ kubectl exec it cyberchef-973l2 bash
->$ curl http://cyberchef.default.svc.cluster.local
```

### 1.3.5 Connect to services from outside the cluster

In most cases we need apps will exposed to external services through the Kubernetes services feature. Instead of having the service redirect connections to pods in the cluster, you want it to redirect to external IP(s) and port(s). This allows you to take advantage of both service load balancing and service discovery. Client pods running in the cluster can connect to the external service like they connect to internal services.

- Service Endpoints: An Endpoints resource (yes, plural) is a list of IP addresses and ports exposing a service. The Endpoints resource is like any other Kubernetes resource, so you can display its basic info with `kubectl get endpoints`:

#### Creating the service endpoints

If you create a service without a pod selector, Kubernetes won’t even create the
Endpoints resource (after all, without a selector, it can’t know which pods to include
in the service). It’s up to you to create the Endpoints resource to specify the list of endpoints for the service. To create a service with manually managed endpoints, you need to create both a Service and an Endpoints resource.

```yaml
# Creating an endpoints resource for a service without a selector
apiVersion: v1
kind: Endpoints
metadata:
  name: cyberchef-service
subsets:
  - addresses:
    - ip: 11.11.11.11 # Ip of endpoint that the service will forward connections to
    - ip: 22.22.22.22
    ports:
    - port: 80
```

The Endpoints object needs to have the same name as the service and contain the list
of target IP addresses and ports for the service. After both the Service and the Endpoints resource are posted to the server, the service is ready to be used like any regular service with a pod selector. Containers created after the service is created will include the environment variables for the service, and all connections to its IP:port pair will be load balanced between the service’s endpoints.

#### Creating an alias for an external service

Instead of exposing an external service by manually configuring the service’s End-
points, a simpler method allows you to refer to an external service by its fully qualified domain name (FQDN).

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cyberchef-service
spec:
  type: ExternalName
  externalName: someapi.somecompany.com # API Endpoint
  ports:
  - port: 80
```

After the service is created, pods can connect to the external service through the cyberchef-service.default.svc.cluster.local domain name (or even external- service) instead of using the service’s actual FQDN. This hides the actual service name and its location from pods consuming the service, allowing you to modify the service definition and point it to a different service any time later, by only changing the externalName attribute or by changing the type back to ClusterIP and creating an Endpoints object for the service—either manually or by specifying a label selector on the service and having it created automatically.

### 1.3.6 Exposing services to external clients

We need apps like web server to the outside, so external can access them.

You have a few ways to make a service accessible externally:

- Setting the service type to **NodePort**—For a NodePort service, each cluster node
opens a port on the node itself (hence the name) and redirects traffic received
on that port to the underlying service. The service isn’t accessible only at the
internal cluster IP and port, but also through a dedicated port on all nodes.

- Setting the service type to **LoadBalancer**, an extension of the NodePort type— This makes the service accessible through a dedicated load balancer, provisioned
from the cloud infrastructure Kubernetes is running on. The load balancer redirects traffic to the node port across all the nodes. Clients connect to the service
through the load balancer’s IP.

- Creating an **Ingress** resource, a radically different mechanism for exposing multiple services through a single IP address— It operates at the HTTP level (network layer 7) and can thus offer more features than layer 4 services.

#### 1.3.6.1 NodePort

Creating a NodePort service, you make Kubernetes reserve a port on all its nodes (the same port number is used across all of them) and forward incoming connections to the pods that are part of the service. This is similar to a regular service (their actual type is ClusterIP), but a NodePort service can be accessed not only through the service’s internal cluster IP, but also through any node’s IP and the reserved node port.

```bash
---cyberchef_nodeport.yml
apiVersion: v1
kind: Service
metadata:
  name: cyberchef-nodeport
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8000
    nodePort: 30123
  selector:
  app: cyberchef

--or--

# Directly using the kubectl
$ kubectl expose deployment cyberchef --type=NodePort --port=80 --targetPort=8000

# Examining a NodePort service
$ kubectl get svc cyberchef
NAME             CLUSTER-IP       EXTERNAL-IP    PORT(S)        AGE
cyberchef        10.111.254.223   <nodes>        80:30123/TCP   2m
```

Look at the EXTERNAL-IP column. It shows `nodes`, indicating the service is accessible
through the IP address of any cluster node. The PORT(S) column shows both the internal port of the cluster IP (80) and the node port (30123).

When an external client connects to a service through the node port (this also includes cases when it goes through the load balancer first), the randomly chosen pod may or may not be running on the same node that received the connection. An additional network hop is required to reach the pod, but this may not always be desirable. You can prevent this additional hop by configuring the service to redirect external traffic only to pods running on the node that received the connection. This is done by setting the externalTrafficPolicy field in the service’s spec section:

```yaml
spec:
  externalTrafficPolicy: Local
```

If a service definition includes this setting and an external connection is opened
through the service’s node port, the service proxy will choose a locally running pod. If no local pods exist, the connection will hang (it won’t be forwarded to a random global pod, the way connections are when not using the annotation). You therefore need to ensure the load balancer forwards connections only to nodes that have at least one such pod.

#### 1.3.6.2 External LoadBalancer

Finally, if you have support from the cloud that you are running on (and your cluster is configured to take advantage of it), you can use the LoadBalancer type. This builds on the NodePort type by additionally configuring the cloud to create a new load balancer and direct it at nodes in your cluster. Edit the cyberchef service again (kubectl edit service cyberchef) and change spec.type to LoadBalancer.

If you do a kubectl get services right away you’ll see that the EXTERNAL-IP column for cyberchef now says `pending`. Wait a bit and you should see a public address assigned by your cloud. You can look in the console for your cloud account and see the configuration work that Kubernetes did for you.

If Kubernetes is running in an environment that doesn’t support LoadBalancer
services, the load balancer will not be provisioned, but the service will still behave like a NodePort service. That’s because a LoadBalancer service is an extension of a NodePort service.

```bash
---cyberchef_loadbalancer.yml
apiVersion: v1
kind: Service
metadata:
  name: cyberchef
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8000
    selector:
      app: cyberchef

# Connecting through a load balancer
$ kubectl get svc cyberchef
NAME             CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
cyberchef        10.111.254.223   210.110.15.4    80:30123/TCP   2m
# Access at http://210.110.15.4
```

#### 1.3.6.3  Ingress

One important reason is that each LoadBalancer service requires its own load balancer with its own public IP address, whereas an Ingress only requires one, even when providing access to dozens of services. When a client sends an HTTP request to the Ingress, the host and path in the request determine which service the request is forwarded to Ingresses operate at the application layer of the network stack (HTTP) and can provide features such as cookie-based session affinity and the like, which services can’t.

To make ingress work, you need to install ingress as per your kubernetes enviornment.

```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: cyberchef
spec:
  rules:
  - host: cyberchef.example.com
    http:
      paths:
      - path: /
        backend:
        serviceName: cyberchef-nodeport
        servicePort: 80
```

This defines an Ingress with a single rule, which makes sure all HTTP requests received by the Ingress controller, in which the host cyberchef.example.com is requested, will be sent to the cyberchef-nodeport service on port 80.

To access your service through `http://cyberchef.example.com`, you’ll need to make sure the domain name resolves to the IP of the Ingress controller.

```bash
# List Ingress
$ kubectl get ingresses
NAME       HOSTS                  ADDRESS         PORTS   AGE
cyberchef  cyberchef.example.com  192.168.99.100  80      29m

# Note: Make sure you need to add this to your hosts file ( /etc/hosts )
192.168.99.100  cyberchef.example.com
```

- How ingress works??
The client first performed a DNS lookup of cyberchef.example.com, and the DNS server (or the local operating system) returned the IP of the Ingress controller. The client then sent an HTTP request to the Ingress controller and specified cyberchef.example.com in the Host header. From that header, the controller determined which service the client is trying to access, looked up the pod IPs through the Endpoints object associated with the service, and forwarded the client’s request to one of the pods.

- Exposing multiple services through the same ingress

```yaml
---
- host: apps.example.com
  http:
    paths:
    - path: /cyberchef
      backend:
        serviceName: cyberchef
        servicePort: 80
    - path: /linkding
      backend:
        serviceName: linkding
        servicePort: 80
```

- Exposing diff services to diff hosts

```yaml
---
spec:
  rules:
  - host: cyberchef.example.com
    http:
     paths:
     - path: /
       backend:
         serviceName: cyberchef
         servicePort: 80
  - host: linkding.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: linkding
          servicePort: 80
```

- Configuring Ingress to handle TLS traffic ( for HTTPS )

When a client opens a TLS connection to an Ingress controller, the controller terminates the TLS connection. The communication between the client and the controller is encrypted, whereas the communication between the controller and the backend pod isn’t. The application running in the pod doesn’t need to support TLS. For example, if the pod runs a web server, it can accept only HTTP traffic and let the Ingress controller take care of everything related to TLS. To enable the controller to do that, you need to attach a certificate and a private key to the Ingress. The two need to be stored in a Kubernetes resource called a Secret, which is then referenced in the Ingress manifest.

```bash
openssl genrsa --out tls.key 2048
openssl req -new -x509 tls.key -out tls.cert -days 360 -subj /CN=cyberchef.example.com

kubectl create secret tls tls-secret --cert=tls.cert --key=tls.key
```

The private key and the certificate are now stored in the Secret called tls-secret. Now, you can update your Ingress object so it will also accept HTTPS requests for cyberchef.example.com.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: cyberchef
spec:
  tls:
  - hosts:
    - cyberchef.example.com
    secretName: tls-secret
  rules:
  - host: cyberchef.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: cyberchef-nodeport
          servicePort: 80
```

```bash
# Create Ingress
$ kubectl apply -f cyberchef-ingress-tls.yaml

$ curl -k -v https://cyberchef.example.com/cyberchef
* About to connect() to cyberchef.example.com port 443 (#0)
...
* Server certificate:
*
 subject: CN=cyberchef.example.com
...
> GET /cyberchef HTTP/1.1
> ...
```

### 1.3.7 Signaling when a pod is ready to accept connections

Consider a scenario where a new pod started with proper labels is created, it becomes part of the service and requests start to be redirected to the pod. But what if the pod isn’t ready to start serving requests immediately?

The pod may need time to load either configuration or data, or it may need to perform a warm-up procedure to prevent the first user request from taking too long and affecting the user experience. In such cases you don’t want the pod to start receiving requests immediately, especially when the already-running instances can process requests properly and quickly. It makes sense to not forward requests to a pod that’s in the process of starting up until it’s fully ready.

#### Readiness probes

The readiness probe is invoked periodically and determines whether the specific pod should receive client requests or not. When a container’s readiness probe returns success, it’s signaling that the container is ready to accept requests.

- How readiness probes works
When a container is started, Kubernetes can be configured to wait for a configurable amount of time to pass before performing the first readiness check. After that, it invokes the probe periodically and acts based on the result of the readiness probe. If a pod reports that it’s not ready,

- TYPES OF READINESS PROBES
  - An Exec probe, where a process is executed. The container’s status is determined by the process’ exit status code.
  - An HTTP GET probe, which sends an HTTP GET request to the container and the HTTP status code of the response determines whether the container is ready or not.
  - A TCP Socket probe, which opens a TCP connection to a specified port of the
container. If the connection is established, the container is considered ready.

- Difference between liveness probe and rediness probe
Liveness probes keep pods healthy by killing off unhealthy containers and replacing
them with new, healthy ones, whereas readiness probes make sure that only pods that
are ready to serve requests receive them.

- Why we need readiness probe
Imagine that a group of pods (for example, pods running application servers) depends on a service provided by another pod (a backend database, for example). If at any point one of the frontend pods experiences connectivity problems and can’t reach the database anymore, it may be wise for its readiness probe to signal to Kubernetes that the pod isn’t ready to serve any requests at that time. If other pod instances aren’t experiencing the same type of connectivity issues, they can serve requests normally. A readiness probe makes sure clients only talk to those healthy pods and never notice there’s anything wrong with the system.

```yaml
apiVersion: v1
kind: ReplicationController
...
spec:
  ...
  template:
    ...
  spec:
    containers:
    - name: cyberchef
      image: mpepping/cyberchef
      readinessProbe:
        exec:
          command:
          - ls
          - /var/ready
```

The readiness probe will periodically perform the command ls /var/ready inside the container. The ls command returns exit code zero if the file exists, or a non-zero exit code otherwise. If the file exists, the readiness probe will succeed; otherwise, it will fail. Note: To check status check logs.

This mock readiness probe is useful only for demonstrating what readiness probes do.
In the real world, the readiness probe should return success or failure depending on whether the app can (and wants to) receive client requests or not. Manually removing pods from services should be performed by either deleting the pod or changing the pod’s labels instead of manually flipping a switch in the probe.

### 1.3.8 Using a headless service for discovering individual pods

If you want to connect to all of the pods. you need a IP of an individual pod, which is can't possible in default configuration in kubernetes.  One option is to have the client call the Kubernetes API server and get the list of pods and their IP addresses through an API call, but because you should always strive to keep your apps Kubernetes-agnostic, using the API server isn’t ideal.

Luckily, Kubernetes allows clients to discover pod IPs through DNS lookups. Usually, when you perform a DNS lookup for a service, the DNS server returns a single IP—the service’s cluster IP. But if you tell Kubernetes you don’t need a cluster IP for your service (you do this by setting the clusterIP field to None in the service specification ), the DNS server will return the pod IPs instead of the single service IP.

Instead of returning a single DNS A record, the DNS server will return multiple A records for the service, each pointing to the IP of an individual pod backing the ser- vice at that moment. Clients can therefore do a simple DNS A record lookup and get the IPs of all the pods that are part of the service. The client can then use that information to connect to one, many, or all of them.

```yaml
# All you need to do is to set ClusterIP to None
apiVersion: v1
kind: Service
metadata:
  name: cyberchef-headless
spec:
  clusterIP: None
  ports:
  - port: 80
    targetPort: 8000
selector:
  app: cyberchef
```

```bash
# Setup and dns lookup container ( We need nslookup and dif : tutum/dnsutils )

$ kubectl run dnsutils --image=tutum/dnsutils --generator=run-pod/v1 --command -- sleep infinity

# Perform DNSLookup for headless
$ kubectl exec dnsutils nslookup cyberchef-headless
...
Name: cyberchef-headless.default.svc.cluster.local
Address: 10.108.1.4

Name: cyberchef-headless.default.svc.cluster.local
Address: 10.108.2.5 # Pod IP

# Perform DNSLookup for non-headless
$ kubectl exec dnsutils nslookup cyberchef
...
Name: cyberchef.default.svc.cluster.local
Address: 10.111.249.153 ## ClusterIP
```

### 1.3.9 Troubleshooting Best Practices

- First, make sure you’re connecting to the service’s cluster IP from within the cluster, not from the outside.
- Don’t bother pinging the service IP to figure out if the service is accessible (remember, the service’s cluster IP is a virtual IP and pinging it will never work).
- If you’ve defined a readiness probe, make sure it’s succeeding; otherwise the pod won’t be part of the service.
- To confirm that a pod is part of the service, examine the corresponding Endpoints object with kubectl get endpoints.
- If you’re trying to access the service through its FQDN or a part of it (for example, myservice.mynamespace.svc.cluster.local or myservice.mynamespace) and it doesn’t work, see if you can access it using its cluster IP instead of the FQDN.
- Check whether you’re connecting to the port exposed by the service and not the target port.
- Try connecting to the pod IP directly to confirm your pod is accepting connec- tions on the correct port.
- If you can’t even access your app through the pod’s IP, make sure your app isn’t only binding to localhost.

---
