---
title: "HomeLab Series - PostgreSQL 101"
date: 2021-01-28T15:43:48+08:00
draft: false
tags: ["Database", "Linux", "DevOps"]
categories: ["Database"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

### What is PostgresSQL

PostgreSQL is powerful, open-source advanced object-relational database management system that supports an extended subset of the SQL standard, including transactions, foreign keys, subqueries, triggers, user-defined types and functions. It is also one of the most used databases in the industry for production and deployment.

---

<!--more-->

## Installing PostgreSQL


```bash
================================================================================================
Arch
================================================================================================

$ sudo pacman -S postgresql

# Create a DB Cluster
$ sudo -u postgres -i initdb --locale $LANG -E UTF8 -D /var/lib/postgres/data

# Start and Enable Service
$ sudo systemctl enable --now postgresql.service

# Check Status
$ sudo systemctl status postgresql.service

# pgadmin
$ sudo pacman -S pgadmin4

================================================================================================
Ubuntu 20.04
================================================================================================

# Create the file repo config
$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Import the repo signing key:
$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Install postgres
$ sudo apt update
$ sudo apt install postgresql-12

# Check Status ( if not enabled: Enabled It )
$ sudo systemctl status postgresql.service

# Note: If you want to only install postgres-client ( which will be use to connect to existing remote db's ) ->
$ sudo apt install postgresql-client-12

# To Install GUI Client ( pdadmin4 )
$ sudo apt install pgadmin4

# Note: if you are getting issues with postgres
$ sudo pacman -S postgresql-libs
```

---

## PostgreSQL Basics

### Create a User & DB

```bash

# Switch to postgres user
$ sudo -iu postgres

# Create User
$ createuser --interactive -P
Enter name of role to add: backops
Enter password for new role:
Enter it again:...

# Create a DB
$ createdb -O {db_user} {db_name}

# Check users
$ psql -c "select usename from pg_user;"

 usename
----------
 backops
 postgres
(2 rows)

# Check DB
$ psql backops
backops=# \l
                                   List of databases
    Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
------------+----------+----------+-------------+-------------+------------------------
 backops    | backops  | UTF8     | en_IN.UTF-8 | en_IN.UTF-8 | =Tc/backops            +
            |          |          |             |             | backops=CTc/backops    +
            |          |          |             |             | postgres=C*T*c*/backops
 postgres   | postgres | UTF8     | en_IN.UTF-8 | en_IN.UTF-8 |
```

### Granting and Roles

```sql
# GRANT - Define Access Privileges

# Grant Connect to the DB
postgres=# GRANT CONNECT ON DATABASE db_name TO user;

# Grant USAGE to schema
postgres=# GRANT USAGE ON SCHEMA schema_name TO user;

# Grant CRUD to the DB
postgres=# GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA schema_name TO user;

# Grant ALL privileges on all Tables in the schema
postgres=# GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA schema_name TO user;

# Grant all privileges on the database
postgres=# GRANT ALL PRIVILEGES ON DATABASE db_name TO user;

#------------------------------------------------------------------#
# Alter Role - Change a database role

# Give Permission to create database
postgres=# ALTER USER user_name CREATEDB;

# Give Permission to create roles
postgres=# ALTER USER user_name CREATEROLE;

# Give Permission to create users
postgres=# ALTER USER user_name CREATEUSER;

# Make a user SUPERUSER
postgres=# ALTER USER user_name WITH SUPERUSER;
```

### Accessible from other hosts

```bash
# PostgreSQL Dir Locations
  # Ubuntu: /etc/postgresql/12/main/
  # Arch: /var/lib/postgres/data

# for Arch
----------------------------------------------------------------------------------
# Check for listen address
$ grep listen_addresses /var/lib/postgres/data/postgresql.conf
listen_addresses = 'localhost'		# what IP address(es) to listen on;

# To make it available on your network
$ sudo vim /var/lib/postgres/data/postgresql.conf

# To accept any host
listen_addresses = '*'

# To accept only localhost and particular ip
listen_addresses = 'localhost,local_ip'

----------------------------------------------------------------------------------
# Check for authentication
$ grep -v -E "^#|^$" /var/lib/postgres/data/pg_hba.conf
local   all             all                                     trust
host    all             all             127.0.0.1/32            trust
host    all             all             ::1/128                 trust
local   replication     all                                     trust
host    replication     all             127.0.0.1/32            trust
host    replication     all             ::1/128                 trust

# To make it available for your hosts
$ sudo vim /var/lib/postgres/data/pg_hba.conf ( put it to the end )

# Authenticate to only host
host    all             all             ip_address/24   md5

# Authenticate to everyone
host    all             all             0.0.0.0/0      md5
----------------------------------------------------------------------------------

# Restart Postgres
$ sudo systemctl restart postgresql.service
# Note: for ubuntu use the above listed dir
```

### Basic psql Commands

```bash
# Daily Use psql commands
\? : show all psql commands.
\h : sql-command: show syntax on SQL command.
\c : dbname [username]: Connect to database, with an optional username (or \connect)
\l : List all database (or \list).
\d : Display all tables, indexes, views, and sequences.
\dt : Display all tables.
\di : Display all indexes.
\dv : Display all views.
\ds : Display all sequences.
\dT : Display all types.
\dS : Display all system tables.
\du : Display all users.
\x : auto|on|off: Toggle|On|Off expanded output mode.
```

### Basic CRUD Operations with PostgreSQL

#### Create

```
# Basic CRUD Operations

# Create a DB
postgres=# CREATE DATABASE test;

# use db: test
postgres=# \c test

---------------------------------------------------------------------------------
## Create
# We will be using an enumeration (TYPE) in our table
test=# CREATE TYPE cat_enum AS ENUM ('coffee', 'tea');

-- Create a new table.
test=# CREATE TABLE IF NOT EXISTS cafe (
  id SERIAL PRIMARY KEY,        -- AUTO_INCREMENT integer, as primary key
  category cat_enum NOT NULL,   -- Use the enum type defined earlier
  name VARCHAR(50) NOT NULL,    -- Variable-length string of up to 50 characters
  price NUMERIC(5,2) NOT NULL,  -- 5 digits total, with 2 decimal places
  last_update DATE              -- 'YYYY-MM-DD'
);

-- Insert Rows
test=# INSERT INTO cafe (category, name, price) VALUES
  ('coffee', 'Espresso', 3.19),
  ('coffee', 'Cappuccino', 3.29),
  ('coffee', 'Caffe Latte', 3.39),
  ('coffee', 'Caffe Mocha', 3.49),
  ('coffee', 'Brewed Coffee', 3.59),
  ('tea', 'Green Tea', 2.99),
  ('tea', 'Wulong Tea', 2.89);

-- Display all tables
test=# \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | cafe | table | postgres
(1 row)
```

#### Read

```
test=# SELECT * FROM cafe;
 id | category |     name      | price | last_update
----+----------+---------------+-------+-------------
  1 | coffee   | Espresso      |  3.19 | 2013-12-14
  2 | coffee   | Cappuccino    |  3.29 | 2013-12-14
  3 | coffee   | Caffe Latte   |  3.39 | 2013-12-14
  4 | coffee   | Caffe Mocha   |  3.49 | 2013-12-14
  5 | coffee   | Brewed Coffee |  3.59 | 2013-12-14
  6 | tea      | Green Tea     |  2.99 | 2013-12-14
  7 | tea      | Wulong Tea    |  2.89 | 2013-12-14
```

#### Update

```
# Update
test=# UPDATE cafe SET price = price * 1.1 WHERE category = 'tea';

test=# SELECT * FROM cafe WHERE category = 'tea';
 id | category |    name    | price | last_update
----+----------+------------+-------+-------------
  6 | tea      | Green Tea  |  3.29 | 2013-12-14
  7 | tea      | Wulong Tea |  3.18 | 2013-12-14
(2 rows)
```

#### Delete

```
# Delete
test=# DELETE FROM cafe WHERE id = 6;

test=# SELECT * FROM cafe WHERE category = 'tea';
 id | category |    name    | price | last_update
----+----------+------------+-------+-------------
  7 | tea      | Wulong Tea |  3.18 | 2013-12-14
(1 row)
```

---

## Deploy PostgreSQL on Docker Container

```bash
$ docker run -d --name=postgreSQL -e POSTGRES_PASSWORD='password' -p 5432:5432 -v postgresql-data:/var/lib/postgresql/data postgres

-d = detached mode
-e = env variable ( we use it for password )
-p = host port to container
-v = volume ( docker creates a volume postgresql_data & mount it to container: /var/lib/postgresql/data )
postgres = docker image ( https://hub.docker.com/_/postgres )

# To create a Db & User
# You can also create these with env variables, as given above in docker command
$ docker exec -it {container_id} bash
$ psql -U postgres
> CREATE DATABASE database_name;
> CREATE USER user_name WITH PASSWORD 'password';
> GRANT ALL PRIVILEGES ON DATABASE database_name TO user_name;
> \du # List users
> \l  # list databases
> \q  # Quit
> \dt # List Tables
```

---

## Automate with Ansible

To follow these steps everytime when you create a new VM instance is a very time consuming and boring stuff. Here I use to automate the setup with ansible.

- Needs
  - Install PostgreSQL
  - Setup PostgreSQL
    - Enable and Start the Postgresql Systemd Service
    - Setup on Custom Dir
  - Remote Access
  - Create a Users and Databases

```bash
# Note: Here, I use preconfigured ansible role to setup postgresql database. Created By GeerlingGuy ( https://github.com/geerlingguy/ansible-role-postgresql ). You can reference this and create your own role to setup postgresql.


# STEP - 1 : Install Ansible

# STEP - 2 : Download geerlingguy's - ansible-role-postgresql
$ ansible-galaxy install geerlingguy.postgresql

# STEP - 3 : Setup Ansible Env

# Note: Before proceding, please add a password-less communication between your client and server

# Create a Dir and Create these files
.
|____postgresql.yaml
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
# PostgreSQL.yaml
---
- name: Install PostgreSQL
  hosts: server
  become: yes
  roles:
    - geerlingguy.postgresql
  vars:
    ansible_python_interpreter: /usr/bin/python3
    postgresql_unix_socket_directories:
      - /var/run/postgresql
    postgresql_service_enabled: true
    postgresql_service_state: started
    postgresql_restarted_state: "restarted"
    postgresql_user: postgres
    postgresql_group: postgres
    postgresql_locales:
      - 'en_US.UTF-8'
    postgresql_hba_entries:
      - { type: host, database: all, user: all, address: '10.0.1.0/24', auth_method: md5 }
    postgresql_databases:
      - name: test_db
        owner: backops
        state: present
    postgresql_users:
      - name: backops
        password: iamverysecure
        encrypted: true
        db: test_db
        priv: 'all'
        role_attr_flags: 'SUPERUSER'
        state: present
    postgresql_users_no_log: false
---
```

```bash
# Run a playbook
$ ansible-playbook -i hosts postgresql.yaml

# BOOM: This playbook will install and setup postgresql and create a backops user with SUPERUSER privilege and also creates a DB ( test_db ).
# Now, whenever you setup a new server machine. You only need to run this playbook. You can also setup this for creating a postgres docker container.
```

---

## How to Backup & Restore PostgreSQL-DB

### Backup
```bash
$ pg_dump -h host_name -U user_name -W -d db_name -Ft | gzip > x.sql.gz
# -W : ask for password
# -F : format of the output file
  # p – plain-text SQL script
  # c – custom-format archive
  # d – directory-format archive
  # t – tar-format archive
# gzip : for compressed backups

# Note: If you want to create a script to regularly backup your db ( Here's the template )
---[x-SQL.sh]
#!/bin/bash

USER=username
DB=password
HOST=hostname
DATE=$(date +%Y-%m-%d--%H--%M)


PGPASSWORD="password" pg_dump -h $HOST -U $USER -d $DB -Fp | gzip > x-backup-$DATE.sql.gz
---

# file: x-backup-2020-12-08--11--30.sql.gz

# Note: For all pg_dump options: https://www.postgresql.org/docs/13/app-pgdump.html
```
- **Note**: You can also use backup scripts like this, that create a backup of your and pushed to the cloud: eg: S3. ( Please Read The Code Before Using )
  - [postgres-manage-python](https://github.com/valferon/postgres-manage-python)

### Restore

```bash
# ( x.sql )
$ psql -h hostname -U user-name -d database < x.sql

# ( x.tar or x.dump )
$ pg_restore --clean --verbose -h hostname -U user -d database x.dump

# ( x.sql.gz )
$ zcat x.sql.gz | psql -h hostname -U user -d database
```

---
