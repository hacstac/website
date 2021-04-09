---
title: "How To Monitor Your Raspberry PI's!!"
date: 2020-11-29T15:43:48+08:00
draft: false
tags: ["Linux", "DevOps", "RaspberryPI"]
categories: ["Linux"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

### 1.0 Introduction

There are so many monitoring services out there to monitor Linux servers and IoT Devices like Raspberrypi's. So here, I mention the two leading systems to monitor raspi or any other Linux server.

<!--more-->

1. Prometheus + Node Exporter + Grafana
2. InfluxDB + Telegraf + Grafana

- NOTE: So before moving further, first, I let introduce you to what these monitoring tools are and how they differ from each other:

### 1.1 Comparison Between Prometheus and InfluxDB

* **Prometheus** : Prometheus is a database optimized for time series data and an ideal way to store monitoring metrics. Prometheus graduated from the Cloud Native Computing Foundation (CNCF), which means it has great integration with other CNCF components. It features built-in service discovery, making it easy to use in highly dynamic environments. Unlike traditional monitoring tools, Prometheus uses a pull model which allows it to scale better. To access data, Prometheus offers a flexible query language called PromQL.
  * Prometheus is a pull-based system. An application publishes the metrics at a given endpoint, and Prometheus fetches them periodically.
  * Prometheus is focused on metrics recording.
  * The main advantage of Prometheus is its huge community support, which is based on its CNCF graduated project status. Many applications, especially cloud native ones, already offer Prometheus support out of the box.
  * Prometheus was built with monitoring in mind—especially distributed, cloud native monitoring. It excels in this category, featuring lots of useful integrations with other existing products.
  * Prometheus Uses RESTful HTTP and JSON

---

* **InfluxDB** : InfluxDB is a time series database designed for fast, high-availability storage and retrieval of time series data. It can work as a stand-alone solution, or it can be used to process data from Graphite. In addition to monitoring, InfluxDB can be used for the Internet of Things, sensor data, and home automation solutions. Just like Prometheus, it features its own query language inspired by SQL. Even though the database itself is an open-source project, it implements closed-source components to allow clustering.
  * InfluxDB is a push-based system. It requires an application to actively push data into InfluxDB.
  * InfluxDB is much more suitable for event logging.
  * While InfluxDB also features many integrations, it is not as ‘well-connected’ as Prometheus.
  * InfluxDB can also handle monitoring, it’s not as popular as Prometheus for accomplishing this task. As a result, you may be required to write your own integrations. However, if you are interested in more than just monitoring, InfluxDB is also a great option for storing time series data, such as data coming from sensor networks or data used in real-time analytics (e.g., financial data or Twitter stats).
  * InfluxDb uses HTTP, TCP, and UDP API

---

* For More Detailed Comparison
  * **[DB-Engine : InfluxDB vs Prometheus vs Graphite](https://db-engines.com/en/system/Graphite%3BInfluxDB%3BPrometheus)**
  * **[DB-Engine : InfluxDB vs Prometheus](https://db-engines.com/en/system/InfluxDB%3BPrometheus)**

---

## 2.0 Monitor with Prometheus

### 2.1 Services Used

* Prometheus : Time Series Database
* Grafana : Visualization
* Blackbox_exporter : To probe things using HTTP, ICMP etc etc, check Internet connectivity
* Node_Exporter : Collects host stats. This needs to be running on each host.
* rpi_cpu_stats : Raspberry Pi CPU temp and freq exporter.

### 2.2 Installation

Download the Latest `armv7` version files from [Prometheus Download Center](https://prometheus.io/download/). ( download with wget )

* Download Prometheus
* Download Node_exporter
* Download BlackBox_exporter

#### 2.2.1 Node Exporter

```bash
================================================================================================
Node Exporter
================================================================================================
# Download & Install node_exporter
$ wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-armv7.tar.gz

# un-tar the tar.gz
$ tar -xvzf x.tar.gz

# Copy Binary to the /usr/local/bin
$ sudo cp node_exporter /usr/local/bin

# Make it executable
$ sudo chmod +x /usr/local/bin/node_exporter

# Create a service account for Node_exporter
$ sudo useradd -m -s /bin/bash node_exporter

# Make a directly in /var/lib/ that will be used by the node_exporter.
$ sudo mkdir /var/lib/node_exporter
$ sudo chown -R node_exporter:node_exporter /var/lib/node_exporter

# Setup systemd unit service ( to run the service as a system service )
$ sudo nano /etc/systemd/system/node_exporter.service

# Unit file

---
[Unit]
Description=Node Exporter

[Service]
# Provide a text file location for https://github.com/fahlke/raspberrypi_exporter data with the
# --collector.textfile.directory parameter.
ExecStart=/usr/local/bin/node_exporter --collector.textfile.directory /var/lib/node_exporter/textfile_collector

[Install]
WantedBy=multi-user.target
---

# Reload daemon and start the service
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now node_exporter.service

# To Check Status of Service
$ sudo systemctl status node_exporter.service

# Check for service status
# Get the metrics
$ curl http://localhost:9100/metrics
# Returns with lots of diff kinds of Metrics
```

#### 2.2.2 Black Box Exporter

```bash
================================================================================================
Black Box Exporter
================================================================================================
# 2.0 Blackbox_exporter
$ wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.18.0/blackbox_exporter-0.18.0.linux-armv7.tar.gz

# un-tar
$ tar -xvf x.tar.gz

# Content of dir
.
├── blackbox_exporter
├── blackbox.yml
├── LICENSE
└── NOTICE

# Copy Binary to the /usr/local/bin
$ sudo cp blackbox_exporter /usr/local/bin

# Make it executable
$ sudo chmod +x /usr/local/bin/blackbox_exporter

# Create a service account for Node_exporter
$ sudo useradd --no-create-home --shell /bin/false blackbox_exporter
$ sudo chown blackbox_exporter:blackbox_exporter /usr/local/bin/blackbox_exporter

# Create a Config
$ sudo mkdir /etc/blackbox_exporter
$ sudo cp blackbox.yml /etc/blackbox_exporter/
$ sudo chown blackbox_exporter:blackbox_exporter /etc/blackbox_exporter/blackbox.yml

# Create Systemd unit
$ sudo nano /etc/systemd/system/blackbox_exporter.service

--Unit.file
[Unit]
Description=Blackbox Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=blackbox_exporter
Group=blackbox_exporter
Type=simple
ExecStart=/usr/local/bin/blackbox_exporter --config.file /etc/blackbox_exporter/blackbox.yml

[Install]
WantedBy=multi-user.target
---

# Reload daemon and start the service
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now blackbox_exporter.service
$ sudo systemctl status blackbox_exporter.service

# Check for service status
# check localhost:9115
$ curl http://localhost:9115
```

#### 2.2.3 RPI Cpu Stats

```bash
================================================================================================
RPI Cpu Stats
================================================================================================

# Note: This is installed through docker So first install Docker

# Install
$  docker container run --name rpi_cpu_stats -v /sys:/sys:ro -p 9669:9669 -d thundermagic/rpi_cpu_stats:latest

--or--

# Use docker Compose
version: "2"
services:
  rpi_cpu_stats:
    image: thundermagic/rpi_cpu_stats:latest
    restart: always
    container_name: rpi_cpu_stats
    ports:
      - "9669:9669"
    environment: # Add the PUID and PGID of the user you want to run the container as
      - PUID=1001
      - PGID=1001
    volumes:
      - /sys:/sys

$ docker-compose -f x.yml up -d

# Port in 9669
# This will Export Rpi_cpu_temps
# To check
$ curl http://localhots/9669 # { Returns a various Metrics }
```

#### 2.2.4 Prometheus

```bash
================================================================================================
Prometheus
================================================================================================
# 4.0 Install Prometheus
$ wget https://github.com/prometheus/prometheus/releases/download/v2.23.0/prometheus-2.23.0.linux-armv7.tar.gz

# un-tar
$ tar -xvf ./x.tar.gz

# Content
.
├── console_libraries
│   ├── menu.lib
│   └── prom.lib
├── consoles
│   ├── index.html.example
│   ├── node-cpu.html
│   ├── node-disk.html
│   ├── node.html
│   ├── node-overview.html
│   ├── prometheus.html
│   └── prometheus-overview.html
├── LICENSE
├── NOTICE
├── prometheus
├── prometheus.yml
└── promtool

# Copy Binaries
$ sudo cp prometheus /usr/local/bin
$ sudo cp promtool /usr/local/bin

# Make it executable
$ sudo chmod +x /usr/local/bin/prometheus
$ sudo chmod +x /usr/local/bin/promtool

# Create a Service Account
$ sudo useradd --no-create-home --shell /bin/false prometheus

# Create system dir
$ sudo mkdir /etc/prometheus
$ sudo mkdir /var/lib/prometheus
$ sudo chown prometheus:prometheus /etc/prometheus
$ sudo chown prometheus:prometheus /var/lib/prometheus

$ sudo cp -r ./consoles/ /etc/prometheus
$ sudo cp -r ./console_libraries/ /etc/prometheus

# Set ownership
$ sudo chown prometheus:prometheus /usr/local/bin/prometheus
$ sudo chown prometheus:prometheus /usr/local/bin/promtool
$ sudo chown -R prometheus:prometheus /etc/prometheus/consoles
$ sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries


# Prometheus Config file : Add ports of services
$ sudo nano /etc/prometheus/prometheus.yml

---
global:
  scrape_interval: 15s


scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'blackbox_exporter'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - http://10.0.1.2/admin
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9115


  - job_name: 'rpi_cpu_stats'
    scrape_interval: 5s
    static_configs:
      - targets: ['10.0.1.2:9669']

# RPI IP: 10.0.1.2
---

# Set ownership
$ sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml

# Create a Systemd Unit File
$ sudo nano /etc/systemd/system/prometheus.service

---unit-file
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
---

# Reload daemon and start the service
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now prometheus.service
$ sudo systemctl status prometheus.service

# Check for service status
# Check http://{RPI_IP}:9090
# Check Targets
# Check Various Metrics

```

![Prometheus_Targets_Dasboard](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/RaspberryPI/RPI_Prometheus.png)

#### 2.2.5 Grafana

```bash
================================================================================================
Grafana
================================================================================================
# 5.0 Install Grafana

# Install Grafana with docker
$ docker run -d --name=grafana -p 3000:3000 grafana/grafana

# Default user:pass [admin:admin]
# Configuration -> Add Data Source : Prometheus [pi_ip:9090]

# Load Pannels and Graphs
# For BlackBox :  ID : 7587
# Host Overview : ID : 6287
# rpi_cpu_stats : Create Custom Pannel
# For Containers Monitoring add CAdvisor Service
```

![Host Overview](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/RaspberryPI/Grafana_Host_Overview.png)

![BlackBox](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/RaspberryPI/BlackBox.png)

#### 2.2.6 cAdvisor

```bash
================================================================================================
C_Advisor
================================================================================================
# 6.0 Conatiner Monitoring with cAdvisor

# Install with docker
$ docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  budry/cadvisor-arm:latest

# Check http://{IP}:8080/containers/

# Add To prometheus.yml
- job_name: cadvisor
  scrape_interval: 5s
  static_configs:
  - targets: ['10.0.1.2:8080']


$ sudo systemctl restart prometheus
# Check Service is Successfully Added In Prometheus or not : In Targets

# Pannels
# ID : 893
# ID : 179
```

![Docker Overview](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/RaspberryPI/Docker_Overview.png)

---

## 3.0 Monitor with InfluxDB

### 3.1 Services Used

* InfluxDB : open-source time series database. It is written in Go and optimized for fast, high-availability storage and retrieval of time series data in fields such as operations monitoring, application metrics, Internet of Things sensor data, and real-time analytics

* Grafana : Visualization

* Telegraf : is a plugin-driven server agent for collecting and sending metrics (CPU, RAM, Storage and so on) and events from databases, systems, and IoT sensors.

### 3.2 Installation

#### 3.2.1 InfluxDB

```bash
================================================================================================
InfluxDB
================================================================================================
# Note : If system is ARM64/ aarch64 then direct download and install binaries from influxDb website, Like we do in Prometheus section

# If system is arm7
# Installation
$ curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
$ echo "deb https://repos.influxdata.com/debian stretch stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

$ sudo apt-get update && sudo apt-get install influxdb
$ sudo systemctl unmask influxdb.service
$ sudo systemctl enable --now influxdb

# Create a DB and User
$ influx
$ create database rpi_monitoring;
$ show databases;
$ CREATE USER demo WITH PASSWORD 'demo' WITH ALL PRIVILEGES;
$ show users;
$ exit
```

#### 3.2.2 Telegraf

```bash
================================================================================================
Telegraf
================================================================================================
# Install Telegraf
# Before adding Influx repository, run this so that apt will be able to read the repository.

$ sudo apt-get update && sudo apt-get install apt-transport-https
$ sudo apt-get install telegraf
$ sudo systemctl enable --now telegraf
$ sudo systemctl status telegraf

# Add telegraf user to other groups ( To collect Metrics )
$ sudo usermod -aG docker telegraf ( To collect docker metrics )
$ sudo usermod -aG video telegraf ( To collect GPU metrics )


# Config File : /etc/telegraf/telegraf.conf
# Uncomment These
---
[global_tags]
[agent]
  interval = "10s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "10s"
  flush_jitter = "0s"
  precision = ""
  hostname = ""
  omit_hostname = false
[[outputs.influxdb]]
  urls = ["http://<<INFLUX-DB-Server>>:8086"]
  database = "rpi_monitoring"
  username = "demo"
  password = "demo"
[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false
  report_active = false
[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]
[[inputs.mem]]
[[inputs.swap]]
[[inputs.file]]
  files = ["/sys/class/thermal/thermal_zone0/temp"]
  name_override = "cpu_temperature"
  data_format = "value"
  data_type = "integer"
[[inputs.exec]]
  commands = [ "/opt/vc/bin/vcgencmd measure_temp" ]
  name_override = "gpu_temperature"
  data_format = "grok"
  grok_patterns = ["%{NUMBER:value:float}"]
[[inputs.net]]
  interfaces = ["eth0"]
[[inputs.netstat]]
[[inputs.ping]]
  urls = ["www.google.com"] # required
  count = 4
  interface = "wlan0"
  name_override = "google_ping"
[[inputs.docker]]
  endpoint = "unix:///var/run/docker.sock"
[[inputs.dns_query]]
  servers = ["8.8.8.8"]
  domains = ["."]
  record_type = "A"
  timeout = 10
  name_override = "google_dns"
[[inputs.kernel]]
[[inputs.system]]
[[inputs.processes]]
[[inputs.diskio]]
---

# Restart Telegraf
$ sudo systemctl restart telegraf

# Telegraf Supports Multiple Plugins  : Visit https://github.com/influxdata/telegraf to Install & Enable Custom Plugins

-------------------------------------------------------------------------------------
# Grafana
# Add data source in Grafana
db : rpi_monitoring
user : demo
password : demo
Url : http://localhost:8086

# Grafana Dashboards
Raspberry Pi Monitoring (ID 10578) by Jorge de la Cruz
InfluxDB Docker (ID 1150)by Thomas Cheronneau
```

![Influx RPI Monitoring](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/RaspberryPI/Influx_RPI_Monitoring.png)

![Influx Docker Monitoring](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/RaspberryPI/Influx_db_docker.png)

---

## 4.0 Monitor with Telegram Bot

RPi-TELEBOT : Python based Telegram bot to monitor and control the raspberry pi

```bash
# Install on Raspi
$ sudo apt-get install python-pip
$ sudo pip3 install telepot

# In telegram search for BotFather and create a new Bot
# Set up an API Token
# Copy API Token

# Clone RPI-Telbot Repo
$ git clone https://github.com/RNA3210d/RPi-TELEBOT.git

# cd into it and edit gpiotel20.py
# Paste API Token
# bot = telepot.Bot('  Enter your Telegram bot API token here  ')

# Run gpiotel20 as sudo
$ sudo python3 gpiotel20.py &

# Check Connection in Telegram
# Search for Bot Name
# /hi
Hi. BLEEP..BLOP.., bot is online

# Other Commands
# /help - List of commands
# /ledon1 - Switch on LED 1
# /ledoff1 - Switch off LED 1
# /ledon2 - Switch on LED 2
# /ledoff2 - Switch off LED 2
# /cpu - Get CPU info (lscpu)
# /usb - See connected USB devices (lsusb)
# /hi - To check if online
# /time - Returns time
# /date - Returns date
# /temp - CPU Temperature
# /repoupdate - update repositories (sudo apt-get update)
# /upgrade - upgrade packages (sudo apt-get upgrade -y)
# /shutdown - Shutdown RPi (sudo shutdown -h now)
# /reboot - Reboot RPi (sudo reboot)
```

Note: If you are using an amd64 machine, then you can use `stefanprodan/dockprom` :

{{< highlight bash "linenos=inline">}}
- Prometheus (metrics database) http://<host-ip>:9090
- Prometheus-Pushgateway (push acceptor for ephemeral and batch jobs) http://<host-ip>:9091
- AlertManager (alerts management) http://<host-ip>:9093
- Grafana (visualize metrics) http://<host-ip>:3000
- NodeExporter (host metrics collector)
- cAdvisor (containers metrics collector)
- Caddy (reverse proxy and basic auth provider for prometheus and alertmanager)
{{</ highlight >}}

It setup all of these services in docker containers in just 2 mins, without any other configuration.

---
