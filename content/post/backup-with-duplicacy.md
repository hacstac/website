---
title: 'HomeLab Series - Backup with Duplicacy'
date: 2021-02-03T15:43:48+08:00
draft: false
tags: ['Backup', 'Linux', 'DevOps']
categories: ['Backup']
author: 'Akash Rajvanshi'

autoCollapseToc: false
---

### What is Duplicacy?

Duplicacy is a new generation cross-platform cloud backup tool based on the idea of Lock-Free Deduplication. By means of what is this Lock-Free deduplication?. We first look into some general terms then moving forward with this.

<!--more-->

### Deduplication

Deduplication is the identification and elimination of duplicate blocks within a dataset. It is similar to compression, which only identifies redundant blocks in a single file. Deduplication can find redundant blocks of data between files from different directories, different data types, even different servers in different locations.

### How Deduplication works?

The usual way that deduplication works is that data to be deduped is chopped up into chunks. A chunk is one or more contiguous blocks of data. Now these small series of chunks will then be compared against all previous chunks seen by a given deduplication system. The way the comparison works is that each chunk is run through a deterministic cryptographic hashing algorithm, such as SHA-1, SHA-2, or SHA-256, which creates what is called a hash. For example, if one enters “Data” into a SHA-256 hash calculator, you get the following hash value: `cec3a9b89b2e391393d0f68e4bc12a9fa6cf358b3cdf79496dc442d52b8dd528`. If the hashes of two chunks match, they are considered identical, because even the smallest change causes the hash of a chunk to change eg, if you change "D" to "d" in above "data" then your will be `3a6eb0790f39ac87c94f3856b2dd2c5d110e6811602261a9a923d3bb23adc8b7`. A SHA-256 hash is 256 bits. If you create a 256-bit hash for an 8 MB chunk, you save almost 8 MB every time you back up that same chunk. This is why deduplication is such a space saver.

### What is Lock-Free Deduplication?

In lock-free deduplication, Backup system doesn't use a centralized indexing database for tracking all existing chunks and instead, to check if a chunk has already been uploaded before, one can just perform a file lookup via the file storage API using the file name derived from the hash of the chunk. This effectively turns a cloud storage offering only a very limited set of basic file operations into a powerful modern backup backend capable of both block-level and file-level deduplication. More importantly, the absence of a centralized indexing database means that there is no need to implement a distributed locking mechanism on top of the file storage.

By eliminating the chunk indexing database, lock-free duplication not only reduces the code complexity but also makes the deduplication less error-prone. Each chunk is saved individually in its own file, and once saved there is no need for modification. Data corruption is therefore less likely to occur because of the immutability of chunk files. Another benefit that comes naturally from lock-free duplication is that when one client creates a new chunk, other clients that happen to have the same original file will notice that the chunk already exist and therefore will not upload the same chunk again. This pushes the deduplication to its highest level -- clients without knowledge of each other can share identical chunks with no extra effort.

### Features

- Cross Computer Deduplication
- Centralize Chunk Database-less Approach ( Lock-free Deduplication )
- Faster Performance
- Supports wide variety of storage solutions like - S3, Backblaze, DropBox, SFTP, Microsoft OneDrive, Google Drive etc.
- Feature Comparison with others

![Duplicacy Features](https://s3.ap-south-1.amazonaws.com/akash.r/Devops_Notes_screenshots/Backup/Duplicacy_Comparison.png)

Note - Read More Here - https://github.com/gilbertchen/duplicacy

---

## Backup Servers with Duplicacy GUI

Note: Duplicacy GUI is not a free ( it gives 15 day trial ). If you are a home user duplicacy pricing lies 20$ for first year and 5$ after every subsequent year.

{{< highlight bash "linenos=inline">}}
# STEP-1: Download the binary from website : https://duplicacy.com/download.html

# STEP-2: Make it Executable
$ chmod +x ./duplicacy

# STEP-3: Move to /usr/local/bin
$ mv ./duplicacy /usr/local/bin

# STEP-4: Run the Duplicacy Once and Stop It ( It will create a .duplicacy_web dir in yours user home dir )

--------------------------------------------------


# STEP-5: Go to ~/.duplicacy_web dir and Change Listening Address
$ cd ~/.duplicacy_web

# Now your duplicacy will listen on your ip
$ vim settings.json
---
{
  "listening_address": "{ip_of_server}:3875"
}
---

--------------------------------------------------


# STEP-6: Create a Script
---duplicacy.sh
#!/bin/bash

nohup duplicacy > /dev/null 2>&1
---

# Make it Executable
$ chmod +x ./duplicacy.sh

# Move to /usr/local/bin
$ mv duplicacy.sh /usr/local/bin/duplicacy.sh

--------------------------------------------------

# STEP-7: Create system file

$ vim /etc/systemd/system/duplicacy.service
---
[Unit]
Description=Duplicacy Backup Service
After=network.target
After=network.online.target

[Service]
User=root
Type=simple
ExecStart=/usr/local/bin/duplicacy.sh
TimeoutSec=30
Restart=on-failure
RestartSec=30
StartLimitInterval=350
StartLimitBurst=10

[Install]
WantedBy=default.target
---

# Set Permission on file
$ chmod 644 /etc/systemd/system/duplicacy.service

--------------------------------------------------

# STEP-8: Run the systemd service
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now duplicacy.service
$ sudo systemctl status duplicacy.service
{{</ highlight >}}

---

## Duplicacy CLI

{{< highlight bash "linenos=inline">}}
# Download the latest binary from their website

# Intialize a Repository
# Change to directory, You want to backup : Eg- /home/user
$ duplicacy init -e -erasure-coding 5:2 HomeNas sftp://user@ip/dir # IP: 10.0.1.10

# -e : Encrypted
# -erasure-coding - erasure coding to protect against storage corruption: <data shards>:<parity shards> - Eg Data shards: 5, parity shards: 2
# HomeBackup - Name of Storage
# sftp://user@ip/dir - Remote Dir where you want to store your backups

# Add a redundancy to your backups ( add a second storage to existing backup )
$ duplicacy add RemoteNas HomeNas sftp://user@ip/dir # IP: 10.0.1.15
# RemoteNAS - Storage-ID for Second Storage Service

# Backup
$ duplicacy backup -stats -hash

# To list all snapshots
$ duplicacy list -all

# Info about storage
$ duplicacy info -e -storage-name HomeNas sftp://user@ip/dir

# Check Integrity
$ duplicacy check

# Restore
# Change to dir where you want to restore
$ duplicacy restore -r {id}

# For Storing Passwords
# WARNING - This is not safe option
$ export DUPLICACY_PASSWORD=password
$ export DUPLICACY_SSH_PASSWORD=password

# You can also save the passwords in duplicacy config file ( It save your password in repository's .duplicacy dir )
$ duplicacy set -storage HomeNas -key password -value "password"
$ duplicacy set -storage HomeNas -key ssh_password -value "password"
{{</ highlight >}}

---
