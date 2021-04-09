---
title: "Docker Hard Parts [Part - 2]"
date: 2020-10-15T15:43:48+08:00
tags: ["Docker", "DevOps"]
categories: ["DevOps"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

### 7.0 Building Images automatically with DockerFiles

---

### 7.1 Instructions

* `.dockerignore`
* `FROM` Sets the Base Image for subsequent instructions.

<!--more-->

* `MAINTAINER` (deprecated - use LABEL instead) Set the Author field of the generated images.
* `RUN` execute any commands in a new layer on top of the current image and commit the results.
* `CMD` provide defaults for an executing container.
* `EXPOSE` informs Docker that the container listens on the specified network ports at runtime. NOTE: does not actually make ports accessible.
* `ENV` sets environment variable.
* `ADD` copies new files, directories or remote file to container. Invalidates caches. Avoid ADD and use COPY instead.
* `COPY` copies new files or directories to container. By default this copies as root regardless of the USER/WORKDIR settings. Use `--chown=<user>:<group>` to give ownership to another user/group. (Same for ADD.)
* `ENTRYPOINT` configures a container that will run as an executable.
* `VOLUME` creates a mount point for externally mounted volumes or other containers.*   USER sets the user name for following RUN / CMD / ENTRYPOINT commands.
* `WORKDIR` sets the working directory.
* `ARG` defines a build-time variable.
* `ONBUILD` adds a trigger instruction when the image is used as the base for another build.
* `STOPSIGNAL` sets the system call signal that will be sent to the container to exit.
* `LABEL` apply key/value metadata to your images, containers, or daemons.

**NOTE** : If your running dangerous commands ( add a logic that it fails if your shell is outside docker )

```bash
#  Shell script fails if it’s run outside a container
#!/bin/bash
if ! [ -f /.dockerenv ]
then
    echo 'Not in a Docker container, exiting.'
    exit 1
fi
```

---

### 7.2 Getting Practical

#### 7.2.1 Packaging Git with Dockerfile

```bash
# Create a file name Dockerfile
---
FROM ubuntu:latest
LABEL maintainer="dia@allingeek.com"
RUN apt-get update && apt-get install -y git
ENTRYPOINT ["git"]
---

# Instantly Create Dockerfile Images
$ docker build -t htop - << EOF
FROM alpine
RUN apk --no-cache add htop
EOF

$ docker image build --tage ubuntu-git:auto  .

---
FROM ubuntu:latest — Tells Docker to start from the latest Ubuntu image just as
you did when creating the image manually.

LABEL maintainer — Sets the maintainer name and email for the image. Provid-
ing this information helps people know whom to contact if there’s a problem
with the image. This was accomplished earlier when you invoked
commit.

RUN apt-get update && apt-get install -y git — Tells the builder to run the
provided commands to install Git.

ENTRYPOINT ["git"] — Sets the entrypoint for the image to  git.
---

# --file or -f will read from diff file like 'BuildScript or anything'
# --quiet or -q will run in quiet mode
```

---

#### 7.2.2 Dockerignore

- `.dockerignore` file will help us to exclude some files to add to the image during the build
- CLI modifies the context to exclude files and directories that match patterns in it. This helps to avoid unnecessarily sending large or sensitive files and directories to the daemon and potentially adding them to images using ADD or COPY.

---

#### 7.2.3 File System Instructions

```bash
- COPY : Will copy files from the filesystem where the image is being built.
- VOLUME : Same as --volume flag ( bound mount volume )
- CMD : This is a closely related to ENTRYPOINT
- ADD : This operates similarly to the COPY instruction with two imp differences:
           - fetch remote sources files if a Url is specified
           - Extract the files of any source determined to be an archive file
- ONBUILD : This instruction let the other instructions to execute ( if resulting image is used as base img for another build )


# Example
# ADD : We can ADD large no of files to a container without any problem
# Docker will unpack tarfiles of most standard types (.gz, .bz2, .xz, .tar).
# some.tar
---
FROM Debian
RUN mkdir -p /opt/libeatmydata
ADD some.tar.gz /opt/libeatmydata/
RUN ls -lRt /opt/libeatmydata
---

# ONBUILD : Use the ONBUILD command to automate and encapsulate the building of an image.
# GO Example  ( Outyet : Simple Go App )
---
$ git clone https://github.com/golang/example
$ cd example/outyet
$ docker build -t outyet .
$ docker run --publish 8080:8080 --name outyet1 -d outyet
---

With ONBUILD
---
FROM golang:onbuild
EXPOSE 8080
---

golang:onbuild Dockerfile
---
FROM golang:1.7
RUN mkdir -p /go/src/app
WORKDIR /go/src/app
CMD ["go-wrapper", "run"]
ONBUILD COPY . /go/src/app
ONBUILD RUN go-wrapper download
ONBUILD RUN go-wrapper install
---
The result of this technique is that you have an easy way to build an image that only contains  the  code  required  to  run  it, and no more.
There are also other examples of ONBUILD exists : node:onbuild , python:onbuild


# ENTRYPOINT : Sets the entrypoint for the image
# Basic Shell Script for clean logs
---
#!/bin/bash
echo "Cleaning logs over $1 days old"
find /log_dir -ctime "$1" -name '*log' -exec rm {} \;
---

# Create a DockerFile ( Create a container with clean_log script )
---
FROM ubuntu:17.04
ADD clean_log /usr/bin/clean_log
RUN chmod +x /usr/bin/clean_log
ENTRYPOINT ["/usr/bin/clean_log"]
CMD ["7"]
---

$ docker build -t log-cleaner .
$ docker run -v /var/log/myapplogs:/log_dir log-cleaner 365
# Clean The logs of over a year ( default 7 days if no arg given )
```

---

#### 7.2.4 Create a Maintainable Dockerfiles

```bash
- ARG : arg defines a variable thta users can provide to docker when building and a image.
Ex: Dockerfile
ARG VERSION=unknown
ENV VERSION="${VERSION}"
LABEL base.version="${VERSION}"

$ version=0.6; docker image build -t dockerinaction/mailer-base:${version} \
-f mailer-base.df \
--build-arg VERSION=${version} \
.

$ docker image inspect --format '{{ json .Config.Labels }}' \
dockerinaction/mailer-base:0.6

{  "base.name": "Mailer Archetype",
   "base.version": "0.6",
   "maintainer": "dia@allingeek.com"
}
```

---

#### 7.2.5 Init System for Docker

- Most Popular init systems are : runit, tini, BusyBox init, Supervisord, and DAEMON
- By Default docker comes with tini init system

```bash
$ docker container run -it --init alpine:3.6 nc -l -p 3000
# Docker ran /dev/init -- nc -l -p 3000 inside the container instead of just nc
```

---

#### 7.2.6 Health Check In Docker

```bash
- There are two ways to specify the health check command:
1. Use a HEALTHCHECK instruction when defining the image
2. On the command-line when running a container

1. This is a 1st Mothed : It is used when we define an Image
$ FROM nginx:1.13-alpine
HEALTHCHECK --interval=5s --retries=2 \
 CMD nc -vz -w 2 localhost 80 || exit 1

$ docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}'
NAMES            IMAGE                       STATUS
healthcheck_ex   dockerinaction/healthcheck  Up 3 minutes (healthy)


#  Exit Status Codes
- 0: success—The container is healthy and ready for use.
- 1: unhealthy—The container is not working correctly.
- 2: reserved—Do not use this exit code.

2. Command-line Method
$ docker container run --name=healthcheck_ex -d \
--health-cmd='nc -vz -w 2 localhost 80 || exit 1' \
nginx:1.13-alpine
```

---

#### 7.2.7 Hardening Application Images

```bash
- There are Three Methods to hardended the images
1. We can enforce that our images are built from a specific image.
2. we can make sure that regardless of how containers are built from our image, they will have a sensible default user.
3. we should eliminate a common path for root user escalation
from programs with setuid or setgid attributes set.


1. Content Addressable Images identifiers ( CAIID )
# Build Images using authentic digest : that digest is known as CAIID
# So now how many images we build from these they are authentic.
---
docker pull debian:stable
stable: Pulling from library/debian
31c6765cabf1: Pull complete
Digest: sha256:6aedee3ef827...
# Dockerfile:
FROM debian@sha256:6aedee3ef827...
...
---

2. Create a User & groups
$ RUN groupadd -r postgres && useradd -r -g postgres postgres

3. SUID & GUID
---
$ FROM ubuntu:latest
# Set the SUID bit on whoami
RUN chmod u+s /usr/bin/whoami
# Create an example user and set it as the default
RUN adduser --system --no-create-home --disabled-password --disabled-login \
 --shell /bin/sh example
USER example
# Set the default to compare the container user and
# the effective user for whoami
CMD printf "Container running as: %s\n" $(id -u -n) && \
 printf "Effectively running whoami as: %s\n" $(whoami)
---
Output
---
Container running as: example
Effectively running whoami as: root
---

The output of the default command shows that even though you’ve executed the
whoami command as the example user, it’s running from the context of the root user.
```

---

#### 7.2.8 Complete Story of Cache

```bash
# --no-cache will downlaod fresh containers from source. it will not use the cache files

# Example
$ docker build --no-cache .

# Busting the Cache
# we need this because some time our image build takes so much time. so we need cache up to a certain point.
# Method 1 (cheat) :
Add a benign comment after the command to invalidate the cache. This works because Docker treats the non-whitespace change to theline as though it were a new command, so the cached layer is not re-used.
---
CMD ["npm","start"] #bust the cache
---

# Method 2 ( using ARG )
Use the ARG directive in your Dockerfile to enable surgical cache-busting.
If this ARG variable isset to a value never used before on your host, the cache will be busted from that point.
---
WORKDIR todo
ARG CACHEBUST=no
RUN npm install
---

$ docker build --build-arg CACHEBUST=${RANDOM} .
$ echo ${RANDOM} 19856
$ echo ${RANDOM} 26429

# If not using Bash
$ docker build --build-arg CACHEBUST=$(date +%s) .

# Method 3 ( Using ADD )
There are two useful features of ADD that you can use to your advantage in this context: it caches the contents of the file it refers to, and it can take a network resource as an argument.

# Git Repo Example
It means if repo is not changed it uses cache or if git repo is changed it rebuild the image from scratch.
But it will vary from resources type to resource type.

we can take help of Github API here : It  has  URLs  foreach repository that return JSON for the most recent commits. When a new commit ismade, the content of the response changes.

---
FROM ubuntu:16.04
ADD https://api.github.com/repos/nodejs/node/commits  /dev/null
RUN git clone https://github.com/nodejs/node
---
```

---

#### 7.2.9 Flattening Images

```bash
# This is imp because images can reveal the imp information
Example :

# create a docker file : It has sensitive information
---
FROM debian
RUN echo "My Big Secret" >> /tmp/secret_key
RUN cat /tmp/secret_key
RUN rm /tmp/secret_key
---
$ docker build -t secret .
# But now problem arise
$ docker history secret
...
5e39caf7560f 3 days ago /bin/sh -c echo "My Big Secret" >> /tmp/se 14 B
...

$ docker run 5b376ff3d7cd cat /tmp/secret_key
My Big Secret

But if someone could download this image from public repo and insect the history & run thi command : It reveal the secret.

# To get rid of this type of problem : we need to remove intermediate layering
# we need to export the image as a trivially run container
and then re-import and tag the resulting image:
$ docker run -d secret /bin/true
- new_id

# Runs a docker export, taking a conatiner iD as arg and outgoing a tar file of fs contents.
# This is piped to docker import which takes tar file and create a new image.
$ docker export new_id | docker import - new_secret
$ docker history new_secret
```

---

### 7.3 Refer

* [Examples](https://docs.docker.com/engine/reference/builder/#dockerfile-examples)
* [Best practices for writing Dockerfiles](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)
* [Michael Crosby](http://crosbymichael.com/) has some more [Dockerfiles best practices](http://crosbymichael.com/dockerfile-best-practices.html) / [take 2](http://crosbymichael.com/dockerfile-best-practices-take-2.html).
* [Building Good Docker Images](http://jonathan.bergknoff.com/journal/building-good-docker-images) / [Building Better Docker Images](http://jonathan.bergknoff.com/journal/building-better-docker-images)
* [Managing Container Configuration with Metadata](https://speakerdeck.com/garethr/managing-container-configuration-with-metadata)
* [How to write excellent Dockerfiles](https://rock-it.pl/how-to-write-excellent-dockerfiles/)

---
---

## 8.0 Registry & Repository

![Registry.png](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Docker/Docker_Hard_Parts/Repository.png)

A repository is a *hosted* collection of tagged images that together create the file system for a container.

A registry is a *host* -- a server that stores repositories and provides an HTTP API for [managing the uploading and downloading of repositories](https://docs.docker.com/engine/tutorials/dockerrepos/).

### 8.1 Enviornment

* [`docker login`](https://docs.docker.com/engine/reference/commandline/login) to login to a registry.
* [`docker logout`](https://docs.docker.com/engine/reference/commandline/logout) to logout from a registry.
* [`docker search`](https://docs.docker.com/engine/reference/commandline/search) searches registry for image.
* [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull) pulls an image from registry to local machine.
* [`docker push`](https://docs.docker.com/engine/reference/commandline/push) pushes an image to the registry from local machine.

**NOTE** : Refer Chapter 9 of Docker in Action : Public and private
software distribution ( This Cover's Every Basic detail about Registry & Repository )

### 8.2 Setup Local Docker Registry

```bash
# To start a registry on local Network
$ docker run -d -p 5000:5000 -v $HOME/registry:/var/lib/registry registry:2

# This  command  makes  the  registry  available  on  port  5000  of  the  Docker  host(-p  5000:5000).  With  the  -v  flag,  it  makes  the  registry folder on your host(/var/lib/registry)  available  in  the  container  as  $HOME/registry.  The  registry’s  fileswill therefore be stored on the host in the /var/lib/registry folder.
# HOSTNAME is the hostname or IP address of your new reg-istry server
# we also use --insecure-registry ( We know our local network is secure :) [Docker will only allow you to pull from registries with a signedHTTPS certificate.] )

# Push the image to the registry
$ docker push HOSTNAME:5000/image:tag.
```

---
---

## 9.0 Docker Compose

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. To learn more about all the features of Compose, see the [list of features](https://docs.docker.com/compose/overview/#features).

The standard filename for Compose files is `docker-compose.yml`.

### 9.1 YAML Basics

* A YAML document can include a comment at the end of any line. Com￾ments are marked by a space followed by a hash sign ( #). Any characters that follow until the end of the line are ignored by the parser.
* YAML uses three types of data and two styles of describing that data, block and flow.
  * Flow collections are specified similarly to collection literals in JavaScript and other
languages. For example, the following is a list of strings in the flow style: `["PersonA","PersonB"]`
  * The block style is more common and will be used in this primer except where noted.
The three types of data are `maps`, `lists`, and `scalar values`.
    * Maps are defined by a set of unique properties in the form of key/value pairs that
are delimited by a colon and space (: ).
    * Scaler Values : Scaler String `image: "alpine"`
        , Scaler Command : `command: echo hello world`
    * Scaler Rules :
      1. Must not be empty,
      2. Must not contain leading or trailing whitespace characters
      3. Must not begin with an indicator character (for example, - or :) in places where
doing so would cause an ambiguity.
      4. Must never contain character combinations using a colon (:) and hash sign (#)
    * Lists (or block sequences) are series of nodes in which each element is denoted by a
leading hyphen (-) indicator. For example:
            `- item 1`
            `- item 2`
            `- item 3`
            `- # an empty item`
            `- item 4`

* Indentation Rules : YAML uses indentation to indicate content scope. Scope determines which
block each element belongs to. There are a few rules:
  * Only spaces can be used for indentation.
  * The amount of indentation does not matter as long as
      – All peer elements (in the same scope) have the same amount of indentation.
      – Any child elements are further indented.

These documents are equivalent:

```text
top-level:
 second-level: # three spaces
 third-level: # two more spaces
 - "list item" # single additional indent on items in this list
 another-third-level: # a third-level peer with the same two spaces
 fourth-level: "string scalar" # 6 more spaces
 another-second-level: # a 2nd level peer with three spaces
 - a list item # list items in this scope have
 # 15 total leading spaces
 - a peer item # A peer list item with a gap in the list

---
# every scope level adds exactly 1 space
top-level:
 second-level:
 third-level:
 - "list item"
 another-third-level:
 fourth-level: "string scalar"
 another-second-level:
 - a list item
 - a peer item
```

### 9.2 Docker Compose Basics

By using the following command you can start up your application:

```bash
# first install docker-compose on your system (eg: Ubuntu )
$ sudo apt install docker-compose
$ docker-compose -f <docker-compose-file> up
```

You can also run docker-compose in detached mode using -d flag, then you can stop it whenever needed by the following command:

```bash
docker-compose stop
```

You can bring everything down, removing the containers entirely, with the down command. Pass `--volumes` to also remove the data volume.

Let understand Docker compose with an Example :

#### 9.2.1 Example yml

```bash
# wikijs.yml
---
version: '2'
services:
  db:
    image: postgres:11-alpine
    environment:
      POSTGRES_DB: wiki
      POSTGRES_PASSWORD: wikijsrocks
      POSTGRES_USER: wikijs
    logging:
      driver: "none"
    restart: unless-stopped
    volumes:
      - db-data:/var/lib/postgresql/data

  wiki:
    image: requarks/wiki:2
    depends_on:
      - db
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: wikijsrocks
      DB_NAME: wiki
    restart: unless-stopped
    ports:
      - "80:3000"

volumes:
  db-data:
---

# we can create a docker stack with this yml file ( it established a wikijs and postgresql db )
$ docker stack deploy -c wikijs.yml wikijs

# if we use docker swarn or Kubernetes we can create a replicas of this images into 3 diff servers
--
deploy:
 replicas: 3
--
# add this to end of yml and again deploy it. It will update the container

# To Check The State of stack
$ docker stack ps \
 --format '{{.Name}}\t{{.CurrentState}}' \
 wikijs

# To remove a service from stack ( like limit the replicas from 3 to 2 )
$ docker stack deploy \
-c wikijs.yml \
--prune \
wikijs


We Need To use Prue Here : Because without Prune it will not completely remove services which causes problmes ( like for Instead of using postgressql we want to use a mysql so if we updated our stack yml file and deploy it. it will add mysql db but doesnt remove postgres containers so to remove postgres we use prune )

The --prune flag will clean up any resource in the stack that isn’t explicitly referenced
in the Compose file used for the deploy operation.

# Prune

The new [Data Management Commands](https://github.com/docker/docker/pull/26108) have landed as of Docker 1.13:

* `docker system prune`
* `docker volume prune`
* `docker network prune`
* `docker container prune`
* `docker image prune`


Note : There is a problem here everytime a container replaced docker will create a new volume space for container and its replicas. This would cause problems in a real-world system.
So, to get rid of this

EX:
---
volumes:
   pgdata: # empty definition uses volume defaults
services:
   postgres:
      image: dockerinaction/postgres:11-alpine
      volumes:
         - type: volume
         source: pgdata # The named volume above
         target: /var/lib/postgresql/data
      environment:
         POSTGRES_PASSWORD: example
---

The file defines a volume named pgdata, and the postgres service
mounts that volume at /var/lib/postgresql/data. That location is where the Postgre￾SQL software will store any database schema or data.

Inspect
$ docker stack deploy \
-c databases.yml \
--prune \
my-databases

$ docker volume ls
DRIVER  VOLUME NAME
local   my-databases_pgdata

$ docker service remove my-databases_postgres

Then restore the service by using the Compose file:
$ docker stack deploy \
-c databases.yml \
--prune \
my-databases
```

---
---

## 10.0 DevOps Operations With Docker

---

### 10.1 Convert Your Virtual Box VM To Docker Container

```bash
# Process : VM FILE (.vdi or anything) => TAR => Import TAR as an Image in Docker.

$ sudo apt install qemu-utils

# Identify the path to your VM disk image. ( Stop the VM )
# Sets up a variable pointingto your VM disk image.
$ VMDISK="$HOME/VirtualBox VMs/myvm/myvm.vdi"

# Initializes a kernelmodule requiredby qemu-nbd
$ sudo modprobe nbd

# Connects the VM disk to a virtual device node
$ sudo qemu-nbd -c /dev/nbd0 -r $VMDISK3((CO1-3))

# Lists the partition numbers available to mount on this disk
$ ls /dev/nbd0p* /dev/nbd0p1 /dev/nbd0p2

# Mounts the selected partition at /mnt with qemu-nbd
$ sudo mount /dev/nbd0p2 /mnt

# Creates a TAR filecalled img.tar from /mnt
$ sudo tar cf img.tar -C /mnt .

# Unmounts and cleans up after qemu-nbd
$ sudo umount /mnt && sudo qemu-nbd -d /dev/nbd0

# Dockerfile
FROM scratch
ADD img.tar /

$ docker build .
```

---

### 10.2 Host Like Container

Containers are not virtual machines—there are significant differences—and pretending there aren’t can cause confusion and issues down the line.

* Differences between VMs and Docker containers:
  - Docker is application-oriented, whereas VMs are operating-system oriented.
  - Docker  containers  share  an  operating  system  with  other  Docker  containers.  Incontrast, VMs each have their own operating system managed by a hypervisor.
  - Docker containers are designed to run one principal process, not manage mul-tiple sets of processes.

```bash
docker run -d phusion/baseimage [ This Image designed to run multiple processes.]
docker exec -i -t container_ID /bin/bash
ps -ef [ It starts (cron, sshd, and syslog). It Much like a host. ]
```

---

### 10.3 Running GUI in Containers

Process : Create  an  image  with  your  user  credentials  and  the  program,  and  bind  mount  your Xserver to it.
Note : another method is to setup VNC server on container

```bash
# Dockerfile for setting up firefox on container
---
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get install -y firefox
RUN groupadd -g GID USERNAME
RUN useradd -d /home/USERNAME -s /bin/bash \-m USERNAME -u UID -g GID
USER USERNAME
ENV HOME /home/USERNAME
CMD /usr/bin/firefox
---

$ docker build -t gui .
$ docker run -v /tmp/.X11-unix:/tmp/.X11-unix -h $HOSTNAME -v $HOME/.Xauthority:/home/$USER/.Xauthority -e DISPLAY=$DISPLAY gui

# It will popup firefox ( which runs on container )
```

---

### 10.4 Using Docker Machine to Provision Docker Hosts

Docker-machine is a tool just like a vagrant. EX: we can setup VM with virtualBox with docker command.
walkthrough:

```bash
# Install Docker Machine on Linux
$ curl -L https://github.com/docker/machine/releases/download/v0.16.2/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine && chmod +x /tmp/docker-machine && sudo cp /tmp/docker-machine /usr/local/bin/docker-machine

# Create a VM by docker daemon on Oracle Virtual Box
$ docker-machine create --driver virtualbox host1

# Run this command to set the DOCKER_HOST environment variable, which sets the default host that Docker commands will be run on
$ docker-machine env host1
$ eval $(docker-machine env host1)

# we can direct to VM
$ docker-machine ssh host1

Commands of Docker-Machine:
create  : Creates a new Machine
ls      : List Machines
stop    : Stop Machines
start   : Start Machines
restart : Restart Machines
rm      : Destroys the machine
inspect : Returns a JSON representation of the machine’s metadata
config  : Return the config of machine
ip      : Returns the IP address of the machine
url     : Returns a URL for the Docker daemon on the machine
upgrade : Upgrades the Docker version on the host to the latest
```

---

### 10.5 Build Images using a Chef Solo

Chef : It is a configuration Management Tool ( using this can reduce the amount of work required to configure Images )

* Here we setup hello world apache website ( Example ).

```bash
# Need a working code
$ git clone https://github.com/docker-in-practice/docker-chef-solo-example.git
# All the working code in there
$ cd into it ( there is a Dockerfile and some other files and folders ( chef recepies etc )
$ docker build -t chef-example .
$ docker run -it -p 8080:80 chef-example
# This is a one time written code we can anywhere to deploy a website ( in such a small steps )

```

---

### 10.6 CI Operations With Docker

CI : Continious Integration

![Image_pipeline.png](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Docker/Docker_Hard_Parts/CIoperations.png)

#### 10.6.1 Steps

* Check out a clean copy of the source code defining the image and build scripts so the origin and process used to build the image is known.
* Retrieve or generate artifacts that will be included in the image, such as the application package and runtime libraries.
* Build the image by using a Dockerfile.
* Verify that the image is structured and functions as intended.
* (Optional) Verify that the image does not contain known vulnerabilities.
* Tag the image so that it can be consumed easily.
* Publish the image to a registry or another distribution channel.

#### 10.6.2 Method 1 : Builds a Image Using DockerHub Workflow ( Test and Push Images )

```bash
# For this you will Git Repo and docker Hub Repo
# Link Docker Hub to to git repo ( it take code from git repo and compile and create a desired Image ( like other ci tools do )
# wait for the docker hub to build to complete
# Remember this is a basic solution

```

#### 10.6.3 Method 2 : Setting up a package cache for faster Builds

While building the images it will take caches instead of download everytime from internet.

```bash
# We are using squid Proxy Here
$ sudo apt-get install squid-deb-proxy
$ check for port 8000
$ create a Docker File

--- using a apt proxy
FROM debian
RUN apt-get update -y && apt-get install net-tools
RUN echo "Acquire::http::Proxy \"http://$( \
route -n | awk '/^0.0.0.0/ {print $2}' \
):8000\";" \
> /etc/apt/apt.conf.d/30proxy
RUN echo "Acquire::http::Proxy::ppa.launchpad.net DIRECT;" >> \
/etc/apt/apt.conf.d/30proxy
CMD ["/bin/bash"]
---

This will cache all the webpages and apt package we downlaod after running this container : if again download them they will be download in miliseconds
```

##### 10.6.3 Method 3 : Running the Jenkins Master Withing the Docker Container

Portable Jenkins Server

```bash
# download git clone https://github.com/docker-in-practice/jenkins.git.
# Put your required plugins in jenkins_plugins.txt

--- Dockerfile
FROM jenkins
COPY jenkins_plugins.txt /tmp/jenkins_plugins.txt
RUN /usr/local/bin/plugins.sh /tmp/jenkins_plugins.txt
USER root
RUN rm /tmp/jenkins_plugins.txt
RUN groupadd -g 999 docker
RUN addgroup -a jenkins docker
USER jenkins
---

$ docker build -t jenkins .
$ docker run --name jenkins -p 8080:8080 \
-p 50000:50000 \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /tmp:/var/jenkins_home \
-d \
jenkins

# go to localhost:8080 : Copy master password from logs

# Reliably Upgrade a Jenkins Server

--- Dockerfile
FROM docker
ADD jenkins_updater.sh /jenkins_updater.sh
RUN chmod +x /jenkins_updater.sh
ENTRYPOINT /jenkins_updater.sh
---

--- Shell script to backup and restart Jenkins
#!/bin/sh
set -e
set -x

if ! docker pull jenkins | grep up.to.date
then
docker stop jenkins
docker rename jenkins jenkins.bak.$(date +%Y%m%d%H%M)
cp -r /var/docker/mounts/jenkins_home \
/var/docker/mounts/jenkins_home.bak.$(date +%Y%m%d%H%M)

docker run -d \
--restart always \
-v /var/docker/mounts/jenkins_home:/var/jenkins_home \
--name jenkins \
-p 8080:8080 \
jenkins
fi
---

# docker Command to run the Jenkins Updater
$ docker run --rm -d -v /var/lib/docker:/var/lib/docker -v /var/run/docker.sock:/var/run/docker.sock -v/var/docker/mounts:/var/docker/mounts dockerinpractice/jenkins-updater

# to automate the process add this command to crontab
0 * * * * dokcker_command
```

---

### 10.7 CD Operations with Docker

CD : Continious Delivery ( CI + Deployment )

```bash

# Copy an Image Between Two Registries
Process => Pulling the image from the registry -> retag -> pushing the new Image
$ docker tag -f $OLDREG/$MYIMAGE $NEWREG/$MYIMAGE
$ docker push $NEWREG/$MYIMAGE
$ docker rmi $OLDREG/$MYIMAGE
$ docker image prune -f

# Copy an Image btw Two Machine With a very low-bandwidth Connection

Process => Here we use backup tool called Bup ( creates a bup data tool )
# Example ( Not Exact data is used so please Don't judge me )
# pull two images like Ubuntu:18.04 & 19.10 ( Both are example = 65 MB Each = 130 MB )
$ mkdir bup_pool
$ alias dbup="docker run --rm \
-v $(pwd)/bup_pool:/pool -v /var/run/docker.sock:/var/run/docker.sock dockerinpractice/dbup"
$ dbup save ubuntu:18.04
$ du -sh bup_pool ( 74 MB )
$ dbup save ubuntu:19.10
$ du -sh bup_pool ( 96 MB ) ( Saves 35 MB )

# On other machine ( rsync from host1 to host2
)
$ dbup load ubuntu:18.04

# Also copies files between host with TAR
Docker export : creates Container To TAR
Docker Import : TAR to Image
Docker save   : Image To TAR
Docker load   : TAR to Docker Image

Example : Transfer docker Image directory over ssh
$ docker export $(docker run -d  debian:7.3 true) | ssh user@host docker import
```

---

### 10.8 Cordination Between Containers

We need coordination between containers : Like if we take an example of one server and one python based echo client.
P1: If we start the client container first : It will lead to failure
P2 : forgetting to remove the containers will result in
problems when you try to restart
P3 :  Naming containers incorrectly will result
in failure.
So how to get rid of this types of problem : we need a solution,where we can run the container without any problem.

```bash

# Solution : create a compose file
---
version: "3"
   services:
     echo-server:
       image: server
       expose:
         - "2000"
     client:
       image: client
       links:
- echo-server:talkto
---
# with this we can start container in correct order & also we call rebuild the container anywhere
$ docker-compose up
Attaching to dockercompose_server_1, dockercompose_client_1
client_1 | Received: Hello, world
client_1 |
client_1 | Received: Hello, world
client_1 |

```

---

## 11.0 Security With Docker

---
This is where security tips about Docker go. The Docker [security](https://docs.docker.com/engine/security/security/) page goes into more detail.

First things first: Docker runs as root. If you are in the `docker` group, you effectively [have root access](https://web.archive.org/web/20161226211755/http://reventlov.com/advisories/using-the-docker-command-to-root-the-host). If you expose the docker unix socket to a container, you are giving the container [root access to the host](https://www.lvh.io/posts/dont-expose-the-docker-socket-not-even-to-a-container/).

Docker should not be your only defense. You should secure and harden it.

For an understanding of what containers leave exposed, you should read [Understanding and Hardening Linux Containers](https://www.nccgroup.trust/globalassets/our-research/us/whitepapers/2016/april/ncc_group_understanding_hardening_linux_containers-1-1.pdf) by [Aaron Grattafiori](https://twitter.com/dyn___). This is a complete and comprehensive guide to the issues involved with containers, with a plethora of links and footnotes leading on to yet more useful content. The security tips following are useful if you've already hardened containers in the past, but are not a substitute for understanding.

### 11.1 Security Tips

For greatest security, you want to run Docker inside a virtual machine. This is straight from the Docker Security Team Lead -- [slides](http://www.slideshare.net/jpetazzo/linux-containers-lxc-docker-and-security) / [notes](http://www.projectatomic.io/blog/2014/08/is-it-safe-a-look-at-docker-and-security-from-linuxcon/). Then, run with AppArmor / seccomp / SELinux / grsec etc to [limit the container permissions](http://linux-audit.com/docker-security-best-practices-for-your-vessel-and-containers/). See the [Docker 1.10 security features](https://blog.docker.com/2016/02/docker-engine-1-10-security/) for more details.

Docker image ids are [sensitive information](https://medium.com/@quayio/your-docker-image-ids-are-secrets-and-its-time-you-treated-them-that-way-f55e9f14c1a4) and should not be exposed to the outside world. Treat them like passwords.

See the [Docker Security Cheat Sheet](https://github.com/konstruktoid/Docker/blob/master/Security/CheatSheet.adoc) by [Thomas Sjögren](https://github.com/konstruktoid): some good stuff about container hardening in there.

Check out the [docker bench security script](https://github.com/docker/docker-bench-security), download the [white papers](https://blog.docker.com/2015/05/understanding-docker-security-and-best-practices/).

Snyk's [10 Docker Image Security Best Practices cheat sheet](https://snyk.io/blog/10-docker-image-security-best-practices/)

You should start off by using a kernel with unstable patches for grsecurity / pax compiled in, such as [Alpine Linux](https://en.wikipedia.org/wiki/Alpine_Linux). If you are using grsecurity in production, you should spring for [commercial support](https://grsecurity.net/business_support.php) for the [stable patches](https://grsecurity.net/announce.php), same as you would do for RedHat. It's $200 a month, which is nothing to your devops budget.

Since docker 1.11 you can easily limit the number of active processes running inside a container to prevent fork bombs. This requires a linux kernel >= 4.3 with CGROUP_PIDS=y to be in the kernel configuration.

```bash
docker run --pids-limit=64
```

Also available since docker 1.11 is the ability to prevent processes from gaining new privileges. This feature have been in the linux kernel since version 3.5. You can read more about it in [this](http://www.projectatomic.io/blog/2016/03/no-new-privs-docker/) blog post.

```bash
docker run --security-opt=no-new-privileges
```

From the [Docker Security Cheat Sheet](http://container-solutions.com/content/uploads/2015/06/15.06.15_DockerCheatSheet_A2.pdf) (it's in PDF which makes it hard to use, so copying below) by [Container Solutions](http://container-solutions.com/is-docker-safe-for-production/):

Turn off interprocess communication with:

```bash
docker -d --icc=false --iptables
```

Set the container to be read-only:

```bash
docker run --read-only
```

Verify images with a hashsum:

```bash
docker pull debian@sha256:a25306f3850e1bd44541976aa7b5fd0a29be
```

Set volumes to be read only:

```bash
docker run -v $(pwd)/secrets:/secrets:ro debian
```

Define and run a user in your Dockerfile so you don't run as root inside the container:

```bash
RUN groupadd -r user && useradd -r -g user user
USER user
```

### 11.2 Security Videos

* [Using Docker Safely](https://youtu.be/04LOuMgNj9U)
* [Securing your applications using Docker](https://youtu.be/KmxOXmPhZbk)
* [Container security: Do containers actually contain?](https://youtu.be/a9lE9Urr6AQ)
* [Linux Containers: Future or Fantasy?](https://www.youtube.com/watch?v=iN6QbszB1R8)

### 11.3 Security Roadmap

The Docker roadmap talks about [seccomp support](https://github.com/docker/docker/blob/master/ROADMAP.md#11-security).
There is an AppArmor policy generator called [bane](https://github.com/jfrazelle/bane), and they're working on [security profiles](https://github.com/docker/docker/issues/17142).

---

Till 21 Aprill

### Path to Complete

* Docker access
* Security Measures In Docker
* Securing Access to Docker
* Security from Outside docker

---
