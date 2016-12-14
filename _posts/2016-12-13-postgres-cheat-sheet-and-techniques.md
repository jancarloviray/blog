---
layout: post
title: Postgres Cheat Sheet and Techniques
permalink: blog/postgres-cheat-sheet-and-techniques/
comments: True
excerpt_separator: <!--more-->
---

This post is continually in updated. Modify it by editing it [here](https://github.com/jancarloviray/jancarloviray.github.io/edit/master/_posts/2016-12-13-postgres-cheat-sheet-and-techniques.md). Thanks in advance and I hope this helps you!

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

Compare Outputs

```
\o a.txt
EXPLAIN SELECT * FROM users WHERE id IN (SELECT user_id FROM groups WHERE name = 'admins');
\o b.txt
EXPLAIN SELECT users.* FROM users LEFT JOIN groups WHERE groups.name = 'admins';
\! vimdiff a.txt b.txt
```

## PostgreSQL Installation and Configuration

### Install Postgres

```shell
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
```

On installation, Postgres is set up to use "ident" authentication, meaning it associates roles with a matching Unix system account. If a role exists, it can be signing in by logging into the associated Linux system account. The installation created a user account called "postgres" that is associated with the default Postgres role. To log into that account, run `sudo -i -u postgres`. You will then get a shell as a "postgres" user. Get into Postgres by typing `psql`

## Roles (Unix-style Users)

### Create a User and a Role

```sql
-- create role but don't give a password
CREATE ROLE jonathan LOGIN;
-- create role with a password (CREATE USER is the same as CREATE ROLE except that it implies LOGIN)
CREATE USER davide WITH PASSWORD 'jw8s0F4';
-- create role that can create databases and manage roles
CREATE ROLE admin WITH CREATEDB CREATEROLE;
```

### Change a User's Password

```sql
ALTER ROLE [role_name] WITH PASSWORD '[new_password]';
```

### Allow User to Create Databases

```sql
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

<!--more-->

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
CREATE TABLE mytable (
  id BIGINT PRIMARY KEY,
  name VARCHAR(20),
  price INT,
  created_at timestamp without time zone default now()
)

CREATE TABLE playground (
    equip_id serial PRIMARY KEY,
    type varchar (50) NOT NULL,
    color varchar (25) NOT NULL,
    location varchar(25) check (location in ('north', 'south', 'west', 'east', 'northeast', 'southeast', 'southwest', 'northwest')),
    install_date date
);
```

### Check Table

```
\d
```

### Insert into Table

```sql
INSERT INTO playground (type, color, location, install_date) VALUES ('slide', 'blue', 'south', '2014-04-28');
INSERT INTO playground (type, color, location, install_date) VALUES ('swing', 'yellow', 'northwest', '2010-08-16');
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
CREATE TYPE environment AS ENUM ('development', 'staging', 'production');
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
ALTER TABLE [table_name] RENAME COLUMN [column_name] TO [new_column_name];
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
CREATE TYPE server_states AS ENUM ('running', 'uncertain', 'offline', 'restarting');
CREATE TABLE enum_test(id serial, state server_states);
INSERT INTO enum_test(state) VALUES ('offline');

-- Example of Bad Insert
INSERT INTO enum_test(state) VALUES ('destroyed'); -- ERROR:  invalid input value for enum server_states: "destroyed"

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
INSERT INTO cards VALUES (1, 1, '{"name": "Paint house", "tags": ["Improvements", "Office"], "finished": true}');
INSERT INTO cards VALUES (2, 1, '{"name": "Wash dishes", "tags": ["Clean", "Kitchen"], "finished": false}');
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
pg_dump -Fc
```

### Create Database Dump (binary)

```shell
pg_dump -U [role_name] [db_name] -Fc > backup.dump
```

### Convert Binary Dump to SQL file

```shell
pg_restore binary_file.backup > sql_file.sql
```

### Create Schema Only Dump (sql)

```shell
pg_dump -U [role_name] [db_name] -s > schema.sql
```

### Restore Database From a Dump

```shell
PGPASSWORD=<password> pg_restore -Fc --no-acl --no-owner -U <user> -d <database> <filename.dump>
```

### Copy Database

```shell
createdb -T app_db app_db_backup
```

### Change Database Ownership

```sql
ALTER DATABASE mydb OWNER TO jancarlo;
```

## Monitoring and Logging

### Get Total Number of Connections

```sql
SELECT count(*) FROM pg_stat_activity;

-- break down connections by state
SELECT state, count(*) FROM pg_stat_activity GROUP BY state;
```

## Tips and Techniques

### Write `where` before doing a `delete` or `update`

Before you run your code and most especially during deletes and updates, make sure you take a deep breath and do another scan. This will save you from pain and misery.

### Get list of databases and their users

`\l`
