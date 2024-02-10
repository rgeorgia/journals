---
title: "Mongodb"
date: 2021-12-30T15:12:47-05:00
draft: true
categories: [notes]
tags: [database, system]
---


## Mongodb Notes

*references*:
[Enable MongoDB Authentication](https://devops.ionos.com/tutorials/enable-mongodb-authentication/)

### Create Administrative User

An administrative user is added to the MongoDB "admin" database. Use the mongo client to connect to MongoDB.
```bash
mongo
```

Connect to the "admin" database and create the administrative user.

```javascript
use admin
db.createUser(
      {
          user: "username",
          pwd: "password",
          roles: [ "root" ]
      }
  )
```

The new administrative user can now be used to log into a MongoDB database.

```bash
mongo -u username -p password --authenticationDatabase admin mydatabase
```

It is worth noting that the -p parameter without a following argument will prompt for the password. The authentication process will fail if the following argument is a database name as the command will interpret the argument as the password. The -p parameter must be followed by the password argument or another hyphenated parameter.

```bash
mongo -u username -p password admin
```

### Create Database User

A database user is added to the specific database requiring authentication. The following commands will create a user in the database named "mydatabase" with read-write privileges.

```javascript
use mydatabase
db.createUser(
    {
        user: "username",
        pwd: "password",
        roles: [
            { role: "readWrite", db: "mydatabase" }
        ]
    }
)
```

The database can then be accessed with the new user.

```bash
$ mongo -u username -p password --authenticationDatabase mydatabase
```

The --authenticationDatabase parameter is unnecessary if the database being accessed at the command line contains the user.
```bash
$ mongo -u username -p password --host localhost mydatabase
```

The --host parameter is used to prevent "mydatabase" from being interpreted as the password argument.
