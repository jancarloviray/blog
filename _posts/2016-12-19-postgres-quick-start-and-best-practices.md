---
layout: post
title: Postgres Quick Start and Best Practices
permalink: blog/postgres-quick-start-and-best-practices/
comments: True
excerpt_separator: <!--more-->
---

Want to add or change something? Feel free to [create a pull request](https://github.com/jancarloviray/jancarloviray.github.io/blob/master/_posts/2016-12-19-postgres-quick-start-and-best-practices.md). I hope this helps!

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

# update to include added repo and install postgres
sudo apt-get update
sudo apt-get install postgresql-9.6

# start the postgres server and enable autostart
service postgresql start
systemctl enable postgresql
```

Postgres is set up to **peer** auth by default, associating roles with a matching system acct. This is why you need to login as a specific user before you can `psql`. The installation also created a system user, **postgres**. Check out **/etc/passwd**.

```shell
# signin to "postgres" system acct
sudo su - postgres

# this command by default, is equivalent to:
# psql -U current_system_user \
#      -d current_system_user_as_db_name
psql
```

## Intro to Postgres Configuration

Let's enter through the starting point and work our way in. Make sure postgres is running first with `service postgresql start`

```shell
# check if the process is running
ps aux | grep postgres | grep -- -D

# /usr/lib/postgresql/9.6/bin/postgres -D /var/lib/postgresql/9.6/main -c config_file=/etc/postgresql/9.6/main/postgresql.conf
```

`postgres` starts the server, `-D` points where the data will live, and `-c` points to the main configuration file it will use. The main configuration file is **postgresql.conf**.

Open the file, `less /etc/postgresql/9.6/main/postgresql.conf` and check out the section called "FILE LOCATIONS"

## Server Configuration

```
config_file = '/etc/postgresql/9.6/main/postgresql.conf'
```

The main server config where you can tune: performance, change connection settings, security and authentication settings, ssl, memory consumption, replication, query planning, error reporting and logging and etc.

## Client Authentication

```
hba_file = '/etc/postgresql/9.6/main/pg_hba.conf'
```

This file is stored in the database cluster's data directory. HBA stands for **host-based authentication**. This is where you set rules on who or what can connect to the server.

Fields include: **Connection Type, Database Name, User Name, Address, Authentication Method**

The first record with a matching connection type, client address, requested database, and user name is used to perform authentication.

There is no "fall-through" - if one record is chosen and the authentication fails, subsequent records are not considered. If no record matches, access is denied.

```
local   database    user    auth-method   [auth-opts]
host    database    user    address       auth-method     [auth-opts]
# ...

# allow any user on the local system to connect to any database
local   all         all                   trust

# allow any user from any host with specified ip to connect
# to database "postgres" as the same user name that ident reports
host    postgres    all     192.168.93.0/24   ident

# allow any user from host ip to
# connect to db "postgres" if pass is valid
host    postgres    all     192.168.12.10/32  md5

# allow any user from hosts in the example.com domain if pass is valid
host    all         all     .example.com      md5
```

### Connection Types:

`local` record matches connection attemps using unix-domain sockets, which are inter-process communication on the same host operating system. Without a record of this type, unix-domain socket connections are disallowed.

`host` record matches connection attempts made using TCP/IP. Note that this will also not work if it's not given an appropriate **listen_addressess** configuration parameter since the default for this is only on the local loopback address **localhost**.

Check out documentation for more types.

### Database:

Specifies which db names this record matches; value of `all` specifies that it matches all. `sameuser` specifies if database name is the same as the user.

### User:

Specifies which database user name(s) this record matches. The value `all` specifies that it matches all users.

### Address:

Specifies the client machine address(es) that this record matches.

### Auth Methods:

`trust` assumes that anyone who can connect to the server is authorized to access the database. This is appropriate for single-user workstation, but not on multi-user machines.

`password` (cleartext) and `md5` if you want to authenticate by text. **Note that if no password has been setup for a user, the stored password is null and authentication will always fail**

`peer` works by obtaining the client's OS system user name from the kernel and uses it as the allowed database user name

...

## User Name Mapping

```
ident_file = '/etc/postgresql/9.6/main/pg_ident.conf'
```

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

## Misc

Path to additional PID:

```
external_pid_file = '/var/run/postgresql/9.6-main.pid'
```

Data storage location:

```
data_directory = '/var/lib/postgresql/9.6/main'
```

## Quick Start Overview

### Nice Helpers for `psql`

Add these settings to your `~/.psqlrc` file:

- `\set COMP_KEYWORD_CASE upper` to auto-complete keywords in CAPS
- `\pset null ¤` to render NULL as ¤ instead
- `\x [on|off|auto]` for expanded output (default is "off")
- `\timing [on|off]` to toggle timing of commands - great for benchmarks

### Add a System User

```shell
sudo adduser some_user
```

### Add a Postgres User

Inside Postgres prompt, create a new Postgres user with the same name as the user we created earlier, "some_user".

```shell
sudo su - postgres
psql
```

```sql
CREATE USER some_user WITH PASSWORD 'pass';
```

### Create a Database for Postgres User

```sql
CREATE DATABASE my_postgres_db;

-- associate to some_user
GRANT ALL ON DATABASE my_postgres_db TO some_user;
```

Exit the prompt `\q`.

```shell
# Log into the user you created
sudo su - some_user

# connect to the database you created
psql my_postgres_db
```

### Add a Table

```sql
-- create a table
CREATE TABLE items (
  equip_id serial PRIMARY KEY,
  type VARCHAR(50) NOT NULL,
  color VARCHAR(25) NOT NULL,
  location VARCHAR(25)
    CHECK (
      location IN ('north', 'south', 'west', 'east')
    ),
  install_date DATE
);
```

### Modify a Table

```sql
-- add column
ALTER TABLE items ADD COLUMN functioning bool;

-- alter column
ALTER TABLE items ALTER COLUMN functioning SET DEFAULT 'true';
ALTER TABLE items ALTER COLUMN functioning SET NOT NULL;
ALTER TABLE items RENAME COLUMN functioning TO working_order;

-- remove column
ALTER TABLE items DROP COLUMN working_order;

-- rename entire table
ALTER TABLE items RENAME TO playground_equip;

-- drop table
DROP TABLE IF EXISTS playground_equip;
```

Exit the postgres prompt `\q`

### Import a Database

```shell
# log into default "postgres" user
sudo su - postgres

# Download sample database.
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

### Querying

```sql
-- `\dt+` to see list of tables in this database
-- `\d city` to see columns, constraints, indexes, etc

-- select
SELECT * FROM city;
SELECT name,continent FROM country;
```

#### Order By

```sql
-- order by
SELECT name,continent FROM country ORDER BY continent, name;
```

#### Filtering

```sql
-- filter
SELECT name FROM city WHERE cc = 'USA';
SELECT name FROM city WHERE cc = 'USA' AND name LIKE 'N%';
SELECT name FROM city WHERE cc = 'USA' AND name LIKE 'N%' ORDER BY name;
```

#### Join

```sql
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

<!--more-->

## Database Roles

A role can be thought of either as, a user, a group of users depending how it is set up. Roles can own database objects and have specific privileges. There are no users or groups - just roles. Every connection is made using a particular role.

### Creating Roles

```sql
-- \du: list roles

CREATE ROLE name;

-- create role but don't give a password
CREATE ROLE name LOGIN; -- add login ability
CREATE USER name; -- same as above

-- create role with a password
-- CREATE USER is same as CREATE ROLE except it implies LOGIN
CREATE USER name WITH PASSWORD 'pass';
```

### Altering Roles

```sql
-- \h CREATE ROLE: check available privileges

-- change a user's password
ALTER ROLE name WITH PASSWORD 'new_password';

-- change privileges
ALTER ROLE demo_role WITH NOLOGIN;
ALTER ROLE demo_role WITH LOGIN;

DROP ROLE IF EXISTS role_name;
```

## Database

### Connect to a Database

```shell
psql -d postgres
```

- `\l` to list all databases
- `\conninfo` to get info on current db
- `c dbname` to connect to a different db

### Create Database

```sql
-- create user to own database
CREATE USER some_user WITH PASSWORD 'password';

-- create database
CREATE DATABASE my_postgres_db;

-- associate database to owner
GRANT ALL ON DATABASE my_postgres_db TO some_user;
```

### Delete Database

Get a list of databases with `\l` inside `psql`.

```sql
DROP DATABASE IF EXISTS my_postgres_db;
```

You cannot drop a database that has any open connections, including the one you are in through `psql` or pgAdmin. You must switch to another database or `template1`. It's typically more convenient to use `dropdb` command instead. `dropdb -h localhost -p 5432 -U postgres testdb`

## Tables

### Create Table

List Tables:

```
\dt
```

Syntax:

```sql
CREATE TABLE table_name (
    column_name1 col_type (field_length) column_constraints,
    column_name2 col_type (field_length),
);
```

Example:

```sql
CREATE TABLE playground (
  equip_id      SERIAL PRIMARY KEY,
  type          VARCHAR(50) NOT NULL,
  color         VARCHAR(25) NOT NULL,
  location      VARCHAR(25)
  CHECK (location IN ('north', 'south', 'west', 'east', 'northeast',
  'southeast', 'southwest', 'northwest')),
  install_date  DATE
);
```

Real World:

```sql
CREATE TABLE inherit_base_transaction (
  created_date  TIMESTAMP WITHOUT TIME ZONE default NOW(),
  updated_date  TIMESTAMP WITHOUT TIME ZONE,
  created_by    TEXT default current_user,
  updated_by    TEXT
)

CREATE TABLE cc_inv_order (
  id                  SERIAL NOT NULL PRIMARY KEY,
  acct_id             INT REFERENCES acct(id),
  inv_payment_id  INT REFERENCES inv_payment(id),
  total               NUMERIC(10,2),
  paid                NUMERIC(10,2) DEFAULT 0,
  paid_date           TIMESTAMP WITHOUT TIME ZONE,
  descr               TEXT NOT NULL,
  ref_num    TEXT
) INHERITS(inherit_base_transaction)

CREATE TABLE cc_transaction(
  id                      SERIAL NOT NULL PRIMARY KEY,
  cc_transaction_type_id  INT NOT NULL
                          REFERENCES cc_inv_transaction_type(id),
  cc_session_id           TEXT,
  cc_token_id             INT REFERENCES cc_inv_token(id),
  cc_transaction_id       INT REFERENCES cc_inv_transaction(id),
  cc_order_id             INT NOT NULL
                          REFERENCES cc_inv_order(id),
  amount                  NUMERIC(10,2),
  payload                 TEXT,
  captured_date           TIMESTAMP WITHOUT TIME ZONE,
  notified_date           TIMESTAMP WITHOUT TIME ZONE,
  ref_num                 TEXT,
  job_id                  INT REFERENCES job(id)
) INHERITS(inherit_base_transaction)
```

### Example Usage

Check Table:

```
\d
```

```sql
INSERT INTO playground (type, color, location, install_date)
VALUES ('slide', 'blue', 'south', '2014-04-28');

-- delete rows from table
DELETE FROM mytable;

-- drop table
DROP TABLE IF EXISTS mytable

-- drop table and dependencies
DROP TABLE table_name CASCADE;
```

## Common Types

### Primary Key

#### GUID

Creates globally unique identifiers. If you need offline writes that will synchronize later to an online server, this is your only option. Another use case if if you have tables in multiple databases that must be merged later on. For scalability, this is better choice than serial.

Downsides include performance implications when using Indexes as new inserts cause rewrites instead of just adding to last page when using serials. Also, due to its size, it can potentially add disk and memory overhead.

This should be a safe default. Read [this article](https://www.clever-cloud.com/blog/engineering/2015/05/20/why-auto-increment-is-a-terrible-idea/) on why. Ways you can generate UUID:

**Within Application Code**

```javascript
// npm install node-uuid
var uuid = require("node-uuid");
uuid.v4();
```

**Within Database Using Extension [pgcrypto](https://www.postgresql.org/docs/9.6/static/pgcrypto.html)**

```sql
-- load precompiled library code
CREATE EXTENSION pgcrypto;
-- SELECT gen_random_uuid();

CREATE SCHEMA IF NOT EXISTS snw;
CREATE TABLE snw.contacts(
   id UUID  PRIMARY KEY DEFAULT gen_random_uuid(),
   name     TEXT,
   email    TEXT
);
```

#### Serial

Note that `SERIAL` type is the same as:

```sql
CREATE TABLE tablename (
    colname SERIAL
);
```

```sql
CREATE SEQUENCE tablename_colname_seq;
CREATE TABLE tablename (
    colname integer NOT NULL DEFAULT nextval('tablename_colname_seq')
);
ALTER SEQUENCE tablename_colname_seq OWNED BY tablename.colname;
```

### Boolean

Use `BOOLEAN`, which has valid values of: **TRUE/FALSE, 't'/'f', 'y'/'n', 1/0**

```sql
CREATE TABLE test1 (a boolean, b text);
INSERT INTO test1 VALUES (TRUE, 'sic est');
INSERT INTO test1 VALUES (FALSE, 'non est');
```

### Text

Use `TEXT` and avoid `VARCHAR` or `CHAR` and especially `VARCHAR(n)` unless you specifically want to have a hard limit. Read [this](http://stackoverflow.com/questions/4848964/postgresql-difference-between-text-and-varchar-character-varying). Performance wise, `TEXT` is faster.

### Numbers

`INT` is a typical choice for integers. There are more choices [here](https://www.postgresql.org/docs/9.6/static/datatype-numeric.html) if needed for your use case.

### Currency

Use [NUMERIC](http://stackoverflow.com/questions/15726535/postgresql-which-datatype-should-be-used-for-currency) instead of money due to potential for rounding errors.

```sql
CREATE TABLE cc_inv_order (
  ...
  acct_id             INT REFERENCES acct(id),
  total               NUMERIC(10,2),
  paid                NUMERIC(10,2) DEFAULT 0,
  paid_date           TIMESTAMP WITHOUT TIME ZONE,
  descr               TEXT NOT NULL,
  ...
)
```

### Arrays

Arrays of any built-in or user-defined base type, enum type or composite type can be created.

```sql
CREATE TABLE sal_emp (
    name            text,
    pay_by_quarter  integer[],  -- one dimensional array
    schedule        text[][]    -- two dimensional array
);
```

Inserting:

```sql
-- '{ val1 delim val2 delim ... }'
INSERT INTO sal_emp
    VALUES (
      'Bill',
      '{10000, 10000, 10000, 10000}',
      '{ {"meeting", "lunch"}, {"training", "presentation"} }'
    );
```

Accessing:

```sql
SELECT name FROM sal_emp WHERE pay_by_quarter[1] <> pay_by_quarter[2];
SELECT pay_by_quarter[3] FROM sal_emp;
```

Modifying:

```sql
UPDATE sal_emp
SET pay_by_quarter = '{25000,25000,27000,27000}'
WHERE name = 'Carol';

-- update a single element
UPDATE sal_emp
SET pay_by_quarter[4] = 15000
WHERE name = 'Bill';

-- New array values can also be constructed
-- using the concatenation operator, ||
SELECT ARRAY[1,2] || ARRAY[3,4]; -- {1,2,3,4}
```

More info in [official documentation](https://www.postgresql.org/docs/9.6/static/arrays.html)

### JSON

Check out [official documentation](https://www.postgresql.org/docs/9.6/static/datatype-json.html)

### Enum

Are fast, transparent mapping of words to integer and lives in `pg_enum`. Use this as labels, otherwise, it's similar to other languages.

```sql
CREATE TYPE weekdays AS ('Mon', 'Tue', 'Wed', 'Thu', 'Fri');

-- create type
CREATE TYPE server_states
  AS ENUM ('running', 'offline', 'restarting');

-- create table with enum type
CREATE TABLE enum_test(
  id    serial,
  state server_states
);

-- good insert
INSERT INTO enum_test(state) VALUES ('offline');

-- Example of Bad Insert: "ERROR: invalid input value..."
INSERT INTO enum_test(state) VALUES ('destroyed');

-- You Can Add New Values
ALTER TYPE server_states ADD VALUE 'destroyed' AFTER 'offline';
INSERT INTO enum_test(state) VALUES ('destroyed');
```

## Data Definition

### Default Values

A column can be assigned a default value. **If no default value is declared, the default value is null**.

```sql
CREATE TABLE products (
  product_no  integer,
  name        text,
  price       numeric DEFAULT 9.99
);
```

Default value can be an expression, which is evaluated whenever the value is inserted, not when the table is created.

```sql
CREATE TABLE products (
  product_no  integer DEFAULT nextval('products_product_no_seq'),
  ...
);
```

### Constraints

#### Check Constraints

```sql
CREATE TABLE products (
  product_no  integer,
  name        text,
  price       numeric
    CHECK (price > 0),              -- column constraint
  discounted_price numeric
    CHECK (discounted_price > 0),   -- column constraint

  CHECK (price > discounted_price)  -- table constraint
);
```

#### NotNull Constraints

```sql
CREATE TABLE products (
  product_no integer NOT NULL,
  name text NOT NULL,
  price numeric
);
```

#### Unique Constraints

```sql
CREATE TABLE products (
  product_no integer UNIQUE,
  name text,
  price numeric
);
```

#### Primary Keys Constraints

```sql
CREATE TABLE products (
  product_no integer PRIMARY KEY, name text,
  price numeric
);
```

```sql
CREATE EXTENSION pgcrypto;
CREATE SCHEMA IF NOT EXISTS snw;
CREATE TABLE snw.contacts(
   id UUID  PRIMARY KEY DEFAULT gen_random_uuid(),
   name     TEXT,
   email    TEXT
);
```

#### Foreign Keys Constraints

```sql
CREATE TABLE products (
    product_no integer PRIMARY KEY,
    name text,
    price numeric
);
CREATE TABLE orders (
  order_id integer PRIMARY KEY,
  product_no integer REFERENCES products (product_no),
  quantity integer
);
```

Many to Many

```sql
CREATE TABLE products (
    product_no integer PRIMARY KEY,
    name text,
    price numeric
);

CREATE TABLE orders (
    order_id integer PRIMARY KEY,
    shipping_address text,
    ...
);

CREATE TABLE order_items (
  product_no integer
    REFERENCES products ON DELETE RESTRICT,
  order_id integer
    REFERENCES orders ON DELETE CASCADE,
  quantity integer,
  PRIMARY KEY (product_no, order_id)
);
```

- `RESTRICT` prevents deletion of referenced row.
- `CASCADE` means that when a referenced row is deleted, rows referencing it should be automatically deleted as well.

### Modifying Table Columns

Adding a Column:

```sql
-- Add Column to Table
ALTER TABLE table_name ADD COLUMN column_name data_type;

-- Add Column with Check
ALTER TABLE products ADD COLUMN description text
  CHECK (description <> '');
```

Adding a Constraint:

```sql
ALTER TABLE products ADD CHECK (name <> '');
ALTER TABLE products ADD CONSTRAINT some_name UNIQUE (product_no);
ALTER TABLE products ADD FOREIGN KEY (product_group_id)
  REFERENCES product_groups;

-- Removing a Constraint.
-- If it reference, add CASCADE
ALTER TABLE products DROP CONSTRAINT some_name;
```

Adding a Default Value:

```sql
ALTER TABLE products ALTER COLUMN price SET DEFAULT 7.77;

-- remove default value
ALTER TABLE products ALTER COLUMN price DROP DEFAULT;
```

Removing a Column:

```sql
-- Remove Column from Table. If it has a constraint,
-- Postgres will not silently drop that constraint.
ALTER TABLE table_name DROP COLUMN column_name;

-- Remove Column and Dependencies
ALTER TABLE products DROP COLUMN description CASCADE;
```

Changing Data Type:

```sql
-- Change Column Data Type
-- ALTER TABLE table_name ALTER COLUMN column_name data_type;
ALTER TABLE products ALTER COLUMN price TYPE numeric(10,2);
```

Renaming a Column:

```sql
-- Change Column Name
ALTER TABLE table_name RENAME COLUMN column_name TO new_name;
```

Renaming a Table:

```sql
ALTER TABLE products RENAME TO items;
```

## Data Manipulation

### Insert

```sql
CREATE TABLE weather (
    city            varchar(80),
    temp_lo         int,
    temp_hi         int,
    prcp            real,
    date            date
);

-- basic insert
INSERT INTO weather
VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');

-- explicit columns
INSERT INTO weather (city, temp_lo, temp_hi, prcp, date)
VALUES ('San Francisco', 43, 57, 0.0, '1994-11-29');

-- unordered columns
INSERT INTO weather (date, city, temp_hi, temp_lo)
VALUES ('1994-11-29', 'Hayward', 54, 37);
```

### Update

```sql
UPDATE products SET price = 10 WHERE price = 5;
UPDATE products SET price = price * 1.10;

-- update multiple columns
UPDATE mytable SET a = 5, b = 3, c = 1 WHERE a > 0;
```

### Delete

```sql
DELETE FROM products WHERE price = 10;
```

## Querying

```sql
SELECT * FROM weather;
SELECT city, temp_lo, temp_hi, prcp, date FROM weather;
SELECT * FROM weather WHERE city = 'San Francisco' AND prcp > 0.0;

-- expressions
SELECT city, (temp_hi + temp_lo)/2 AS temp_avg, date FROM weather;
```

### Joins

### LIMIT and OFFSET

### WITH queries (CTE)

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

### Operators

`+` `-` `*` `/` `%` `^`

Comparison:

`=` `!=` `<>` `>` `<` `>=` `<=`

`AND` `NOT` `OR`

### WHERE clause

```sql
SELECT * FROM COMPANY WHERE AGE >= 25 AND SALARY >= 65000;
SELECT * FROM COMPANY WHERE AGE >= 25 OR SALARY >= 65000;
SELECT * FROM COMPANY WHERE AGE IS NOT NULL;
SELECT * FROM COMPANY WHERE NAME LIKE 'Pa%';
SELECT * FROM COMPANY WHERE AGE IN ( 25, 27 );
SELECT * FROM COMPANY WHERE AGE NOT IN ( 25, 27 );
SELECT * FROM COMPANY WHERE AGE BETWEEN 25 AND 27;

SELECT AGE FROM COMPANY
  WHERE EXISTS (SELECT AGE FROM COMPANY WHERE SALARY > 65000);

SELECT * FROM COMPANY
  WHERE AGE > (SELECT AGE FROM COMPANY WHERE SALARY > 65000);

-- integer needs to be converted to text to use LIKE
SELECT * FROM COMPANY WHERE AGE::text LIKE '2%';

-- find records where address has hyphen
SELECT * FROM COMPANY WHERE ADDRESS  LIKE '%-%';
```

### LIMIT clause

```sql
SELECT * FROM COMPANY LIMIT 4;
SELECT * FROM COMPANY LIMIT 3 OFFSET 2;
```

### GROUP BY and HAVING clause

`GROUP BY` divides rows returned from `SELECT` statement into groups. For each group, you can apply an aggregate function like `SUM` or `COUNT`.

```sql
-- basic format
SELECT column_1, aggregate_function(column_2)
FROM tbl_name
GROUP BY column_1;
```

```sql
SELECT customer_id, SUM (amount)
FROM payment
GROUP BY customer_id;

SELECT staff_id, COUNT (payment_id)
FROM payment
GROUP BY staff_id;
```

This can also act like `DISTINCT`, for example: `SELECT cust_id FROM payment GROUP BY cust_id`.

To filter, use `HAVING` clause instead of `WHERE`

```sql
-- basic format
SELECT column_1, aggregate_function (column_2)
FROM tbl_name
GROUP BY column_1
HAVING condition;
```

```sql
-- select customers who spend >200
SELECT customer_id, SUM (amount)
FROM payment
GROUP BY customer_id
HAVING SUM (amount) > 200;
```

```sql
-- select store that has >300 customers
SELECT store_id, COUNT (customer_id)
FROM customer
GROUP BY store_id
HAVING COUNT (customer_id) > 300;
```

Can also sort with `ORDER BY`

```sql
SELECT customer_id, SUM (amount)
FROM payment
GROUP BY customer_id
ORDER BY SUM (amount) DESC;
```

## Create Triggers

What if you want to update a row's `update_date` and `updated_by` whenever a row is modified? Use **triggers**, which is a function that is invoked automatically whenever an event associated with a table occurs. Any event could be `INSERT`, `UPDATE`, `DELETE` or `TRUNCATE`.

A trigger is a special user-defined function that binds to a table. To create a new trigger, you must define a trigger function first, adn then bind this trigger function to a table.

Postgres has two main types of triggers: row and statement. For example, if you issue an UPDATE statement that affects 20 rows, the row level trigger will be invoked 20 times, while the statement level trigger will be invoked 1 time.

Syntax when creating a trigger:

```sql
CREATE TRIGGER trigger_name {BEFORE | AFTER | INSTEAD OF} {event [OR ...]}
   ON table_name
   [FOR [EACH] {ROW | STATEMENT}]
   EXECUTE PROCEDURE trigger_function
```

Example:

Let's create Tables:

```sql
-- create a table
CREATE TABLE employees(
   id int4 serial primary key,
   first_name varchar(40) NOT NULL,
   last_name varchar(40) NOT NULL
);
```

```sql
-- whenever employee's last name changes, we log
-- it to a separate table
CREATE TABLE employee_audits (
   id int4 serial primary key,
   employee_id int4 NOT NULL,
   last_name varchar(40) NOT NULL,
   changed_on timestamp(6) NOT NULL
)
```

First, create a new function:

```sql
-- checks if last name of employee changes, it will insert the
-- old last name into the employee_audits table
CREATE OR REPLACE FUNCTION log_last_name_changes()
  RETURNS trigger AS
$BODY$
BEGIN
 IF NEW.last_name <> OLD.last_name THEN
 INSERT INTO employee_audits(employee_id,last_name,changed_on)
 VALUES(OLD.id,OLD.last_name,now());
 END IF;

 RETURN NEW;
END;
$BODY$
```

Let's bind the trigger function to the employee's table:

```sql
CREATE TRIGGER last_name_changes
  BEFORE UPDATE
  ON employees
  FOR EACH ROW
  EXECUTE PROCEDURE log_last_name_changes();
```

Let's insert some sample data for testing:

```sql
INSERT INTO employees (first_name, last_name)
VALUES ('John', 'Doe');

INSERT INTO employees (first_name, last_name)
VALUES ('Lily', 'Bush');
```

Check if they worked:

```sql
SELECT * FROM employees;
SELECT * FROM employee_audits;
```

Another Example:

```sql
-- Table
CREATE TABLE cc_order (
  id            SERIAL NOT NULL PRIMARY KEY,
  acct_id       INT REFERENCES acct(id),
  inv_payment_id  INT REFERENCES inv_payment(id),
  total         NUMERIC(10,2),
  paid          NUMERIC(10,2) DEFAULT 0,
  paid_date     TIMESTAMP WITHOUT TIME ZONE,
  descr         TEXT NOT NULL,
  ref_num       TEXT
);

-- Create a Trigger
DROP TRIGGER IF EXISTS cc_order_trigger ON cc_order;
CREATE TRIGGER cc_order_trigger BEFORE UPDATE ON
  cc_order FOR EACH ROW
  EXECUTE PROCEDURE cc_order_check_paid()

-- Function to Execute
CREATE OR REPLACE FUNCTION cc_order_check_paid() RETURNS TRIGGER AS $$
BEGIN
  IF NEW.paid >= NEW.total THEN
    NEW.paid_date := NOW();
  END IF;

  RETURN NEW;
END;
$$ LANGUAGE plpgsql
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
pg_dump dbame > dbname.bak

# backup a database remotely
pg_dump -U user_name -h remote_host -p remote_port name_of_database > name_of_backup_file
```

### Compressed Dump

```shell
# use a compressed dump
pg_dump dbname | gzip > filename.gz

# restore
gunzip -c filename.gz | psql dbname
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

### Scenarios

Get Quick Snapshot and Send Remotely

```shell
# get backup
# note that this file will be big and uncompressed
pg_dump -d mydb -f mydb.sql

# transfer files
scp mydb.sql joe@198.199.19.19:~/mydb.sql

# on remote
psql mydb < mydb.sql
```

Create Nightly Backups with Cron

```shell
# readup on cron documentation
crontab -e

# backup everyday of the month, everyday of the week
# pg_dump -Fc means custom file backup
0 0 * * * joe pg_dump -Fc mydb > backups/mydb.bk
```

For more advanced scenarios like rotating backups, check the official documentation about [Automated Backup on Linux](https://wiki.postgresql.org/wiki/Automated_Backup_on_Linux). Add the script provided in the documentation to your cron and you should be set!

Make sure that you also create backups offsite! Like Amazon S3 and etc!

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

### Remove Default Authentication (NOT recommended)

If you don't want to deal with authentication, you can change the settings in **pg_hba.conf** file by changing **peer** to **trust**.

```shell
# search-replace the methods
sed -i 's/local.*all.*postgres.*peer/local all postgres trust/' /etc/postgresql/9.6/main/pg_hba.conf

sed -i 's/local.*all.*all.*peer/local all all trust/' /etc/postgresql/9.6/main/pg_hba.conf

# reload config and login to any user/db
service postgresql restart
psql -U postgres -d postgres
```

## Dos and Donts

- Do not store images in your database. Instead, upload to a storage service like S3, then store image URL in your database as a text field.
- Pagination is tough, but the worst technique is doing an `ORDER BY`, followed by a `LIMIT` and an `OFFSET` since this does not scale.
- Do not use integers as primary keys since this does not scale and you'll eventually run out. Use UUIDs instead. `create extension "uuid-ossp"; select uuid_generate_v4();`
- If you add a column with a default value on an existing table, this will trigger a full re-write of your table. Instead, it's better to allow null values at first so the operation is instant, then set your default, and then, with a background process go and retroactively update the data. When generating new schemas and you know that you should set a default value, go for it.
- Instead of over-normalization, think about whether you can make a column an array of a type enum, like in a categories column in a post table.
