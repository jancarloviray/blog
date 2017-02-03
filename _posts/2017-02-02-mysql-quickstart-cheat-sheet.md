---
layout: post
title: MySQL Quick Start and Cheat Sheet
permalink: blog/mysql-quickstart-cheat-sheet/
comments: True
excerpt_separator: <!--more-->
---

## Quick Start

```sh
mysqladmin -u root password # set root pass
service mysql start         # start mysql if not started
mysql -u root -p            # login with password prompt
```

### Setup User and Database

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password'; -- change root pass
```

```sql
SHOW DATABASES;
CREATE DATABASE cats;

USE cats;
SHOW TABLES;
```

```sql
CREATE USER 'jump'@'localhost' IDENTIFIED BY 'secret'; -- create a new user
GRANT ALL ON cats.* TO 'jump'@'localhost';             -- grant privileges
FLUSH PRIVILEGES;                                      -- refresh privileges
SHOW GRANTS;                                           -- show users and grants
```

### Create Tables

```sql
CREATE TABLE cats
(
  id              INT unsigned NOT NULL AUTO_INCREMENT, -- Unique ID for the record
  name            VARCHAR(150) NOT NULL,                -- Name of the cat
  owner           VARCHAR(150) NOT NULL,                -- Owner of the cat
  birth           DATE NOT NULL,                        -- Birthday of the cat
  PRIMARY KEY     (id)                                  -- Make the id the primary key
);

ALTER TABLE cats ADD gender CHAR(1) AFTER name;
ALTER TABLE cats DROP gender;

DESCRIBE cats;
```

### CRUD

```sql
INSERT INTO cats ( name, owner, birth) VALUES
  ( 'Sandy', 'Lennon', '2015-01-03' ),
  ( 'Cookie', 'Casey', '2013-11-13' ),
  ( 'Charlie', 'River', '2016-05-21' );
```

```sql 
SELECT * FROM cats;
SELECT name FROM cats WHERE owner = 'Casey';

DELETE FROM cats WHERE name='Cookie';
```
<!--more-->

## Cheat Sheet

## Common Shell Commands

```sh
# run the mysql client
mysql -h [host] -u [user] -p 
mysql -h [host] -u [user] -p [database]

# export data
mysqldump -u [user] -p [database] > data_backup.sql
```

## Common MySQL Commands

```sql
\s;                 -- show status
SHOW VARIABLES;     -- show configuration params
SHOW PROCESSLIST;   -- show running queries

SELECT @@version;   -- mysql version
SELECT @@datadir;   -- location of db files
SELECT @@hostname;  -- current hostname
SELECT USER();      -- current user
SELECT DATABASE();  -- current database

SHOW DATABASES;
USE db_name;
SHOW TABLES;
DESCRIBE table_name;
```

## Common Admin Commands 

```sql 
-- list all mysql users in database
SELECT host, user FROM mysql.user;  

-- show grant for all users
SHOW GRANTS;
SHOW GRANTS FOR 'user';

CREATE USER 'username'@'hostname' IDENTIFIED BY 'password';

GRANT ALL ON db_name.* TO 'username'@'hostname' IDENTIFIED BY 'password';

-- refresh privileges
FLUSH PRIVILEGES;

DROP USER 'username'@'hostname';
```

## Common Data Types

|PURPOSE|EXAMPLE|NOTES|
|-|-|-|
|integers|`int(5)`|
|floats|`float(12,3)`|Will round your values|
|money|`decimal(10,2)`|Up to 10 digits. 2 points|
|date|`date`|
|datetime|`timestamp(8)`|(8)YYYYMMDD, (12)YYYYMMDDHHMMSS|
|string|`varchar(20)`|
|large text|`blob`|
|enum|`enum('blue','red','gray')`|Not recommended to use|

## Common Functions

|FUNCTION|NOTES|
|-|-|
|`NOW()`|datetime input|
|`strcomp(str1,str2)`|Compare string|
|`lower(str)`|
|`upper(str)`|
|`ltrim(str)`|
|`substring(str,idx1,idx2)`|
|`password(str)`|Encry password|
|`curdate()`|Get date|
|`curtime()`|Get time|

## Common CRUD

```sql
-- INSERT
INSERT INTO people (name) VALUES ("tom"),("dick"),("harry");  -- Batch Insert w/ given name column
INSERT INTO people VALUES ('MyName', '2002­08­31');             -- Inserting one row at a time (Use NULL for NULL)
INSERT INTO people (name, company_id, created_at) 
    SELECT name, 50, NOW() FROM people WHERE company_id = 49; -- copying rows from the same table

-- SELECT
SELECT col FROM tbl WHERE conditions;           -- Retrieving information (general):  
SELECT * FROM tbl;                              -- All values 
SELECT * FROM tbl WHERE rec_name = "value";     -- Some values
SELECT * FROM tbl WHERE rec1 = "value1"         -- Multiple critera
    AND rec2 = "val2"; 

SELECT column_name FROM table;                    -- Selecting specific columns
SELECT DISTINCT column_name FROM table;           -- Retrieving unique output records
SELECT col1, col2 FROM table ORDER BY col2;       -- Sorting
SELECT col1, col2 FROM table ORDER BY col2 DESC;  -- Sorting Backward
SELECT COUNT(*) FROM table;                       -- Counting Rows
SELECT owner, COUNT(*) FROM table GROUP BY owner; -- Grouping with Counting
SELECT MAX(col_name) AS label FROM table;         -- Maximum value

SELECT pet.name, comment FROM pet, event -- Selecting from multiple tables 
WHERE pet.name = event.name;             -- (You can join again the same table by alias w/ "AS")

-- SEARCH

SELECT * FROM table WHERE rec LIKE "blah%"; -- % is wildcard ­ arbitrary ### of chars
SELECT * FROM table WHERE rec LIKE "_____"; -- Find 5­char values: _ is any single character
SELECT * FROM table WHERE rec RLIKE "^b$";  -- regex

-- JOIN 
SELECT * FROM table_1 INNER JOIN table_2 ON conditions;
SELECT * FROM table1 LEFT JOIN table2 ON conditions;

-- UPDATE
UPDATE table SET column_name = "new_value" -- Fixing all records with a certain value
WHERE record_name = "value";

-- DELETE 
DELETE FROM table WHERE condition;

DELETE table_1, table2 FROM table_1
INNER JOIN table_2 ON table_1.column_1 = table_2.column_2
WHERE condition;
```

## Common Table Manipulations

```sql
CREATE TABLE table_name (field1_name TYPE(SIZE), field2_name TYPE(SIZE));  

DROP TABLE table_name

ALTER TABLE authors 
    ADD     name VARCHAR(255), 
    CHANGE  author_work_id wokr_id INT,
    DROP    nickname,
    CHANGE `count(*)` cnt bigint(21),  ### renaming
    ALTER   is_rich SET DEFAULT FALSE;

-- Adding a column to an already­created table
ALTER TABLE tbl ADD COLUMN [column_create syntax] AFTER col_name;

-- Removing a column
ALTER TABLE tbl DROP COLUMN col;

-- Adding index to an existing table;
ALTER TABLE table_name ADD INDEX index_name (col_name);
```
