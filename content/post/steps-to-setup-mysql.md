---
title: 'HomeLab Series - MySQL 101'
date: 2021-01-31T15:43:48+08:00
draft: false
tags: ['Database', 'Linux', 'DevOps']
categories: ['Database']
author: 'Akash Rajvanshi'

autoCollapseToc: false
---

### What is MySQL

MySQL is an open-source relational database management system (RDBMS). Its name is a blend of “My”, the name of prime supporter Michael Widenius’ little girl, and “SQL”, the abbreviation for Structured Query Language. Mastering MySQL is a must for programmers. Especially when associated with web programming that (almost) all use MySQL as a database.

---

<!--more-->

## MariaDB? 🤔

The MySQL improvement extend has made its source code accessible under the terms of the GNU General Public License, and under an assortment of restrictive understandings. MySQL was possessed and supported by a solitary revenue-driven firm, the Swedish Organization MySQL AB, now claimed by Oracle Corporation ( For exclusive use, a few paid releases are accessible and offer extra usefulness. ). During the acquisition of Sun Microsystems by Oracle, some of the senior engineers who were working on the development of MySQL felt that there is a conflict of interest between MySQL and Oracle’s commercial database - Oracle Database Server. As a result, these engineers created a fork of MySQL code base and started their own organization. This is how MariaDB was born. MariaDB is free under the GNU GPL. It is outstanding for being driven by the first designers of MySQL, who forked it because of worries over its procurement by Oracle. Contributors are required to impart their copyright to the MariaDB Foundation. MariaDB means to keep up high similarity with MySQL, guaranteeing a “drop-in” supplanting capacity with library twofold equivalency and correct coordinating with MySQL APIs and commands.

MariaDB offers more and better storage engines. NoSQL support, provided by Cassandra, allows you to run SQL and NoSQL in a single database system. MariaDB also supports TokuDB, which can handle big data for large organizations and corporate users. MySQL’s usual (and slow) database engines MyISAM and InnoDB are replaced in MariaDB by Aria and XtraDB respectively. Aria offers better caching, which makes a difference when it comes to disk-intensive operations. Temporary tables also use Aria, which speeds up complex queries, such as those involving GROUP BY and DISTINCT. Percona’s XtraDB gets rid of all of the InnoDB problems with slow performance and stability, especially in high load environments.

---

## Installation

```bash
================================================================================================
Arch
================================================================================================

# Note: Arch Linux favours Mariadb over MySQL. If you want to use mysql on arch, User mysql (aur) package.

$ sudo pacman -S mariadb

$ sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql

$ sudo systemctl enable --now mariadb.service

================================================================================================
Ubuntu 20.04
================================================================================================

$ sudo apt update

# For MySQL
$ sudo apt install mysql-server

# For MariaDB
$ sudo apt install mariadb-server

# Check the service status
$ sudo systemctl status mysql.service | mariadb.service

================================================================================================

## Configuration

# For Intial Security
$ sudo mysql_secure_installation

# To check version of database
$ mysql

mysql> SELECT VERSION();
+----------------+
| VERSION()      |
+----------------+
| 10.5.8-MariaDB |
+----------------+

# Configuration Files
# Arch
/etc/my.cnf || /etc/my.cnf.d || ~/.my.cnf

# Ubuntu 20.04 Server
/etc/mysql/my.cnf || /etc/mysql/my.cnf.d

# Note: By default, MySQL will listen on the 0.0.0.0 address ( So, It is open for everyone ), To restrict

# Change Bind Address
# On Arch
$ sudo vim /etc/my.cnf.d/server.cnf

# On Ubuntu 20.04
$ sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
bind-address = 127.0.0.1

# To turn off the Networking Access ( listen only on via unix sockets )
$ sudo vim /etc/my.cnf.d/server.cnf
[mysqld]
skip-networking

# To Change Encoding and Collate
# Note: Before changing the character set sure to create a backup first
$ sudo vim /etc/my.cnf.d/server.cnf
[client]
default-character-set = utf8mb4

[mysqld]
collation_server = utf8mb4_unicode_ci
character_set_server = utf8mb4

[mysql]
default-character-set = utf8mb4

# Enable auto completion
$ sudo vim /etc/my.cnf/d/mysql-clients.cnf
[mysql]
auto-rehash

# Check Remote Access
# Note: For remote access ensure that mysql has networking enabled and listing on the appropriate interface ( Also open a port of your db from firewall )

$ mysql -u root -p
# Allow anyone in network 10.0.1.x can access this db with this user
> GRANT ALL PRIVILEGES ON *.* TO 'root'@'10.0.1.%' IDENTIFIED BY 'password' WITH GRANT OPTION;

> SELECT User, Host FROM mysql.user WHERE Host <> 'localhost';
+------+----------+
| User | Host     |
+------+----------+
| root | 10.0.1.% |
+------+----------+

# Client Installation

# Note - Mysql client to access remote db from your local machine
# Ubuntu
$ sudo apt install mysql-client

# Arch
$ sudo pacman -S mariadb-clients

# GUI Client - DBeaver - https://dbeaver.io/
```

---

## MySQL Basics

### Create a User & DB

