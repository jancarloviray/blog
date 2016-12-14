---
layout: post
title: Postgres Cheat Sheet, Tips and Summary
permalink: blog/postgres-cheat-sheet-tips-and-summary/
comments: True
excerpt_separator: <!--more-->
---

This post is continually updated. Modify it by editing [this](https://github.com/jancarloviray/jancarloviray.github.io/edit/master/_posts/2016-12-14-postgres-cheat-sheet-tips-and-summary.md). Thanks in advance and I hope this helps!

## Create a Postgres Container

```shell
docker pull postgres
docker run --name pg -d postgres
docker exec -it pg bash

# inside container
su - postgres
psql
```

## Productivity Tips inside `psql`

- `\?` shows help with psql commands
- `\c other-db` to connect to another db without quitting the console
- `\l+` to list databases
- `\dn+` to list schemas
- `\dt+` to list tables
- `\dt+ *.users` to list tables with pattern `*.users`
- `\df *somefuncname*` to find functions with pattern `*somefuncname*`
- `\e` to invoke your `$EDITOR` and use a real editor
- `\q` to quit

Add these to your `~/.psqlrc` file!

- `\set COMP_KEYWORD_CASE upper` to auto-complete keywords in CAPS
- `\pset null ¤` to render NULL as ¤ instead
- `\x [on|off|auto]` for expanded output (default is "off")
- `\timing [on|off]` to toggle timing of commands - great for benchmarks

## PostgreSQL Installation and Configuration

### Install Postgres

```shell
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
```

On installation, Postgres is set up to use **ident** auth. This associates roles with a matching Unix account. The installation created a system user called **postgres** that is associated with the default Postgres role. Log into that account by running `sudo -i -u postgres`. You will then get a shell as a "postgres" user. Get inside Postgres with `psql`. By default, `psql` alone runs `psql -U [current_unix_username] -d [current_unix_username_as_db_name]`.

By default, users are only allowed to login locally if the system username matches the PostgreSQL username. *PostgreSQL assumes that when you log in, you will be using a username that matches your operating system username, and that you will be connecting to a database with the same name as well.*

To bypass the default behavior, run: `psql -U some_user -d some_db_name -h 127.0.0.1`

### Quick Start and Overview

#### Add a System User

```shell
# create a new Unix system user
sudo adduser postgres_user
# log onto the default postgres user created by default
sudo su - postgres
# go inside postgres prompt
psql
```

Inside Postgres prompt, create a new Postgres user with the same name as the user we created earlier, "postgres_user".

#### Add a Postgres User

```sql
CREATE USER postgres_user WITH PASSWORD 'password';
```

#### Create a Database for Postgres User

```sql
-- create database
CREATE DATABASE my_postgres_db;
-- associate created database to postgres_user
GRANT ALL PRIVILEGES ON DATABASE my_postgres_db TO postgres_user;
```

Exit the prompt `\q`.

Also exit current shell (associated with "postgres" system account) `exit`

```shell
# Log into the user you created
sudo su - postgres_user
# Sign into the database you created
psql my_postgres_db
```

#### Let's add a Sample Table

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
ALTER TABLE pg_equipment ADD COLUMN functioning bool;
-- add a default value to column
ALTER TABLE pg_equipment ALTER COLUMN functioning SET DEFAULT 'true';
-- set column to not null
ALTER TABLE pg_equipment ALTER COLUMN functioning SET NOT NULL;

-- rename column
ALTER TABLE pg_equipment RENAME COLUMN functioning TO working_order;
-- remove column
ALTER TABLE pg_equipment DROP COLUMN working_order;

-- rename entire table
ALTER TABLE pg_equipment RENAME TO playground_equip;
-- drop table
DROP TABLE IF EXISTS playground_equip;
```

Exit the postgres prompt `\q`

Also exit the shell associated with "postgres_user" `exit`. This should bring you back to root user.

#### Let's now import a sample database

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

#### Let's query the sample database

```sql
-- `\dt+` to see list of tables in this database
-- `\d city` to see column that make up the city table and see information such as "check constraints", "indexes", and "foreign-key constaints"

-- select
SELECT * FROM city;
SELECT name,continent FROM country;

-- order by
SELECT name,continent FROM country ORDER BY continent;
SELECT name,continent FROM country ORDER BY continent,name;

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

#### Let's work with JSON

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

<!--more-->

## Roles (Unix-style Users)

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

```sql
-- `\h CREATE ROLE` to check attributes
ALTER ROLE role_name WITH attribute_options;

ALTER ROLE demo_role WITH NOLOGIN;
ALTER ROLE demo_role WITH LOGIN;
```

## Database

### List all Databses

```
\l
```

### Connect to a Database

```shell
psql -d postgres
```

### Get Information on Current Database

```
\conninfo
```

### Create Database

```sql
CREATE DATABASE [name] OWNER [role_name];

CREATE USER postgres_user WITH PASSWORD 'password';
CREATE DATABASE my_postgres_db OWNER postgres_user;
```

### Drop Database

```sql
DROP DATABASE IF EXISTS [name];
```

## Table

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
DROP TABLE IF EXISTS mytable
```

### Delete all Rows from Table

```sql
DELETE FROM mytable;
```

### Drop Table AND Dependencies

```sql
DROP TABLE table_name CASCADE;
```

## Table Columns

### Create Enum Type

```sql
CREATE TYPE environment
AS ENUM ('development', 'staging', 'production');
```

### Add Column to Table

```sql
ALTER TABLE [table_name] ADD COLUMN [column_name] [data_type];

ALTER TABLE playground ADD last_maint date;
```

### Remove Column from Table

```sql
ALTER TABLE [table_name] DROP COLUMN [column_name];

ALTER TABLE playground DROP last_maint;
```

### Change Column Data Type

```sql
ALTER TABLE [table_name] ALTER COLUMN [column_name] [data_type];
```

### Change Column Name

```sql
ALTER TABLE [table] RENAME COLUMN [column] TO [new_name];
```

### Set Default Value for Existing Column

```sql
ALTER TABLE [table_name] ALTER_COLUMN created_at SET DEFAULT now();
```

### Add UNIQUE constrain to Existing Column

```sql
ALTER TABLE [table_name] ADD UNIQUE ([column_name]);
```

## Rows

### Update Data in a Table

```sql
UPDATE playground SET color = 'red' WHERE type = 'swing';
```

### Delete Data in a Table

```sql
DELETE FROM playground WHERE type = 'slide';
```

## General SQL

### Querying

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

For suffix lookups, use functional indexes.

```sql
CREATE INDEX backsearch ON users (reverse(email));
SELECT * FROM accounts WHERE reverse(email) LIKE reverse('%doe.com');
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

#### JSON

Use `jsonb` which is a binary-encoded version of JSON. This means space padding is gone and is more efficient than `json`. Don't use `json` type.

```sql
CREATE TABLE filmsjsonb ( id BIGSERIAL PRIMARY KEY, data JSONB );

INSERT INTO filmsjsonb (data) VALUES ('{
  "title": "The Shawshank Redemption",
  "num_votes": 1566874,
  "rating": 9.3,
  "year": "1994",
  "type": "feature",
  "can_rate": true,
  "tconst": "tt0111161",
  "image": {
    "url": "http://ia.media-imdb.com/images/M/MV5BODU4MjU4NjIwNl5BMl5BanBnXkFtZTgwMDU2MjEyMDE@._V1_.jpg",
    "width": 933,
    "height": 1388
  }
}');

SELECT * FROM filmsjsonb; -- format is ugly here

SELECT jsonb_pretty(data) FROM filmsjsonb WHERE id=1; -- prettify json format
```

##### Defining Columns

```sql
CREATE TABLE cards (
  id integer NOT NULL,
  board_id integer NOT NULL,
  data jsonb
);
```

##### Inserting JSON data

```sql
INSERT INTO cards
VALUES (1, 1, '{"name": "Paint house", "tags": ["Improvements", "Office"], "finished": true}');
```

##### Querying Data

```sql
SELECT data->>'name' AS name FROM cards
```

##### Filtering Results

```sql
SELECT * FROM cards WHERE data->>'finished' = 'true';
```

##### Checking for Column Existence

```sql
SELECT count(*) FROM cards WHERE data ? 'ingredients';
```

##### Expanding Data into Rows

```sql
SELECT
  jsonb_array_elements_text(data->'tags') as tag
FROM cards
WHERE id = 1;
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

# create a restored database
# psql empty_database < backup_file
createdb -T template0 restored_database
psql restored_database < database.bak

# restore partially but stop on error
psql --set ON_ERROR_STOP=on restored_database < backup_file

# restore, but on error, roll-back
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
CREATE INDEX idx_salary ON employees(salary);

CREATE INDEX idx_salary ON employees(last_name, salary);
```

### Create Indexes Concurrently

```sql
# this prevents locking your table
CREATE INDEX CONCURRENTLY idx_salary ON employees(last_name, salary);
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
