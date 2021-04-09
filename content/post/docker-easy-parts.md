---
title: "Docker Easy Parts [Part - 1]"
date: 2020-10-10T15:43:48+08:00
lastmod: 2020-10-10T15:43:48+08:00
draft: false
tags: ["Docker", "DevOps"]
categories: ["DevOps"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

### 1.0 What is a Docker

---

#### 1.1 Introduction

**Defination** : Docker is a tool designed to make it easier to create, deploy, and run applications by using containers. Containers allow a developer to package up an application with all of the parts it needs, such as libraries and other dependencies, and ship it all out as one package.

<!--more-->

Docker container technology was launched in 2013 as an open source Docker Engine. Containers encapsulate an application as a single executable package of software that bundles application code together with all of the related configuration files, libraries, and dependencies required for it to run.

Containerized applications are “isolated” in that they do not bundle in a copy of the operating system. Instead, an open source Docker engine is installed on the host’s operating system and becomes the conduit for containers to share an operating system with other containers on the same computing system.

![Docker](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Docker/Docker_easy_parts/docker.png)

![Life With Docker](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Docker/Docker_easy_parts/life_with_docker.png)

### 1.2 Below is a list of common Docker terms

- **Docker Engine** is a client-server application with 3 major components - a server which is a type of long-running program called a daemon process; a REST API which specifies interfaces that programs can use to talk to the daemon and instruct it what to do; a command line interface (CLI) client.

- **Docker daemon** (dockerd) listens for Docker API requests and manages Docker objects such as images, containers, networks, and volumes.

- **Docker client** (docker) is the primary way that many Docker users interact with Docker. When you use commands such as docker run, the client sends these commands to dockerd, which carries them out. The docker command uses the Docker API. The Docker client can communicate with more than one daemon.

- **Image** is a read-only template with instructions for creating a Docker container. You might create your own images or you might only use those created by others and published in a registry. To build your own image, you create a Dockerfile with a simple syntax for defining the steps needed to create the image and run it.

- **Docker registry** stores Docker images. Docker Hub is a public registry that anyone can use, and Docker is configured to look for images on Docker Hub by default.

- **Container** is a runnable instance of an image. Containers are made possible by operating system (OS) process isolation and virtualization, which enable multiple application components to share the resources of a single instance of an OS kernel.

### 1.3 Architecture of docker

Docker on your host machine is (at the time of writing) split into two parts—a daemon with a RESTful API and a client that talks to the daemon. You invoke the Docker client to get information from or give instructions to the
daemon; the daemon is a server that receives requests and returns responses from the client using the HTTP protocol. In turn, it will make requests to other services to send and receive images, also using the HTTP protocol. The server will accept requests from the command-line client or anyone else authorized to connect. The daemon is also responsible for taking care of your images and containers behind the scenes, whereasthe client acts as the intermediary between you and the RESTful API.

![Docker Architecture](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Docker/Docker_easy_parts/Docker_architecture.png)

![working_docker_run](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Docker/Docker_easy_parts/working_docker_run.png)

---

---

## 2.0 Installation

---

- **Docker** can operate on most of the Operating Systems In Industries : Windows, MacOS, Most Of flavours of Linux ( Ubuntu, RHEL, Arch)
- Docker Operates on both AMD64 and ARM Based Systems

{{< highlight bash "linenos=inline">}}
# Simple script to install docker on Linux
$ curl -sSL https://get.docker.com/ | sh

# Check Docker Verison
$ docker version --format '{{.Server.Version}}'
19.03.8

# Dump Raw JSON DATA : Like Kernel, Architecture , details , build time etc
$ docker version --format '{{json .}}'

# Running Docker without sudo
$ sudo usermod -aG docker username

or

$ sudo addgroup -a username docker
# restart docker
{{</ highlight >}}

---

---

## 3.0 Docker Basics

---

### 3.1 Day To Day Docker Commands

---

- `docker create` creates a container but does not start it.
- `docker rename` allows the container to be renamed.
- `docker run` creates and starts a container in one operation.
- `docker rm` deletes a container.
- `docker update` updates a container's resource limits.
- `docker images` shows all images.
- `docker cp` copies files or folders between a container and the local filesystem.
- `docker build` creates image from Dockerfile.
- `docker commit` creates image from a container, pausing it temporarily if it is running.
- `docker rmi` removes an image.

- `docker start` starts a container so it is running.
- `docker stop` stops a running container.
- `docker restart` stops and starts a container.
- `docker pause` pauses a running container, "freezing" it in place.
- `docker unpause` will unpause a running container.
- `docker wait` blocks until running container stops.
- `docker kill` sends a SIGKILL to a running container.
- `docker attach` will connect to a running container.

- `docker ps` shows running containers.
- `docker logs` gets logs from container. (You can use a custom log driver, but logs is only available for json-file and journald in 1.10).
- `docker inspect` looks at all the info on a container (including IP address).
- `docker events` gets events from container.
- `docker port` shows public facing port of container.
- `docker top` shows running processes in container.
- `docker stats` shows containers' resource usage statistics.
- `docker diff` shows changed files in the container's FS.
- `docker history` shows history of image.
- `docker tag` tags an image to a name (local or registry).

---

### 3.2 Getting Practical

---

#### 3.2.1 List Images

{{< highlight bash "linenos=inline">}}
docker images // show images
docker ps -a
docker ps // shows started containers -a = all containers
{{</ highlight >}}

#### 3.2.2 Start/Stop/Restart

{{< highlight bash "linenos=inline">}}
docker stop/start/restart
{{</ highlight >}}

**Note** : If you want to detach from a running container, use Ctrl + P, Ctrl + Q.
If you want to integrate a container with a host process manager, start the daemon with -r=false then use docker start -a.

#### 3.2.3 Logs

{{< highlight bash "linenos=inline">}}
docker logs {Name_of_container}
{{</ highlight >}}

#### 3.2.4 Rename

{{< highlight bash "linenos=inline">}}
docker rename new_name current_name // rename container
{{</ highlight >}}

#### 3.2.5 Create container

{{< highlight bash "linenos=inline">}}
docker create nginx // will only create a container
{{</ highlight >}}

#### 3.2.6 Example Run

{{< highlight bash "linenos=inline">}}
$ docker run --interactive --tty \
 --link web:web \
 --name web_test \
 busybox:1.29 /bin/sh

 ---
  -d = detach automatically the container (run container in background and print container ID)
 --interactive
 --tty or -t that will allocate a pseudo-TTY session
 --link = link to other container
 --name = name of docker container
 ---

$ docker exec web_test ps // show extra process run with this containers
{{</ highlight >}}

#### 3.2.7 Docker Run with SHELL Variables

{{< highlight bash "linenos=inline">}}
---
 CID=$(docker create nginx:latest)
 echo $CID // assigns to a Shell variable
 MAILER_CID=$(docker run -d dockerinaction/ch2_mailer)
 WEB_CID=$(docker create nginx)
---
$ docker start $AGENT_CID
$ docker start $WEB_CID
{{</ highlight >}}

#### 3.2.8 Env Variables

{{< highlight bash "linenos=inline">}}
$ docker run -d --name wpdb \
 -e MYSQL_ROOT_PASSWORD=ch2demo \
 mysql:5.7

$ docker run --env MY_ENVIRONMENT_VAR="this is a test" \ busybox:1.29 \ env
# --env flag, or -e for short, can be used to inject any environment variable.
{{</ highlight >}}

#### 3.2.9 With Read Only Option

{{< highlight bash "linenos=inline">}}
docker run -d --name wp --read-only wordpress:5.0.0-php7.2-apache  // create a container with only readonly options
{{</ highlight >}}

#### 3.2.10 Inspect

{{< highlight bash "linenos=inline">}}
#  The docker inspect command will display all the metadata(JSON)
$ docker inspect --format "{{.State.Running}}" wp // Prints true if container is running
$ docker inspect --format "{{.NetworkSettings.IPAddress}}" Id/name // Show ip of container
{{</ highlight >}}

#### 3.2.11 Diff ( Filesystem check )

{{< highlight bash "linenos=inline">}}
$ docker run -d --name wp_writable wordpress:5.0.0-php7.2-apache

# let’s check where Apache changed the container’s filesystem with the docker
$ docker container diff wp_writable

A - A file or directory was added
D - A file or directory was deleted
C - A file or directory was changed

---
C /run
C /run/apache2
A /run/apache2/apache2.pid
---
{{</ highlight >}}

#### 3.2.12 Clean Up

{{< highlight bash "linenos=inline">}}
docker rm -f {container_ID)
docker rm -vf $(docker ps -a -q)
docker rmi [name] // Remove an image
{{</ highlight >}}

#### 3.2.13 Executing Commands

{{< highlight bash "linenos=inline">}}
# docker exec to execute a command in container.

# To enter a running container, attach a new shell process to a running container called foo, use:
$ docker exec -it foo /bin/bash.

# exec Modes :
# 1 Basic : Runs the command in the container synchronously on the command line
$ docker exec name_of_container echo "Hello User"
Hello User

# 2 Daemon : Runs the command in the background on the container
$ docker exec -d name_C \
find / -ctime 7 -name '*log' -exec rm {} \;

# 3 Interactive : Runs the command and allows the user to interact with it
$ docker exec -it ubuntu /bin/bash
{{</ highlight >}}

#### 3.2.15 Linking containers for port isolation

**Note** : This is an older method of declaring container communication—Docker’s link flag. This isn’t the recommended way of working anymore.

{{< highlight bash "linenos=inline">}}
# Example
#  This will allow us communication between containers without using user-defined networks.
$ docker run --name wp-mysql -e MYSQL_ROOT_PASSWORD=yoursecretpassword -d mysql
$ docker run --name wordpress --link wp-mysql:mysql -p 10003:80 -d  wordpress
{{</ highlight >}}

#### 3.2.16 Search a Docker Image

{{< highlight bash "linenos=inline">}}
docker search node
docker pull node // Pull the Image by Name ( on Hub )
docker run -it node /bin/bash ( Start Node Container )
{{</ highlight >}}

#### 3.2.17 Cleanly Kill Containers

{{< highlight bash "linenos=inline">}}
# Always use docker stop ( it actually stops the containers ).
# Docker kill will send immediate signal which will kill process while running ( so they can create temp files )
---
kill           Term     15
docker kill    Kill      9
docker stop    Term     15
---
{{</ highlight >}}

#### 3.2.18 Docker Prune

{{< highlight bash "linenos=inline">}}
# Prune commands

    docker system prune
    docker volume prune
    docker network prune
    docker container prune
    docker image prune

# Example

# Nuclear Option ( if you want to remove all containers of your host machine ) [Removes all : Runnig & exited]
$ docker ps -a -q | xargs --no-run-if-empty docker rm -f

# To keep running containers
$ docker ps -a -q --filter status=exited | xargs --no-run-if-empty docker rm

# To list out all exited & failed Containers
$ comm -3 \
<(docker ps -a -q --filter=status=exited | sort) \
<(docker ps -a -q --filter=exited=0 | sort) | \
xargs --no-run-if-empty docker inspect > error_containers

# Prune Volumes
# List out all docker voluems
$ docker volume ls

# Delete Unused Volumes
$ docker volume prune
{{</ highlight >}}

#### 3.2.19 Space Occupied By docker System

{{< highlight bash "linenos=inline">}}
$ docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              7                   1                   1.963GB             1.885GB (95%)
Containers          1                   1                   0B                  0B
Local Volumes       2                   1                   242.4MB             242.3MB (99%)
Build Cache         0                   0                   0B                  0B
{{</ highlight >}}

#### 3.2.20 Container Stats

{{< highlight bash "linenos=inline">}}
$ docker stats ID
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT    MEM %               NET I/O             BLOCK I/O           PIDS
{{</ highlight >}}

#### 3.2.21 Tag

{{< highlight bash "linenos=inline">}}
docker image tag ubuntu-git:latest ubuntu-git:2.7
{{</ highlight >}}

#### 3.2.22 Commit

{{< highlight bash "linenos=inline">}}
$ docker container commit -a "@dockerinaction" -m "Added git" \
  image-dev ubuntu-git
# Outputs a new unique image identifier like:
# bbf1d5d430cdf541a72ad74dfa54f6faec41d2c1e4200778e9d4302035e5d143

# Build a New Image For Commited Image
$ docker container run -d ubuntu-git
{{</ highlight >}}

#### 3.2.23 Set an EntryPoint

{{< highlight bash "linenos=inline">}}
docker container run --name cmd-git --entrypoint git ubuntu-git
{{</ highlight >}}

---

#### 3.2.24 Versioning Best Practice

{{< highlight bash "linenos=inline">}}
# Docker official Repo's are the best example of tagging an image
# Example for go lang
 1.x
 1.9
 1.9.6
 1.9-stretch
 1.10-alpine
 latest
 # this is an example of tags to build that don't confuse the end user
{{</ highlight >}}

---

### 3.3 States of Docker

**Docker container can be in one of six states:**

- Created
- Running
- Restarting
- Paused
- Removing
- Exited (also used if the container has never been started)

---

- **Created** : A container that has been created (e.g. with docker create) but not started
- **Running** : A currently running container
- **Paused** : A container whose processes have been paused
- **Exited** : A container that ran and completed ("stopped" in other contexts, although a created container is technically also "stopped")
- **Dead** : A container that the daemon tried and failed to stop (usually due to a busy device or resource used by the container)
- **Restarting** : A container that is in the process of being restarted

#### 3.3.1 Restart State

Using the `--restart` flag at container-creation time, you can tell Docker to do any of the following:

- Never restart (default)
- Attempt to restart when a failure is detected
- Attempt for some predetermined time to restart when a failure is detected
- Always restart the container regardless of the condition

- **Methods**

  - no = = Don’t restart when the container exits
  - always == Always restart when the container exits
  - unless-stopped == Always restart, but remember explicitly stopping
  - on-failure[:max-retry] == Restart only on failure

{{< highlight bash "linenos=inline">}}
# Example Run
$ docker run -d --name backoff-detector --restart always busybox:1.29 date
$ docker logs -f backoff-detector
{{</ highlight >}}

---

### 3.4 Init & PID Systems for Docker

Several such init systems could be used inside a container. The most popular include `runit, Yelp/dumb-init, tini, supervisord, and tianon/gosu` .

{{< highlight bash "linenos=inline">}}
$ docker run -d -p 80:80 --name lamp-test tutum/lamp
$ docker top lamp-test
$ docker exec lamp-test ps
$ docker exec lamp-test kill <PID> // kill a process

$ docker run --entrypoint="cat" \
 wordpress:5.0.0-php7.2-apache \
 /usr/local/bin/docker-entrypoint.sh

# If you run through the displayed script, you’ll see how it validates the environment variables against the dependencies of the software and sets default values. Once the script has validated that WordPress can execute
{{</ highlight >}}

---

### 3.5 Software Installation Simplified

**Three main ways to install Docker images:**

- Using Docker registries
- Using image files with docker save and docker load
- Building images with Dockerfiles

**we can install software in three other ways:**

- You can use alternative repository registries or run your own registry.
- You can manually load images from a file.
- You can download a project from some other source and build an image by using a provided Dockerfile.

Note : - Keep in mind for Tags with images [latest, stable, alpha, Beta]

- Download image from another regestry instead of docker hub : `docker pull quay.io/dockerinaction/ch3_hello_registry:latest`
- `[REGISTRYHOST:PORT/][USERNAME/]NAME[:TAG]`

#### 3.5.1 Installing Images using dockerfile

{{< highlight bash "linenos=inline">}}
git clone https://github.com/dockerinaction/ch3_dockerfile.git
docker build -t dia_ch3/dockerfile:latest ch3_dockerfile
{{</ highlight >}}

---

### 3.6 Backup & Restore Docker Images

- `docker export` turns container filesystem into tarball archive stream to STDOUT.
- `docker import` creates an image from a tarball.
- `docker load` loads an image from a tar archive as STDIN, including images and tags (as of 0.7).
- `docker save` saves an image to a tar archive stream to STDOUT with all parent layers, tags & versions (as of 0.7).

#### 3.6.1 Comparison

{{< highlight bash "linenos=inline">}}
Docker export : Container To TAR
Docker Import : TAR to Image
Docker save   : Image To TAR
Docker load   : TAR to Image
{{</ highlight >}}

#### 3.6.2 Getting Practical

{{< highlight bash "linenos=inline">}}
$ docker login/logout // get access to private repo on docker hub
$ docker save [image_name]
$ docker save -o myfile.tar image_name:latest // saves tar file in current directory
$ docker load –i myfile.tar

# Load/Save image
- Load an image from file:
$ docker load < my_image.tar.gz

# Save an existing image:
$ docker save my_image:my_tag | gzip > my_image.tar.gz

# Import/Export container
# Import a container as an image from file:
$ cat my_container.tar.gz | docker import - my_image:my_tag

# Export an existing container:
$ docker export my_container | gzip > my_container.tar.gz

# Difference between loading a saved image and importing an exported container as an image
Loading an image using the load command creates a new image including its history.
Importing a container as an image using the import command creates a new image excluding the history which results in a smaller image size compared to loading an image.

# Save the State of Docker Image:
# we can save the state of image by commiting ( like we do in source control )
$ docker commit my_container
# Creates a new Image ID

# Restore State of Conatiner
$ docker run [options] New_Image_ID

# There is problem here. Docker Images ID's are 256Bit long, So there is no way to remember. what stuff we commit ( solution : is Tagging )

# Tag
$ docker tag ID_OF_IMAGE imagename
$ docker run imagename ( instead of 256Bit long ID )

# We can also Reffer to a specific image in builds
# mention ID of Specific Build of Image ( like we did in previous steps ) and use it docker file.
# Remember : This is image is locally available ( docker is not looking this on Docker HUB )

FROM 8eaa4ff06b53

##  Walkthrough of Saving States
# Install 2048 Game ( for this we need VNC viewer ( TigerVNC )
$ docker run -d -p 5901:5901 -p 6080:6080 --name win2048 imiell/win2048
$ vncviewer localhost:1 (:1 If you have no X display on host )
# connect to port 5901 & default password for vnc viewer is 'vncpass'
# Save a Game ( Commit Container )
$ docker commit win2048 1((co14-1))
$ docker tag ID 2048tag:$(date +%s)

# Return To the Save Game
$ docker run -d -p 5901:5901 -p 6080:6080 --name win2048 my2048tag:$mytag
{{</ highlight >}}

---

### 3.7 Generating Dependency graph of Docker Image

{{< highlight bash "linenos=inline">}}
# genrate a tree of dependecies of image
$ git clone https://github.com/docker-in-practice/docker-image-graph
$ cd docker-image-graph
$ docker build -t dockerinpractice/docker-image-graph
$ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock dockerinpractice/docker-image-graph > docker_images.png
{{</ highlight >}}

---

### 3.8 Tricks for Making an Image Smaller

#### 3.8.1 Method 1 : Reduce the size of Third Party Image

{{< highlight bash "linenos=inline">}}
# Step 1 : Remove Unnecessary file
# Step 2 : Flatten the Image ( describe in this book )
# Step 3 : Check Which Packages we dont need ( $ dpkg -l | awk '{print $2}'
# Step 4 : Remove Packages ( apt-get purge -y package name )
# Step 5 : Clean the cache ( apt-get autoremove, apt-get clean )
# Step 6 : Remove all the man pages and other doc files :
$ rm -rf /usr/share/doc/* /usr/share/man/* /usr/share/info/*
# Step 7 : Clean the Temp data & logs in (/var)
$ find /var | grep '\.log$' | xargs rm -v

# Step 8 : Commit The Image ( These Steps Creates a Much Smaller Image )
{{</ highlight >}}

#### 3.8.2 Method 2 : Tiny Docker Images with BusyBox and Alpine

Small, usable OSs that can be embedded onto a low-power or cheap computer have existed since Linux began.

{{< highlight bash "linenos=inline">}}
# BusyBox ( Weight of BusyBox 2.5 MB )
# BusyBox is so small ( so it can't uses the bash. It uses ash )
$ docker run -it busybox /bin/ash

# problem is that busybox don't uses any package manager : so for installing packages
$ docker run -it progrium/busybox /bin/ash (Size: 5 MB )
# its uses opkg package manager
$ opkg-install bash > /dev/null
$ bash ( Size of contianer is 6 MB with Bash Shell )
( get ready to play with bash )

# Alpine ( 36 MB )
--- Package Manager : APK
FROM gliderlabs/alpine:3.6
RUN apk-install mysql-client
ENTRYPOINT ["mysql"]
---

list of packages : https://pkgs.alpinelinux.org/packages
{{</ highlight >}}

#### 3.8.3 Method 3 : The GO model of minimal containers

{{< highlight bash "linenos=inline">}}
# we can minimal Web server with go [ 5 MB Web Server ]
https://github.com/docker-in-practice/go-web-server

--- Dockerfile
FROM golang:1.4.2
RUN CGO_ENABLED=0 go get \
-a -ldflags '-s' -installsuffix cgo \
github.com/docker-in-practice/go-web-server
CMD ["cat","/go/bin/go-web-server"]
---

$ docker build -t go-webserver .
$ mkdir -p go-web-server && cd go-web-server
$ docker run go-webserver > go-web-server
$ chmod +x go-web-server
$ echo Hi > page.html

---
FROM scratch
ADD go-web-server /go-web-server
ADD page.html /page.html
ENTRYPOINT ["/go-web-server"]

$ docker build -t go-web-server .
$ docker images | grep go-web-server
$ docker run -p 8080:8080 go-web-server -port 8080
{{</ highlight >}}

**Note** : Remember One Large image is much efficient than some small images : Because it saves space on your HDD and save network bandwidth also & Easy to maintainable.

**Example** : One Ubuntu Image with Node, Python, Nginx and other services ( around 1GB) (assigns only 1 IP)
Many small container are request internet ( so they consume bandwidth more & also they will consume more space than 1 GB )

---

---

## 4.0 Storage Volumes

---

### 4.1 Day To Day Volumes Command

- `docker volume create`
- `docker volume rm`
- `docker volume ls`
- `docker volume inspect`

---

### 4.2 Types of Volumes

The three most common types of storage mounted into containers:

- Bind mounts
- In-memory storage
- Docker volumes

#### 4.2.1 Bind Mounts

Bind mounts are mount points used to remount parts of a filesystem tree onto other locations. When working with containers, bind mounts attach a user-specified location on the host filesystem to a specific point in a container file tree.

{{< highlight bash "linenos=inline">}}
---
CONF_SRC=~/example.conf; \
CONF_DST=/etc/nginx/conf.d/default.conf;\
LOG_SRC=~/example.log; \
LOG_DST=/var/log/nginx/custom.host.access.log; \
docker run -d --name diaweb \
--mount type=bind,src=${CONF_SRC},dst=${CONF_DST} \
--mount type=bind,src=${LOG_SRC},dst=${LOG_DST} \
-p 80:80 \
nginx:latest

---
{{</ highlight >}}

#### 4.2.2 In-Memory Storage

Most service software and web applications use private key files, database passwords,
API key files, or other sensitive configuration files, and need upload buffering space.
In these cases, it is important that you never include those types of files in an image or
write them to disk. Instead, you should use in-memory storage. You can add in-memory
storage to containers with a special type of mount.

{{< highlight bash "linenos=inline">}}
# 1777 permissions in octal
# tmpfs-size=16k
---
memory-based filesystem into a containerdocker run --rm \
--mount type=tmpfs,dst=/tmp,tmpfs-size=16k,tmpfs-mode=1770 \
--entrypoint mount \
alpine:latest -v
---
{{</ highlight >}}

#### 4.2.3 Docker Volumes

Docker volumes are named filesystem trees managed by Docker. They can be implemented with disk storage on the host filesystem, or another more exotic backend such as cloud storage. All operations on Docker volumes can be accomplished using the docker volume subcommand set.

{{< highlight bash "linenos=inline">}}
---
docker volume create \
--driver local \
--label example=location \
location-example
docker volume inspect \
--format "{{json .Mountpoint}}" \
location-example
---
{{</ highlight >}}

---

### 4.3 Moving Docker to a different Partition

{{< highlight bash "linenos=inline">}}
# Stop Docker Daemon ( service docker stop )
$ $ dockerd -g /home/dockeruser/mydocker
{{</ highlight >}}

**Note** : This will wipe all the containers and images from your previous Docker daemon.

---

### 4.4 Access Filesystem from Docker Container

{{< highlight bash "linenos=inline">}}
# This will mount /dotfiles folder to container
$ docker run -v /home/hacstac/dotfiles:~/dotfiles -t debian bash
{{</ highlight >}}

---

### 4.5 Share Volumes Across the internet

{{< highlight bash "linenos=inline">}}
# In this we use a technology called Resilio
# Example ( 2 Machines ) : Setup Resilio on both Machine - That synchronized a volume ( Connected through a Secret Key )

# Machine 1
$ docker run -d -p 8888:8888 -p 55555:55555 --name resilio ctlc/btsync

# docker logs resilio Or ( Lazy ) -> Use Portainer ( Logs )
# copy secret key

$ docker run -it --volumes-from resilio ubuntu /bin/bash
# create Data in ubuntu : touch /data/shared_from_server

# Machine 2
# setup resilio client
$ docker run -d --name resilio-client -p 8888:8888 -p 55555:55555 ctlc/btsync key_of_server

# Setup ubuntu
$ docker run -it --volumes-from resilio-client ubuntu /bin/bash
# our data folder is now available in this & if you create a file in here then it will synchronized to Machine 1 also.

{{</ highlight >}}

---

### 4.6 Using a Centralized Data Volumes For Containers

{{< highlight bash "linenos=inline">}}
# Create a volume with docker which store a data which you need in other docker containers
$ docker run -v /codebase --name codebase busybox

# access the codebase
$ docker run -it --volumes-from codebase ubuntu /bin/bash
{{</ highlight >}}

---

### 4.7 Mounting the remote file systems

{{< highlight bash "linenos=inline">}}
# This will need FUSE Kernel Module to be loaded on Host OS ( filesystem and userspace )
# Required Root Access ( Danger )

# 4.7.1. SSHFS
# Local Host
$ docker run -it --privileged debian /bin/bash
# Inside a Container
$ sudo apt-get update && apt-get install sshfs
$ LOCALPATH=/path/to/directory/
$ mkdir $LOCALPATH
$ sshfs -o nonempty user@host:/path/to/directory $LOCALPATH

# to unmount fusermount -u /path/to/local/directory
# Now the remote folder is mount on LOCALPATH

# 4.7.2 NFS
# Install a NFS on host ( because docker doesn't support NFS )
$ apt-get install nfs-kernel-server
$ mkdir /export
$ chmod 777 /export
$ mount --bind /opt/test/db /export

# add this to fstab ( if you want to persist over reboot )
/etc/fstab file: /opt/test/db /export none bind 0 0

# add this to /etc/exports
/export   127.0.0.1(ro,fsid=0,insecure,no_subtree_check,async)
# to Read/Write : change ro to rw & add no_root_squash (After async)
# To open to the internet replace localhost to *
( Danger : Think about it )

$ mount -t nfs 127.0.0.1:/export /mnt
$ exportfs -a
$ service nfs-kernel-server restart

# Run a container
$ docker run -it --name nfs-client --privileged -v /mnt:/mnt busybox /bin/true

# Mount on other container
$ docker run -it --volumes-from nfs-client debian /bin/bash
{{</ highlight >}}

---

---

## 5.0 Networks in Docker

---

### 5.1 Day To Day Network Commands

- `docker network create`
- `docker network rm`
- `docker network ls`
- `docker network inspect`
- `docker network connect`
- `docker network disconnect`

---

### 5.2 Examples

#### 5.2.1 To list all networks

{{< highlight bash "linenos=inline">}}
$ docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
f32f6d51e8c8        bridge              bridge              local
366d3d1f4719        hacstac_default     bridge              local
6c08bddba2c4        host                host                local
b726554d155b        none                null                local
{{</ highlight >}}

#### 5.2.2 To Create a New Network

{{< highlight bash "linenos=inline">}}
$ docker network create \
  --driver bridge \
  --label project=dockerinaction \
  --label chapter=5 \
  --attachable \
  --scope local \
  --subnet 10.0.42.0/24 \
  --ip-range 10.0.42.128/25 \
  user-network

$ docker run -it \
  --network user-network \
  --name network-explorer \
  alpine:3.8 \
    sh

# CTRL-P + CTRL-Q Detech
# docker attach network-explorer

# walkthrough
$ docker network create my_network [ Create a Network ]
$ docker network connect my_network blog1 [ blog1 container connect to a network my_network ]
$ docker run -it --network my_network ubuntu:16.04 bash [ Now this ubuntu container have access to blog1 Container ]

$ ip -f inet -4 -o addr // this will list loopback and assign ip subnet address
{{</ highlight >}}

#### 5.2.3 Create a another bridge network

{{< highlight bash "linenos=inline">}}
$ docker network create \
  --driver bridge \
  --attachable \
  --scope local \
  --subnet 10.0.43.0/24 \
  --ip-range 10.0.43.128/25 \
  user-network2


# Attach priviously created container attach to this network
$ docker network connect \
  user-network2 \
  network-explorer

# then this container lists two ethernet addresses

# scan with nmap
$ nmap -sn 10.0.42.* -sn 10.0.43.* -oG /dev/stdout | grep Status
{{</ highlight >}}

#### 5.2.4 With Network - none : Means container with no external excess

{{< highlight bash "linenos=inline">}}
$ docker run --rm \
    --network none \
    alpine:3.8 \
    ping -w 2 1.1.1.1
# it will try to ping 1.1.1.1 but it failed because this container hace no external network access

# NodePort Publishing
# 8080:8000 will denote 8080 port of host machine and 8000 port of container
$ docker run -d -p 8080 --name listener alpine:3.8
$ docker port listener // To view port of running container
{{</ highlight >}}

#### 5.2.5 DNS with docker

{{< highlight bash "linenos=inline">}}
# Feature 1 : --hostname will add hostname : so we open with DN
$ docker run --rm \
    --hostname barker \
    alpine:3.8 \
    nslookup barker

---
Server:    10.0.2.3
Address 1: 10.0.2.3

Name:      barker
Address 1: 172.17.0.22 barker
----

# Feature 2 : --dns : set dns server on container
$ docker run --rm \
    --dns 8.8.8.8 \
    alpine:3.8 \
    nslookup docker.com

# Feature 3 : --dns-search : allows us to specify a DNS searchdomain,  which  is  like  a  default  hostname  suffix
$ docker run --rm \
    --dns-search docker.com \
    --dns 1.1.1.1 \
    alpine:3.8 cat /etc/resolv.conf
# Will display contents that look like:
# search docker.com
# nameserver 1.1.1.1

# Feature 4 : --add-host
$ docker run --rm \
    --hostname mycontainer \
    --add-host docker.com:127.0.0.1 \
    --add-host test:10.10.10.2 \
    alpine:3.8 \
    cat /etc/hosts

---
172.17.0.45  mycontainer
127.0.0.1    localhost
::1          localhost ip6-localhost ip6-loopbackfe00
::0          ip6-localnetff00::0      ip6-mcastprefixff02
::1          ip6-allnodesff02::2      ip6-allrouters
10.10.10.2   test
127.0.0.1   docker.com
---
{{</ highlight >}}

#### 5.2.6 You can specify a specific IP address for a container

{{< highlight bash "linenos=inline">}}
# create a new bridge network with your subnet and gateway for your ip block
$ docker network create --subnet 203.0.113.0/24 --gateway 203.0.113.254 iptastic

# run a nginx container with a specific ip in that block
$ docker run --rm -it --net iptastic --ip 203.0.113.2 nginx

# curl the ip from any other place (assuming this is a public ip block duh)
$ curl 203.0.113.2
{{</ highlight >}}

#### 5.2.7 Open a Docker Daemon to the World

{{< highlight bash "linenos=inline">}}
# first of stop the docker service
$ sudo service docker stop / or
$ sudo systemctl stop docker

# Checks for docker daemon ??
$  ps -ef | grep -E 'docker(d| -d| daemon)\b' | grep -v grep

# Expose to local host : 2375
$ sudo docker daemon -H tcp://0.0.0.0:2375

# To connect
$ docker -H tcp://<your host's ip>:2375 <subcommand>
{{</ highlight >}}

#### 5.2.8 Get an IP & Ports of a Docker Container

{{< highlight bash "linenos=inline">}}
$ alias dl='docker ps -l -q'  ( latest container ID )
$ docker inspect $(dl) | grep -wm1 IPAddress | cut -d '"' -f 4
Pass ( ID of container Instead of dl ) : this above command gives ip of latest container

# get ports
$ docker inspect -f '{{range $p, $conf := .NetworkSettings.Ports}} {{$p}} -> {{(index $conf 0).HostPort}} {{end}}' 274d2292a137 | name_of_container
{{</ highlight >}}

---

---

## 6.0 Limiting Risk with Resource Controls

---

### 6.1 Memory Limits

{{< highlight bash "linenos=inline">}}

$ docker container run -d --name ch6_mariadb \
    --memory 256m \
    --cpu-shares 1024 \
    --cap-drop net_raw \
    -e MYSQL_ROOT_PASSWORD=test \
    mariadb:5.5

# This container only uses the 256M memory
{{</ highlight >}}

---

### 6.2 CPU Limits

{{< highlight bash "linenos=inline">}}
$ docker container run -d -P --name ch6_wordpress \
--memory 512m \
--cpu-shares 512 \
--cap-drop net_raw \
--link ch6_mariadb:mysql \
-e WORDPRESS_DB_PASSWORD=test \
wordpress:5.0.0-php7.2-apache
# if total cpu share is 1536 - 512 ( 33% ) : this container consume 33% of cpu shares

$ docker container run -d -P --name ch6_wordpress \
--memory 512m \
--cpus 0.75 \
--cap-drop net_raw \
--link ch6_mariadb:mysql \
-e WORDPRESS_DB_PASSWORD=test \
wordpress:5.0.0-php7.2-apache
# This Container : consumed max 75% of cpu cores
# also we can use {--cpuset-cpus 0-4} (cores)
{{</ highlight >}}

---

### 6.3 Access to devices

{{< highlight bash "linenos=inline">}}
$ docker container run -it --rm \
    --device /dev/video0:/dev/video0 \
    ubuntu:16.04 ls -al /dev
# --device flag will mount external device to container
{{</ highlight >}}

---

### 6.4 Sharing Memory ( IPC : Interprocess Communication )

{{< highlight bash "linenos=inline">}}
# Producer
$ docker container run -d -u nobody --name ch6_ipc_producer \
    --ipc shareable \
    dockerinaction/ch6_ipc -producer

# Consumer
$ docker container run -d --name ch6_ipc_consumer \
    --ipc container:ch6_ipc_producer \
    dockerinaction/ch6_ipc -consumer

# we can see the process of one container in another : by using docker logs { they share the memory space }

# IMP NOTE : In docker to clean Volumes
$ docker rm -vf name_of_container
{{</ highlight >}}

---

### 6.5 Understanding Users

{{< highlight bash "linenos=inline">}}
# if we want to setup a container with diff user then
$ docker container run --rm \
    --user nobody \
    busybox:1.29 id

$ docker container run --rm \
    -u 1000:1000 \
    busybox:1.29 \
    /bin/bash -c "echo This is important info > /logFiles/important.log"
# with this userID:GroupID we can access the file system of this users
{{</ highlight >}}

---

### 6.6 OS features Access with Capabilities

Linux capabilities can be set by using cap-add and cap-drop. See <https://docs.docker.com/engine/reference/run/#/runtime-privilege-and-linux-capabilities> for details. This should be used for greater security.

{{< highlight bash "linenos=inline">}}
---
SYS_MODULE —Insert/remove kernel modules
SYS_RAWIO — Modify kernel memory
SYS_NICE — Modify priority of processes
SYS_RESOURCE — Override resource limits
SYS_TIME — Modify the system clock
AUDIT_CONTROL — Configure audit subsystem
MAC_ADMIN — Configure MAC configuration
SYSLOG — Modify kernel print behavior
NET_ADMIN — Configure the network
SYS_ADMIN — Catchall for administrative functions
---

$ docker container run --rm -u nobody \
    --cap-add sys_admin \
    ubuntu:16.04 \
    /bin/bash -c "capsh --print | grep sys_admin"

# this --cap-add sys_admin will add admin facilities to container
# we can inspect docker with .HostConfig.CapAdd && .HostConfig.CapDrop show capabilities

# Give access to a single device:

$ docker run -it --device=/dev/ttyUSB0 debian bash

# Give access to all devices:

$ docker run -it --privileged -v /dev/bus/usb:/dev/bus/usb debian bash

# Docker Container with full privileges
$ docker container run --rm \
    --privileged \
    ubuntu:16.04 capsh ls /dev
# check out list of mounted devices
{{</ highlight >}}

---

### 6.7 Additional Security with Docker

{{< highlight bash "linenos=inline">}}
$ docker container run --rm -it \
    --security-opt seccomp=path_to_the_secomp conf file \
    ubuntu:16.04 sh

# For Linux Security Modules ( LSM )
---
The LSM security option values are specified in one of seven formats:

- To prevent a container from gaining new privileges after it starts, use
'no-new-privileges'

- To  set  a  SELinux  user  label,  use  the  form
label=user:username, where is the name of the user you want to use for the label.

- To set a SELinux role label, use the form  label=role:role where is the name of the role you want to apply to processes in the container.

- To set a SELinux type label, use the form label=type:type , where is the type name of the processes in the container.
- To set a SELinux-level label, use the form 'label:level:label' , where is the level at which processes in the container should run. Levels are specified as  low-high  pairs.  Where  abbreviated  to  the  low  level  only,  SELinux  will  inter-pret the range as single level.

- To disable SELinux label confinement for a container, use the form
label=disable

# NOTE : Avoid Running Containers in privileged mode whenever possible
{{</ highlight >}}

---

### 6.8 Using Socat to monitor docker api traffic

In this technique you’ll insert a proxy Unix domain socket between your request and the server’s socket to see what passes through it. Note that you’ll need root or sudo privileges to make this work.

{{< highlight bash "linenos=inline">}}
# We need a socat ( Install socat as per the OS package Maneger )

$ socat -v UNIX-LISTEN:/tmp/dockerapi.sock,fork \UNIX-CONNECT:/var/run/docker.sock &

In this command, -v makes the output readable, with indications of the flow of data.The UNIX-LISTEN part tells socat to listen on a Unix socket, fork ensures that socatdoesn’t  exit  after  the  first  request,  and  UNIX-CONNECT  tells  socat  to  connect  toDocker’s  Unix  socket.  The  &  specifies  that  the  command  runs  in  the  background.If you usually run the Docker client with sudo, you’ll need to do the same thing here as well.

# List all containers
$ docker -H unix:///tmp/dockerapi.sock ps -a
# This will show how client request to daemon
{{</ highlight >}}

---

### 6.9 Setting TimeZone in Containers

{{< highlight bash "linenos=inline">}}
# Runs a command to display the time zone on the host
$ date +%Z // UTC

# change TimeZone
---
FROM centos:7
RUN rm -rf /etc/localtime
RUN ln -s /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
CMD date +%Z
---

# Build Image
$ docker build -t timezone_change .
$ docker run timezone_change
Asia/Kolkata
{{</ highlight >}}

---

### 6.10 Locale Management

locale will be set in the environment through the LANG,LANGUAGE, and locale-gen variables

{{< highlight bash "linenos=inline">}}
# you are getting encoding error if correct locale is not set.

# Check for locale
$ env | grep LANG
LANG=en_GB.UTF-8
This is British English, with text encoded in UTF-8.

# Set Locale
---
FROM ubuntu:16.04
RUN apt-get update && apt-get install -y locales
RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
CMD env
---

$ docker build -t encoding .
{{</ highlight >}}

---

---
