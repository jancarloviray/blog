---
layout: post
title: Postgres Quick Start and Best Practices
permalink: blog/postgres-quick-start-and-best-practices/
comments: True
excerpt_separator: <!--more-->
---

Want to add or change something? Feel free to [create a pull request](https://github.com/jancarloviray/jancarloviray.github.io/blob/master/_posts/2016-12-14-postgres-quick-start-and-best-practices.md). I hope this helps, and thanks in advance!

## Create a Postgres Docker Container

Want to test something quick? Install [Docker](https://www.docker.com/) and run these commands!

```shell
# get latest image and create a container
docker pull postgres
docker run --name pg -d postgres

# invoke a shell in the container to enter
docker exec -it pg bash

# now that you're inside the container, get inside postgres
# by switching to "postgres" user and running `psql`
su - postgres -c psql

# enjoy!
```

## Installation

### Install Postgres (latest, [9.6](https://www.postgresql.org/docs/9.6/static/release-9-6.html))

This is for Ubuntu/Debian distribution. For other versions, read [this](https://www.postgresql.org/download).

```shell
# update system and get some common tools
apt-get update
apt-get install -y software-properties-common wget sudo

# add repo based on Ubuntu version - `cat /etc/lsb_release`
sudo add-apt-repository "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main"

# get keys
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# update again to include the added repository
sudo apt-get update

# install core, client, contrib and start cluster
sudo apt-get install postgresql-9.6

# start the postgres server - this is important
service postgresql start

# autostart postgres on boot (for Ubuntu 15+)
systemctl enable postgresql
```

Note that Postgres is set up to use **peer** auth by default, which associates roles with a matching Unix account. This is why you need to login as a specific user before you can `psql`. Check out **/etc/postgresql/9.6/main/pg_hba.conf** to change this behavior.

The installation also created a system user called **postgres**, which is associated with a default role. Check out **/etc/passwd**.

```shell
# signin to "postgres" system account created by installer
sudo su - postgres

# invoke postgres console, which without arguments is equivalent to:
# psql -U current_sys_user -d current_sys_user_as_db_name
psql
```

## Quick Intro to Configuration

Let's enter through the starting point and work our way in. Make sure postgres is running. If not, run `service postgresql start`

```shell
# let's check if the process is running and its arguments
# if you don't see any results, postgres server is not running
ps aux | grep postgres | grep -- -D

# You should see this result:
# /usr/lib/postgresql/9.6/bin/postgres -D /var/lib/postgresql/9.6/main -c config_file=/etc/postgresql/9.6/main/postgresql.conf

# `postgres` starts the server
# `-D` points where the data will live
# `-c` points to the main config file postgresql.conf it will use
```

The main configuration file is **postgresql.conf** or whatever you saw in the results above **config_file=/etc/postgresql/9.6/main/postgresql.conf**. Open that file and check out the section callled "FILE LOCATIONS".

### Main Server Configuration

`config_file = '/etc/postgresql/9.6/main/postgresql.conf'`

This is the main server config where you can tune performance, change connection settings, security and authentication settings, ssl, memory consumption, replication, query planning, error reporting and logging and etc. Some basic settings include:

  - `listen_addresses` what IP addresses to listen on; use '*' to allow all; separate IP by comma.
  - ...

### Client Authentication

`hba_file = '/etc/postgresql/9.6/main/pg_hba.conf'`

This file is stored in the database cluster's data directory. HBA stands for host-based authentication. This is where you set rules on who or what can connect to the server.

#### Authentication Methods

**trust** assumes that anyone who can connect to the server is authorized to access the database. This is appropriate for single-user workstation, but not on multi-user machines.

**password** (cleartext) and **md5** if you want to authenticate by text. Change a user's password with `CREATE USER` or `ALTER ROLE`, e.g., `CREATE USER joe WITH PASSWORD 'secret'`. Note that *if no password has been setup for a user, the stored password is null and authentication will always fail*

**peer** works by obtaining the client's OS system user name from the kernel and uses it as the allowed database user name

Check out the official documentation for more.

### User Name Mapping

`ident_file = '/etc/postgresql/9.6/main/pg_ident.conf'`

This maps external user names to their corresponding PostgreSQL user names. General form of setting is: *mapname sys-name pg-name*. To use user name mapping, change `map=map-name` setting in **pg_hba.conf**. Here are some examples and scenarios of mapping:

```
# MAPNAME   SYSTEM-USERNAME   PG-USERNAME
mymap       brian             brian
mymap       jane              jane
# "rob" has postgres role "bob"
mymap       rob               bob
# "brian" can use roles "bob" and "guest1"
mymap       brian             bob
mymap       brian             guest1
```

### Misc

`external_pid_file = '/var/run/postgresql/9.6-main.pid'` - path to additional PID
`data_directory = '/var/lib/postgresql/9.6/main'` - data storage location

### Remove Default Authentication (NOT recommended)

If you don't want to deal with authentication, you can change the settings in **pg_hba.conf** file by changing **peer** to **trust**. These commands will conveniently do it for you:

```shell
# search-replace the methods
sed -i 's/local.*all.*postgres.*peer/local all postgres trust/' /etc/postgresql/9.6/main/pg_hba.conf
sed -i 's/local.*all.*all.*peer/local all all trust/' /etc/postgresql/9.6/main/pg_hba.conf

# reload config
service postgresql restart

# now you can log in as any user
psql -U postgres -d postgres
```

## Quick Start and Overview

### Nice Helpers inside `psql`

- `\?` help
- `\c other-db` connect to another db
- `\l+ *optional-pattern*` list databases
- `\dn+ *optional-pattern*` list schemas
- `\dt+ *optional-pattern*` list tables
- `\df *optional-pattern*` find functions
- `\e` to invoke your `$EDITOR` and use a real editor
- `\q` to quit

Add these settings to your `~/.psqlrc` file:

- `\set COMP_KEYWORD_CASE upper` to auto-complete keywords in CAPS
- `\pset null ¤` to render NULL as ¤ instead
- `\x [on|off|auto]` for expanded output (default is "off")
- `\timing [on|off]` to toggle timing of commands - great for benchmarks

### Add a System User

```shell
sudo adduser postgres_user
```

### Add a Postgres User

Inside Postgres prompt, create a new Postgres user with the same name as the user we created earlier, "postgres_user".

```shell
sudo su - postgres
psql
```

```sql
CREATE USER postgres_user WITH PASSWORD 'pass';
```

### Create a Database for Postgres User

```sql
CREATE DATABASE my_postgres_db;

-- associate to postgres_user
GRANT ALL ON DATABASE my_postgres_db TO postgres_user;
```

Exit the prompt `\q`. Also exit current shell (associated with "postgres" system account) `exit`

```shell
# Log into the user you created
sudo su - postgres_user

# connect to the database you created
psql my_postgres_db
```

<!--more-->

### Add a Sample Table

```sql
-- create a table
CREATE TABLE pg_equipment (
  equip_id serial PRIMARY KEY,
  type VARCHAR(50) NOT NULL,
  color VARCHAR(25) NOT NULL,
  location VARCHAR(25)
    CHECK (
      location IN ('north', 'south', 'west', 'east')
    ),
  install_date DATE
);

-- add column
ALTER TABLE pg_equipment
ADD COLUMN functioning bool;

-- add a default value to column
ALTER TABLE pg_equipment
ALTER COLUMN functioning SET DEFAULT 'true';

-- set column to not null
ALTER TABLE pg_equipment
ALTER COLUMN functioning SET NOT NULL;

-- rename column
ALTER TABLE pg_equipment
RENAME COLUMN functioning TO working_order;

-- remove column
ALTER TABLE pg_equipment
DROP COLUMN working_order;

-- rename entire table
ALTER TABLE pg_equipment
RENAME TO playground_equip;

-- drop table
DROP TABLE IF EXISTS playground_equip;
```

Exit the postgres prompt `\q`

Also exit the shell associated with "postgres_user" `exit`. This should bring you back to root user.

### Import a sample database

```shell
# log into default "postgres" user
sudo su - postgres

# Download sample database. If you don't have wget, run `apt-get update` and `apt-get install`
wget http://pgfoundry.org/frs/download.php/527/world-1.0.tar.gz

# extract archive and change to content directory
tar xzvf world-1.0.tar.gz
cd dbsamples-0.1/world

# create database to import the file structure
createdb -T template0 worlddb

# import sql
psql worlddb < world.sql

# log into database
psql worlddb
```

### Query the sample database

```sql
-- `\dt+` to see list of tables in this database
-- `\d city` to see columns, constraints, indexes, etc

-- select
SELECT * FROM city;
SELECT name,continent FROM country;

-- order by
SELECT name,continent FROM country ORDER BY continent, name;

-- filter
SELECT name FROM city WHERE countrycode = 'USA';
SELECT name FROM city WHERE countrycode = 'USA' AND name LIKE 'N%';
SELECT name FROM city WHERE countrycode = 'USA' AND name LIKE 'N%' ORDER BY name;

-- join
SELECT
  country.NAME AS country,
  city.NAME AS capital,
  continent
FROM country
JOIN city
  ON country.capital = city.id
ORDER BY continent, country;
```

### Let's work with JSON

```sql
-- create table with a json column
CREATE TABLE products (
  id serial PRIMARY KEY,
  name varchar,
  attributes JSONB
);

-- insert some data
INSERT INTO products (name, attributes) VALUES (
 'Geek Love: A Novel', '{
    "author": "Katherine Dunn",
    "pages": 368,
    "category": "fiction"}'
 );

-- create an index
CREATE INDEX idx_products_attributes ON products USING GIN(attributes);

-- query an attribute
SELECT attributes->'category' FROM products;

-- extract query as text
SELECT attributes->>'category' FROM products;
```

## Deeper Dive on Roles / Users

### List Roles

```
\du
```

### Create a User and a Role

```sql
-- create role but don't give a password
CREATE ROLE jonathan LOGIN;

-- create role with a password
-- CREATE USER is same as CREATE ROLE except it implies LOGIN
CREATE USER someuser WITH PASSWORD 'pass';

-- create role that can create databases and manage roles
CREATE ROLE role_name WITH optional_permissions;
CREATE ROLE admin WITH CREATEDB CREATEROLE;
```

### Change a User's Password

```sql
ALTER ROLE [role_name] WITH PASSWORD '[new_password]';
```

### Drop Role

```sql
DROP ROLE IF EXISTS role_name;
```

### Define and Change Provileges

Use `\h CREATE ROLE` to check attributes

```sql
-- ALTER ROLE role_name WITH attribute_options;
ALTER ROLE demo_role WITH NOLOGIN;
ALTER ROLE demo_role WITH LOGIN;
```

## Deeper Dive on Database

### Connect to a Database

```shell
psql -d postgres
```

- `\l` to list all databases
- `\conninfo` to get info on current db
- `c dbname` to connect to a different db

### Create Database

```sql
-- CREATE DATABASE [name] OWNER [role_name];

CREATE USER postgres_user WITH PASSWORD 'password';
CREATE DATABASE my_postgres_db OWNER postgres_user;
DROP DATABASE IF EXISTS my_postgres_db;
```

## Deeper Dive on Table

### List Tables

```
\dt
```

### Create Table

Basic Syntax

```sql
CREATE TABLE table_name (
    column_name1 col_type (field_length) column_constraints,
    column_name2 col_type (field_length),
    column_name3 col_type (field_length)
);
```

```sql
CREATE TABLE playground (
  equip_id serial PRIMARY KEY,
  type varchar(50) NOT NULL,
  color varchar(25) NOT NULL,
  location varchar(25)
  CHECK (location IN ('north', 'south', 'west', 'east', 'northeast',
  'southeast', 'southwest', 'northwest')),
  install_date date
);
```

### Check Table

```
\d
```

### Insert into Table

```sql
INSERT INTO playground (type, color, location, install_date)
VALUES ('slide', 'blue', 'south', '2014-04-28');
```

### Drop Table

```sql
-- drop table
DROP TABLE IF EXISTS mytable

-- drop table and dependencies
DROP TABLE table_name CASCADE;

-- just delete all rows from table
DELETE FROM mytable;
```

## Deeper Dive on Columns

### Modifying Table Columns

```sql
-- Add Column to Table
ALTER TABLE table_name
ADD COLUMN column_name data_type;

-- Remove Column from Table
ALTER TABLE table_name
DROP COLUMN column_name;

-- Change Column Data Type
ALTER TABLE table_name
ALTER COLUMN column_name data_type;

-- Change Column Name
ALTER TABLE table_name
RENAME COLUMN column_name TO new_name;

-- Set Default Value for Existing Column
ALTER TABLE table_name
ALTER_COLUMN created_at SET DEFAULT now();

-- Add UNIQUE constrain to Existing Column
ALTER TABLE table_name
ADD UNIQUE (column_name);
```

## General SQL

```sql
CREATE TABLE weather (
    city            varchar(80),
    temp_lo         int,
    temp_hi         int,
    prcp            real,
    date            date
);

-- basic insert
INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');

-- explicit columns
INSERT INTO weather (city, temp_lo, temp_hi, prcp, date)
VALUES ('San Francisco', 43, 57, 0.0, '1994-11-29');

-- unordered columns
INSERT INTO weather (date, city, temp_hi, temp_lo)
VALUES ('1994-11-29', 'Hayward', 54, 37);
```

### Querying

```sql
SELECT * FROM weather;
SELECT city, temp_lo, temp_hi, prcp, date FROM weather;
SELECT * FROM weather WHERE city = 'San Francisco' AND prcp > 0.0;

-- expressions
SELECT city, (temp_hi + temp_lo)/2 AS temp_avg, date FROM weather;
```

### Joins

### Views

Views are virtual tables. It just encapsulates a query to make life easier. Note that it does not actually duplicate or persist data.

```sql
CREATE OR REPLACE VIEW employee_view AS
SELECT
  employees.last_name,
  employees.salary,
  departments.department
FROM
  employees,
  employee_departments,
  departments
WHERE
  employees.id = employee_departments.employee_id
  AND departments.id = employee_departments.department_id

-- query the view
SELECT *
FROM employee_view
```

## Data Types

### Basic Types and Best Practices

#### Surrogate Keys

Use `uuid` for primary key

```sql
CREATE EXTENSION pgcrypto;
SELECT gen_random_uuid();
```

#### Text

Use `text` and avoid `varchar` or `char` and especially `varchar(n)` unless you specifically want to have a hard limit. Read [this](http://stackoverflow.com/questions/4848964/postgresql-difference-between-text-and-varchar-character-varying). Performance wise, `text` is faster.

Use indexes for pattern matching.

```sql
CREATE INDEX ON users (name);
SELECT * FROM accounts WHERE email LIKE 'Peter%';
```

#### Dates and Times

Always use `timestamptz` instead of `time`.

Use `date` when you just need the date.

#### Boolean

Use `bool` and not `bit`.

#### Numbers

- Avoid `money` since it's not up to standards
- Use `numeric` instead of `float` or `integer`

#### Arrays

Make an array

```sql
SELECT ARRAY[1, 2, 3];

-- another way
SELECT '{1, 2, 3}'::numeric[];
```

Extend an Array

```sql
SELECT ARRAY[1,2] || 3;
SELECT ARRAY[1,2] || ARRAY[3, 4];
```

Access an Array (SQL arrays are 1-indexed instead of the typical 0-index arrays!)

```sql
SELECT ('{one, two, three}'::text[])[1]; -- one
```

#### Enum

Are fast, transparent mapping of words to integer and lives in `pg_enum`. Use this as labels, otherwise, it's similar to other languages.

```sql
CREATE TYPE weekdays AS ('Mon', 'Tue', 'Wed', 'Thu', 'Fri');

-- Enum Example
CREATE TYPE server_states AS ENUM ('running', 'offline', 'restarting');
CREATE TABLE enum_test(id serial, state server_states);
INSERT INTO enum_test(state) VALUES ('offline');

-- Example of Bad Insert
-- ERROR:  invalid input value for enum server_states: "destroyed"
INSERT INTO enum_test(state) VALUES ('destroyed');

-- You Can Add New Values
ALTER TYPE server_states ADD VALUE 'destroyed' AFTER 'offline';
INSERT INTO enum_test(state) VALUES ('destroyed');
```

## Backup and Export

### Export as CSV

```sql
COPY (SELECT * FROM widgets) TO '/absolute/path/to/export.csv'
WITH FORMAT csv, HEADER true;
```

### Backup All Databases

```shell
# backup all
pg_dumpall > backup_file

# restore all
psql -f backup_file postgres
```

### Backup a Database

```shell
# backup a database
sudo su - postgres
pg_dump postgres > postgres_db.bak

# backup a database remotely
pg_dump -U user_name -h remote_host -p remote_port name_of_database > name_of_backup_file
```

### Restore a Database

```shell
# create a restored database
# psql empty_database < backup_file
createdb -T template0 restored_database
psql restored_database < database.bak

# restore partially but stop on error
psql --set ON_ERROR_STOP=on restored_database < backup_file

# restore, but on error, roll-back; real-world use case
psql --single-transaction restored_database < backup_file
```

### Create Binary Database Dump

```shell
pg_dump -U [role_name] [db_name] -Fc > backup.dump

# if you want to convert it to sql file
pg_restore binary_file.backup > sql_file.sql
```

### Create Schema Only Dump

```shell
pg_dump -U [role_name] [db_name] -s > schema.sql
```

## Indexes

### Create Indexes

```sql
CREATE INDEX idx_salary
ON employees(salary);

CREATE INDEX idx_salary
ON employees(last_name, salary);
```

### Create Indexes Concurrently

```sql
# this prevents locking your table
CREATE INDEX CONCURRENTLY idx_salary
ON employees(last_name, salary);
```

## Monitoring and Logging

### Get Total Number of Connections

```sql
SELECT count(*) FROM pg_stat_activity;

-- break down connections by state
SELECT state, count(*) FROM pg_stat_activity GROUP BY state;
```

### Measure Size

```sql
-- measure database size
SELECT pg_size_pretty(pg_database_size('learning'));

-- measure table size
SELECT pg_size_pretty(pg_relation_size('users'));

-- measure index size
SELECT pg_size_pretty(pg_relation_size('users_pkey'));

-- measure table size along with indexes
SELECT pg_size_pretty(pg_total_relation_size('users'));
```

## Security

...to be continued

## Tips and Techniques

### Write `where` before doing a `delete` or `update`

Before you run your code and most especially during deletes and updates, make sure you take a deep breath and do another scan. This will save you from pain and misery.

### Get list of databases and their users

`\l`

### Compare Outputs

```
\o a.txt
EXPLAIN SELECT * FROM users WHERE id IN (SELECT user_id FROM groups WHERE name = 'admins');
\o b.txt
EXPLAIN SELECT users.* FROM users LEFT JOIN groups WHERE groups.name = 'admins';
\! vimdiff a.txt b.txt
```

## Dos and Donts

- Do not store images in your database. Instead, upload to a storage service like S3, then store image URL in your database as a text field.
- Pagination is tough, but the worst technique is doing an `ORDER BY`, followed by a `LIMIT` and an `OFFSET` since this does not scale.
- Do not use integers as primary keys since this does not scale and you'll eventually run out. Use UUIDs instead. `create extension "uuid-ossp"; select uuid_generate_v4();`
- If you add a column with a default value on an existing table, this will trigger a full re-write of your table. Instead, it's better to allow null values at first so the operation is instant, then set your default, and then, with a background process go and retroactively update the data. When generating new schemas and you know that you should set a default value, go for it.
- Instead of over-normalization, think about whether you can make a column an array of a type enum, like in a categories column in a post table.
