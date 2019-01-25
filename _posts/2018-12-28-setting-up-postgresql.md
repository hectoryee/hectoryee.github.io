---
title: Setting Up PostgreSQL
desc: Building a database from scratch.
published: true
date: 2018-12-28
categories: project
tags: intern postgres database notebook
---
To install PostgreSQL follow this [tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04).

### Installing
``` sh
sudo apt update
sudo apt install postgresql postgresql-contrib
```

### Using Postgres
Switch to postgres (default) account on your server type:
``` sh
sudo -i -u postgres
```
Access PostgreSQL prompt:
``` sh
psql
```
Exit interactive Postgres session:
```
\q
```

### New database
Create a new database:
``` sh
sudo -i -u postgres
createdb database-name
```

### Changing user and database
Add user:
``` sh
sudo adduser user-name
```
Switch user and connect to the user database (after creating one with name same as user-name):
``` sh
sudo -i -u user-name
psql
```
Changing it altogether:
```
psql my_database -U postgres
```
Check current connection info:
```
\conninfo
```

### Viewing table and database
List database:
```
\l
```
List table:
```
\dt
\dt *.*
```
List table in a public schema:
```
\dt public.*
```
List schema:
```
\dn
```

### Dumping database for backup
```
pg_dump dbname > \path\outfile
```

## Extension used:
```
CREATE EXTENSION fuzzystrmatch;
CREATE EXTENSION pg_trgm;
```

Refer [rosette](https://www.rosette.com/blog/overview-fuzzy-name-matching-techniques/) and [wikiversity](https://en.wikiversity.org/wiki/Duplicate_record_detection) for more info.