```bash
# Create User
# Create a local user
mysql> CREATE USER 'backops'@'localhost' IDENTIFIED BY 'password';

# Create a remote user
mysql> CREATE USER 'backops'@'10.0.1.%' IDENTIFIED BY 'password';

# Check user
mysql> SELECT User From mysql.user;
+-------------+
| User        |
+-------------+
| backops     |
| root        |
| mariadb.sys |
| mysql       |
+-------------+

--------------------------------------------------------------------------
# Create Database
mysql> CREATE DATABASE example;

# Check Databases
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| example            |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+

# To use particular database
mysql> use example;

--------------------------------------------------------------------------
# Grants Privileges to a User

# Grant all privileges to all the databases
mysql> GRANT ALL PRIVILEGES ON *.* TO 'backops'@'localhost';

# Grant all privileges to Particular database
mysql> GRANT ALL PRIVILEGES ON example.* TO 'backops'@'localhost';

# To create a user - superuser
mysql> GRANT ALL PRIVILEGES ON *.* TO 'username'@'%' WITH GRANT OPTION;

# All the commonly used permissions :
ALL – Allow complete access to a specific database. If a database is not specified, then allow complete access to the entirety of MySQL.
CREATE – Allow a user to create databases and tables.
DELETE – Allow a user to delete rows from a table.
DROP – Allow a user to drop databases and tables.
EXECUTE – Allow a user to execute stored routines.
GRANT OPTION – Allow a user to grant or remove another user’s privileges.
INSERT – Allow a user to insert rows from a table.
SELECT – Allow a user to select data from a database.
SHOW DATABASES- Allow a user to view a list of all databases.
UPDATE – Allow a user to update rows in a table.

# To give any specific permissions
mysql> GRANT {type_of_permission} ON {database_name.table_name} TO 'username'@'localhost';

--------------------------------------------------------------------------
# Show privileges of user
mysql> show grants for 'backops'@'localhost';

# To save th privileges
mysql> FLUSH PRIVILEGES;
```

---

### Basic CRUD Operations on MySQL

```bash
# Create a table ( users )
mysql> CREATE TABLE IF NOT EXISTS users (
    -> id INT AUTO_INCREMENT NOT NULL PRIMARY KEY,
    -> name VARCHAR(30) NOT NULL,
    -> age INT NOT NULL);

mysql> SHOW COLUMNS IN users;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | int(11)     | NO   | PRI | NULL    | auto_increment |
| name  | varchar(30) | NO   |     | NULL    |                |
| age   | int(11)     | NO   |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+

# Insert ( C )
mysql> INSERT INTO users VALUES (1, 'Jon', 24), (2, 'Frank', 29), (3, 'William', 32);

--------------------------------------------------------------------
# Read ( R )

mysql> SELECT * FROM users;
+----+---------+-----+
| id | name    | age |
+----+---------+-----+
|  1 | Jon     |  24 |
|  2 | Frank   |  29 |
|  3 | William |  32 |
+----+---------+-----+

--------------------------------------------------------------------
# Update ( U )

mysql> UPDATE users SET age = 42 WHERE name = 'Frank';

mysql> SELECT * FROM users;
+----+---------+-----+
| id | name    | age |
+----+---------+-----+
|  1 | Jon     |  24 |
|  2 | Frank   |  42 |
|  3 | William |  32 |
+----+---------+-----+

--------------------------------------------------------------------
# Delete ( D )

mysql> DELETE FROM users WHERE name = 'William';

mysql> SELECT * FROM users;
+----+-------+-----+
| id | name  | age |
+----+-------+-----+
|  1 | Jon   |  24 |
|  2 | Frank |  42 |
+----+-------+-----+
```

---

## Deploy MySQL-DB On Docker Container

```bash
$ vim .env
---
MYSQL_ROOT_PASSWORD=root_password
MYSQL_DATABASE=example_db
MYSQL_USER=backops
MYSQL_PASSWORD=user_password

# Note: Use passwords as a text is not a good choice, use any secret-manager to manage secrets
---

$ docker run -d --name=MySQL --env-file=./.env --mount source=MySQL-Vol,target=/var/lib/mysql -p 3308:3306 mysql:latest

-d = detached mode
--env-file =  We use this to set Enviornment Variables
-p = host port to container
--mount = volume ( Creates a MySQL-Vol and mount it on container /var/lib/mysql )
mysql = docker image ( https://hub.docker.com/_/mysql )
# Note: To use specifc version of mysql ( Eg: 5.7 ), use mysql:5.7 as an image and for mariadb use image as mariadb ( all the config are remain same )

# To check a DB & User
$ docker exec -it {container_id} bash
$ mysql -u root -p

# Check DB
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| example_db         |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+

# Check a User
mysql> SELECT User FROM mysql.user;
+------------------+
| User             |
+------------------+
| backops          |
| root             |
| mysql.infoschema |
| mysql.session    |
| mysql.sys        |
| root             |
+------------------+
```

---

## Automate with Ansible

To follow these steps everytime when you create a new VM instance is a very time consuming and boring stuff. Here I use to automate the setup with ansible.

