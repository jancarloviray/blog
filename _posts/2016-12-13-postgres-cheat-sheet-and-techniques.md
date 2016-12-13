---
layout: post
title: Postgres Cheat Sheet and Techniques
permalink: blog/postgres-cheat-sheet-and-techniques/
comments: True
excerpt_separator: <!--more-->
---

Please note that this post will continually be in development as a collection of tips, common commands and best practices. Feel free to [create a pull request](https://github.com/jancarloviray/jancarloviray.github.io) if you'd like to add or improve this post. Thanks in advance and I hope this helps you!

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
db=# \o a.txt
db=# EXPLAIN SELECT * FROM users WHERE id IN (SELECT user_id FROM groups WHERE name = 'admins');
db=# \o b.txt
db=# EXPLAIN SELECT users.* FROM users LEFT JOIN groups WHERE groups.name = 'admins';
db=# \! vimdiff a.txt b.txt
```

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

## User

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

<!--more-->

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

## Data Types

### Basic Types and Best Practices

#### Surrogate Keys

Use `uuid` for primary key

```
CREATE EXTENSION pgcrypto;
SELECT gen_random_uuid();
```

#### Text

Use `text` and avoid `varchar` or `char` and especially `varchar(n)` unless you specifically want to have a hard limit. Read [this](http://stackoverflow.com/questions/4848964/postgresql-difference-between-text-and-varchar-character-varying). Performance wise, `text` is faster.

Use indexes for pattern matching. 

```
CREATE INDEX ON users (name);
SELECT * FROM accounts WHERE email LIKE 'Peter%';
```

For suffix lookups, use functional indexes.

```
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

```
SELECT ARRAY[1, 2, 3];

-- another way
SELECT '{1, 2, 3}'::numeric[];
```

Extend an Array

```
SELECT ARRAY[1,2] || 3;
SELECT ARRAY[1,2] || ARRAY[3, 4];
```

Access an Array (SQL arrays are 1-indexed instead of the typical 0-index arrays!)

```
SELECT ('{one, two, three}'::text[])[1]; -- one
```

#### Enum

Are fast, transparent mapping of words to integer and lives in `pg_enum`. Use this as labels, otherwise, it's similar to other languages.

```
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

```
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

```
CREATE TABLE cards (
  id integer NOT NULL,
  board_id integer NOT NULL,
  data jsonb
);
```

##### Inserting JSON data

```
INSERT INTO cards VALUES (1, 1, '{"name": "Paint house", "tags": ["Improvements", "Office"], "finished": true}');
INSERT INTO cards VALUES (2, 1, '{"name": "Wash dishes", "tags": ["Clean", "Kitchen"], "finished": false}');
```

##### Querying Data

```
SELECT data->>'name' AS name FROM cards
```

##### Filtering Results

```
SELECT * FROM cards WHERE data->>'finished' = 'true';
```

##### Checking for Column Existence

```
SELECT count(*) FROM cards WHERE data ? 'ingredients';
```

##### Expanding Data into Rows

```
SELECT
  jsonb_array_elements_text(data->'tags') as tag
FROM cards
WHERE id = 1;
```

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

## Monitoring and Logging

### Get Total Number of Connections

```
SELECT count(*) FROM pg_stat_activity;

-- break down connections by state
SELECT state, count(*) FROM pg_stat_activity GROUP BY state;
```

## Tips and Techniques

### Write `where` before doing a `delete` or `update`

Before you run your code and most especially during deletes and updates, make sure you take a deep breath and do another scan. This will save you from pain and misery.

### Get list of databases and their users

`\l`
