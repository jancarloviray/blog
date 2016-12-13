---
layout: post
title: Postgres Cheat Sheet and Techniques
permalink: blog/postgres-cheat-sheet-and-techniques/
comments: True
excerpt_separator: <!--more-->
---

## Installation and Configuration

### Add APT repository

```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
```

### Install Postgres

```
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib postgresql-client libpq-dev
```

### Set a Password

```
sudo -u postgres psql
\password
```

### Create a User and a Role

```
CREATE ROLE [role_name] WITH LOGIN CREATEDB PASSWORD '[password]';
```

### Change a User's Password

```
ALTER ROLE [role_name] WITH PASSWORD '[new_password]';
```

### Allow User to Create Databases

```
ALTER USER <username> WITH CREATEDB;
```

### List Roles

```
\du
```

## Database

### List all Databses

```
\l
```

### Create Database

```
CREATE DATABASE [name] OWNER [role_name];
```

### Drop Database

```
DROP DATABASE IF EXISTS [name];
```

### Export as CSV

```
COPY (SELECT * FROM widgets) TO '/absolute/path/to/export.csv'
WITH FORMAT csv, HEADER true;
```

### Backup All Databases

```
pg_dump -Fc
```

### Create Database Dump (binary)

```
pg_dump -U [role_name] [db_name] -Fc > backup.dump
```

### Convert Binary Dump to SQL file

```
pg_restore binary_file.backup > sql_file.sql
```

### Create Schema Only Dump (sql)

```
pg_dump -U [role_name] [db_name] -s > schema.sql
```

### Restore Database From a Dump

```
PGPASSWORD=<password> pg_restore -Fc --no-acl --no-owner -U <user> -d <database> <filename.dump>
```

### Copy Database

```
createdb -T app_db app_db_backup
```

### Change Database Ownership

```
ALTER DATABASE mydb OWNER TO jancarlo;
```

## Tips and Techniques

### Write `where` before doing a `delete` or `update`

Before you run your code and most especially during deletes and updates, make sure you take a deep breath and do another scan. This will save you from pain and misery.

### Get list of databases and their users

`\l`

<!--more-->

Still in development...
