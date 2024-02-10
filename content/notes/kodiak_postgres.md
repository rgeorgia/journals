---
title: "Kodiak Import Notes"
date: 2022-07-31T10:12:22-04:00
draft: true
categories: [notes]
tags: [kodiak, db, ]
---

# Postgresql

I created a couple of jails, one of which I will use for our postgres server.
I wanted to add the servers to the `/etc/hosts` file so I can easily ssh in using the name and it's a place to keep track of the IP address.


```bash
# Jails
192.168.1.187   kodiakdb
192.168.1.188   kodiakweb
```

added keys 

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub kodiakdb
ssh-copy-id -i ~/.ssh/id_rsa.pub kodiakweb
```
## Initializing and starting the database

This is the message we get when postgres is installed. I preserve it here so important info about pg does not get lost of forgotten.


```bash
sudo /usr/local/etc/rc.d/postgresql initdb
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with this locale configuration:
  provider:    libc
  LC_COLLATE:  C
  LC_CTYPE:    C.UTF-8
  LC_MESSAGES: C.UTF-8
  LC_MONETARY: C.UTF-8
  LC_NUMERIC:  C.UTF-8
  LC_TIME:     C.UTF-8
The default text search configuration will be set to "english".

Data page checksums are disabled.

creating directory /var/db/postgres/data15 ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... US/Eastern
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
hint: You can change this by editing pg_hba.conf or using the option -A, or --auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    /usr/local/bin/pg_ctl -D /var/db/postgres/data15 -l logfile star
```
## Starting the pg service

```bash
[57471] LOG:  ending log output to stderr
[57471] HINT:  Future log output will go to log destination "syslog".
rgeorgia@kodiakdb /u/h/rgeorgia> sudo psql -U postgres
psql (15beta1)
Type "help" for help.

postgres=# 
```

## Creating a database

```bash
postgres=# create database kodiak_test;
CREATE DATABASE
postgres=# \l
List of databases
    Name     |  Owner   | Encoding | Collate |  Ctype  | ICU Locale | Locale Provider |   Access privileges   
-------------+----------+----------+---------+---------+------------+-----------------+-----------------------
 kodiak_test | postgres | UTF8     | C       | C.UTF-8 |            | libc            | 
 postgres    | postgres | UTF8     | C       | C.UTF-8 |            | libc            | 
 template0   | postgres | UTF8     | C       | C.UTF-8 |            | libc            | =c/postgres          +
             |          |          |         |         |            |                 | postgres=CTc/postgres
 template1   | postgres | UTF8     | C       | C.UTF-8 |            | libc            | =c/postgres          +
             |          |          |         |         |            |                 | postgres=CTc/postgres
(4 rows)

postgres=# create user kodiak with encrypted password 'kodiakKid';

postgres=# grant all privileges on database kodiak_test to kodiak;
GRANT

```

## Database Setup

### Database

- Database **dbCyberSecurity**

### Tables

1. tblSub_UFC_Table_H1
1. tblSub_UFC_Table_H2
1. tblSub_UFC_Table_H3
1. tblSub_UFC_Table_H4
1. tblSub_UFC_Table_H5
1. tblSub_UFC_Table_H6
1. tblSub_UFC_Table_H7
1. tblMaster_CCI_List
1. tblSub_UFC_Responsibility_Fallbacks
1. tblActive_Jobs
1. tblActive_Systems
1. tblMaster_Changelog
1. tblMaster_Spec_Sections
1. tblMaster_Spec_Sections_Old
1. tblMeta_CIA_Levels
1. tblMeta_Settings
1. tblSaved_Control_Data
1. tblSaved_System_Data
1. tblSub_CCI_Overrides 
1. tblSub_CNSS_1253_Control_CIA_Levels
1. tblSub_CNSS_1253_NIST_Control_CIA_Levels
1. tblSub_Control_Overrides
1. tblSub_Master_Spec_CCI_List
1. tblSub_Master_Spec_CCI_List_BKL
1. tblSub_Master_Spec_CCI_List_DVS
1. tblSub_NIST_800_82_Control_CIA_Levels
1. tblSub_UFC_Table_G
1. tblSub_UFC_Table_H1_Impact_Levels