- Needs
  - Install MySQL
  - Setup MySQL
    - Enable and Start the MySQL Systemd Service
    - Setup Configs
    - Setup on Custom Dir
  - Remote Access
  - Create a Users and Databases


```bash
# Note: Here, I use preconfigured ansible role to setup mysql database. Created By GeerlingGuy ( https://github.com/geerlingguy/ansible-role-mysql ). You can reference this and create your own role to setup mysql.


# STEP - 1 : Install Ansible

# STEP - 2 : Download geerlingguy's - ansible-role-postgresql
$ ansible-galaxy install geerlingguy.mysql

# STEP - 3 : Setup Ansible Env

# Note: Before proceding, please add a password-less communication between your client and server

# Create a Dir and Create these files
.
|____mysql.yaml
|____hosts
|____ansible.cfg
```

```bash
# ansible.cfg
---
[defaults]
inventory = hosts
remote_user = ubuntu
private_key_file = /home/user/.ssh/id_rsa
host_key_checking = False
deprecation_warnings = False
interpreter_python = /usr/bin/python3
---
```

```bash
# hosts
---
[all:vars]
ansible_user = ubuntu
ansible_become = yes
ansible_become_method = sudo

[server]
server1 ansible_host=10.0.1.11 ansible_port=22
server2 ansible_host=10.0.1.12 ansible_port=22
---
```

```yaml
# mysql.yaml
---
- name: Install MySQL
  hosts: server
  become: yes
  roles:
    - geerlingguy.mysql
  vars:
    mysql_root_home: /root
    mysql_root_username: root
    mysql_root_password: 'password'
    mysql_root_password_update: true
    mysql_user_home: /home/ubuntu
    mysql_user_name: ubuntu
    mysql_user_password: 'password'
    mysql_enabled_on_startup: true
    mysql_datadir: /var/lib/mysql
    mysql_port: "3306"
    mysql_bind_address: "0.0.0.0"
    mysql_databases:
      - name: "example_db"
        encoding: "utf8mb4"
        collation: "utf8mb4_0900_ai_ci"
    mysql_users:
      - name: backops
        host: "%"
        password: "password"
        encrypted: no
        priv: "*.*:ALL"
        state: present
---

# Note: This is a test env, That's why i put password's in plain text ( Please use ansible_vault or any secret manager to encrypt you secrets )
# You can also use mariadb, Add these packages in vars -

---
mysql_packages:
  - mariadb-client
  - mariadb-server
	- python-mysqldb
---
```

```bash
# Run a playbook
$ ansible-playbook -i hosts mysql.yaml

# BOOM: This playbook will install and setup mysql and create a backops user with SUPERUSER privilege and also creates a DB ( example_db ).
# Now, whenever you setup a new server machine. You only need to run this playbook. You can also setup this to automate mysql docker container.
```

---

## How to Backup & Restore MySQL-DB

### Backup

```bash
## MySQLDUMP

# To Backup all databases ( Point In Time Recovery )
$ mysqldump --single-transaction --flush-logs --master-data=2 --all-databases -u root -p | gzip > all_databases.sql.gz
# -u : user
# -p : asks for password
# gzip : for compressed backups

# To backup single DB
$ mysqldump -h {host} --port {port} --single-transaction --flush-logs --master-data=2 -u $USER -p$PASS $DB | gzip > example_db--backup--$DATE.sql.gz

# To directly backup a db from docker container
$ docker exec {Container_ID} /usr/bin/mysqldump -h {host} --port {port} --single-transaction --flush-logs --master-data=2 -u $USER -p$PASS $DB | gzip > example_db--backup--$DATE.sql.gz

# Note: If you want to create a script to regularly backup your db ( Here's the template )
---[x-SQL.sh]
#!/bin/bash

USER=backops
PASS=password
DB=example_db
DATE=$(date +%Y-%m-%d--%H-%M)

mysqldump -h {host} --port {port} --single-transaction --flush-logs --master-data=2 -u $USER -p$PASS $DB | gzip > example_db--backup--$DATE.sql.gz

# file: x-backup-2021-01-31--11--30.sql.gz
---

--------------------------------------------------------------------
## MariaDB Backup

# Install MariaDB-Backup
$ sudo apt install mariadb-backup

# Backup a DB
# --target-dir - where to place the backup files.
$ mariabackup --backup --target-dir=/var/mariadb/backup/ --user=backops --password=password
```

### Restore

```bash
## MySQLDUMP
# ( x.sql.gz )
$ zcat example_db-backup-2021-01-31--11--30.sql.gz | mysql -h {host} -P {Port} -u {user} -p {db}

## MariaDB-Backup
# Step 1 - Stop MariaDB
$ systemctl stop mariadb

# Step 2 - Prepare Task
$ mariabackup --prepare --target-dir=/var/mariadb/backup/

# Step 3 - Run Restore
# Note: ensure that the datadir is empty ( /var/lib/mysql ).
$ mariabackup --copy-back --target-dir=/var/mariadb/backup/

# Step 4 - Assign mysql as a user and group
$ chown -R mysql:mysql /var/lib/mysql

# Step 5 - Start MariaDb
$ systemctl start mariadb
```

---
