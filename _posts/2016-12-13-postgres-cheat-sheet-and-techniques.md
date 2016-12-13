---
layout: post
title: Postgres Cheat Sheet and Techniques
permalink: blog/postgres-cheat-sheet-and-techniques/
comments: True
excerpt_separator: <!--more-->
---

## PostgreSQL Installation and Configuration

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

<!--more-->

## Table

### List Tables

```
\dt
```

### Create Table

```
CREATE TABLE mytable (
  id BIGINT PRIMARY KEY,
  name VARCHAR(20),
  price INT,
  created_at timestamp without time zone default now()
)
```

### Insert into Table

```
INSERT INTO mytable VALUES(1,'widget1',100)
INSERT INTO mytable(name, price) VALUES ('widget2', 101)
```

### Drop Table

```
DROP TABLE IF EXISTS mytable
```

### Delete all Rows from Table

```
DELETE FROM mytable;
```

### Drop Table and Dependencies

```
DROP TABLE table_name CASCADE;
```

## Column

### Create Enum Type

```
CREATE TYPE environment AS ENUM ('development', 'staging', 'production');
```

### Add Column to Table

```
ALTER TABLE [table_name] ADD COLUMN [column_name] [data_type];
```

### Remove Column from Table

```
ALTER TABLE [table_name] DROP COLUMN [column_name];
```

### Change Column Data Type

```
ALTER TABLE [table_name] ALTER COLUMN [column_name] [data_type];
```

### Change Column Name

```
ALTER TABLE [table_name] RENAME COLUMN [column_name] TO [new_column_name];
```

### Set Default Value for Existing Column

```
ALTER TABLE [table_name] ALTER_COLUMN created_at SET DEFAULT now();
```

### Add UNIQUE constrain to Existing Column

```
ALTER TABLE [table_name] ADD UNIQUE ([column_name]);
```

## Tips and Techniques

### Render NULL visible in psql

```
\pset null ¤
```

### Extended Display Mode in Auto

```
\x auto
```

### Write `where` before doing a `delete` or `update`

Before you run your code and most especially during deletes and updates, make sure you take a deep breath and do another scan. This will save you from pain and misery.

### Get list of databases and their users

`\l`
