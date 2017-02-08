---
layout: post
title: Intro to SQL Cheat Sheet
permalink: blog/intro-to-sql-cheat-sheet/
comments: True
excerpt_separator: <!--more-->
---

Here's a quick reference / cheat sheet for SQL language. The CRUD are platform agnostic. Just copy-paste this in your SQL editor like MySQL Workbench and you can step through it. Enjoy!

<!--more-->

```sql
-- create database

DROP DATABASE IF EXISTS sample_hr;
CREATE DATABASE sample_hr;
USE sample_hr;

-- create tables and relationships

CREATE TABLE regions (
    region_id INT(11) AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(25) DEFAULT NULL
);

CREATE TABLE countries (
    country_id CHAR(2) PRIMARY KEY,
    name VARCHAR(40) DEFAULT NULL,
    region_id INT(11) NOT NULL,
    FOREIGN KEY (region_id)
        REFERENCES regions (region_id)
        ON DELETE CASCADE ON UPDATE CASCADE
);

CREATE TABLE locations (
    location_id INT(11) AUTO_INCREMENT PRIMARY KEY,
    address VARCHAR(40) DEFAULT NULL,
    postal_code VARCHAR(12) DEFAULT NULL,
    city VARCHAR(30) NOT NULL,
    state VARCHAR(25) DEFAULT NULL,
    country_id CHAR(2) NOT NULL,
    FOREIGN KEY (country_id)
        REFERENCES countries (country_id)
        ON DELETE CASCADE ON UPDATE CASCADE
);

CREATE TABLE jobs (
    job_id INT(11) AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(35) NOT NULL,
    min_salary DECIMAL(8 , 2 ) DEFAULT NULL,
    max_salary DECIMAL(8 , 2 ) DEFAULT NULL
);

CREATE TABLE departments (
    department_id INT(11) AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(30) NOT NULL,
    location_id INT(11) DEFAULT NULL,
    FOREIGN KEY (location_id)
        REFERENCES locations (location_id)
        ON DELETE CASCADE ON UPDATE CASCADE
);

CREATE TABLE employees (
    employee_id INT(11) AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(20) DEFAULT NULL,
    last_name VARCHAR(25) NOT NULL,
    email VARCHAR(100) NOT NULL,
    phone VARCHAR(20) DEFAULT NULL,
    hire_date DATE NOT NULL,
    job_id INT(11) NOT NULL,
    salary DECIMAL(8 , 2 ) NOT NULL,
    manager_id INT(11) DEFAULT NULL,
    department_id INT(11) DEFAULT NULL,
    FOREIGN KEY (job_id)
        REFERENCES jobs (job_id)
        ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (department_id)
        REFERENCES departments (department_id)
        ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (manager_id)
        REFERENCES employees (employee_id)
);

CREATE TABLE dependents (
    dependent_id INT(11) AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    relationship VARCHAR(25) NOT NULL,
    employee_id INT(11) NOT NULL,
    FOREIGN KEY (employee_id)
        REFERENCES employees (employee_id)
        ON DELETE CASCADE ON UPDATE CASCADE
);

-- insert seed data

INSERT INTO regions (region_id,name)
VALUES      (1,'Europe'), (2,'Americas'), (3,'Asia'), (4,'Middle East and Africa');
INSERT INTO countries (country_id,name,region_id)
VALUES      ('AR','Argentina',2), ('AU','Australia',3), ('BE','Belgium',1), ('BR','Brazil',2), ('CA','Canada',2), ('CH','Switzerland',1), ('CN','China',3), ('DE','Germany',1), ('DK','Denmark',1), ('EG','Egypt',4), ('FR','France',1), ('HK','HongKong',3), ('IL','Israel',4), ('IN','India',3), ('IT','Italy',1), ('JP','Japan',3), ('KW','Kuwait',4), ('MX','Mexico',2), ('NG','Nigeria',4), ('NL','Netherlands',1), ('SG','Singapore',3), ('UK','United Kingdom',1), ( 'US','United States of America',2 ), ('ZM','Zambia',4), ('ZW','Zimbabwe',4);
INSERT INTO locations (location_id,address,postal_code,city,state,country_id)
VALUES      ( 1400,'2014 Jabberwocky Rd','26192','Southlake','Texas','US' ), ( 1500,'2011 Interiors Blvd','99236','South San Francisco','California','US' ), ( 1700,'2004 Charade Rd','98199','Seattle','Washington','US' ), ( 1800,'147 Spadina Ave','M5V 2L7','Toronto','Ontario','CA' ), ( 2400,'8204 Arthur St',NULL,'London',NULL,'UK' ), ( 2500,'Magdalen Centre, The Oxford Science Park','OX9 9ZB','Oxford','Oxford','UK' ), ( 2700,'Schwanthalerstr. 7031','80925','Munich','Bavaria','DE' );
INSERT INTO departments (department_id,name,location_id)
VALUES      (1,'Administration',1700), (2,'Marketing',1800), (3,'Purchasing',1700), (4,'Human Resources',2400), (5,'Shipping',1500), (6,'IT',1400), (7,'Public Relations',2700), (8,'Sales',2500), (9,'Executive',1700), (10,'Finance',1700), (11,'Accounting',1700);
INSERT INTO jobs (job_id,title,min_salary,max_salary)
VALUES      ( 1,'Public Accountant','4200.00','9000.00' ), ( 2,'Accounting Manager','8200.00','16000.00' ), ( 3,'Administration Assistant','3000.00','6000.00' ), ( 4,'President','20000.00','40000.00' ), ( 5,'Administration Vice President','15000.00','30000.00' ), ( 6,'Accountant','4200.00','9000.00' ), ( 7,'Finance Manager','8200.00','16000.00' ), ( 8,'Human Resources Representative','4000.00','9000.00' ), ( 9,'Programmer','4000.00','10000.00' ), ( 10,'Marketing Manager','9000.00','15000.00' ), ( 11,'Marketing Representative','4000.00','9000.00' ), ( 12,'Public Relations Representative','4500.00','10500.00' ), ( 13,'Purchasing Clerk','2500.00','5500.00' ), ( 14,'Purchasing Manager','8000.00','15000.00' ), ( 15,'Sales Manager','10000.00','20000.00' ), ( 16,'Sales Representative','6000.00','12000.00' ), ( 17,'Shipping Clerk','2500.00','5500.00' ), ( 18,'Stock Clerk','2000.00','5000.00' ), ( 19,'Stock Manager','5500.00','8500.00' );
INSERT INTO employees (employee_id,first_name,last_name,email,phone,hire_date,job_id,salary,manager_id,department_id)
VALUES      ( 100,'Steven','King','steven.king@sqltutorial.org','515.123.4567','1987-06-17',4,'24000.00',NULL,9 ), ( 101,'Neena','Kochhar','neena.kochhar@sqltutorial.org','515.123.4568','1989-09-21',5,'17000.00',100,9 ), ( 102,'Lex','De Haan','lex.de haan@sqltutorial.org','515.123.4569','1993-01-13',5,'17000.00',100,9 ), ( 103,'Alexander','Hunold','alexander.hunold@sqltutorial.org','590.423.4567','1990-01-03',9,'9000.00',102,6 ), ( 104,'Bruce','Ernst','bruce.ernst@sqltutorial.org','590.423.4568','1991-05-21',9,'6000.00',103,6 ), ( 105,'David','Austin','david.austin@sqltutorial.org','590.423.4569','1997-06-25',9,'4800.00',103,6 ), ( 106,'Valli','Pataballa','valli.pataballa@sqltutorial.org','590.423.4560','1998-02-05',9,'4800.00',103,6 ), ( 107,'Diana','Lorentz','diana.lorentz@sqltutorial.org','590.423.5567','1999-02-07',9,'4200.00',103,6 ), ( 108,'Nancy','Greenberg','nancy.greenberg@sqltutorial.org','515.124.4569','1994-08-17',7,'12000.00',101,10 ), ( 109,'Daniel','Faviet','daniel.faviet@sqltutorial.org','515.124.4169','1994-08-16',6,'9000.00',108,10 ), ( 110,'John','Chen','john.chen@sqltutorial.org','515.124.4269','1997-09-28',6,'8200.00',108,10 ), ( 111,'Ismael','Sciarra','ismael.sciarra@sqltutorial.org','515.124.4369','1997-09-30',6,'7700.00',108,10 ), ( 112,'Jose Manuel','Urman','jose manuel.urman@sqltutorial.org','515.124.4469','1998-03-07',6,'7800.00',108,10 ), ( 113,'Luis','Popp','luis.popp@sqltutorial.org','515.124.4567','1999-12-07',6,'6900.00',108,10 ), ( 114,'Den','Raphaely','den.raphaely@sqltutorial.org','515.127.4561','1994-12-07',14,'11000.00',100,3 ), ( 115,'Alexander','Khoo','alexander.khoo@sqltutorial.org','515.127.4562','1995-05-18',13,'3100.00',114,3 ), ( 116,'Shelli','Baida','shelli.baida@sqltutorial.org','515.127.4563','1997-12-24',13,'2900.00',114,3 ), ( 117,'Sigal','Tobias','sigal.tobias@sqltutorial.org','515.127.4564','1997-07-24',13,'2800.00',114,3 ), ( 118,'Guy','Himuro','guy.himuro@sqltutorial.org','515.127.4565','1998-11-15',13,'2600.00',114,3 ), ( 119,'Karen','Colmenares','karen.colmenares@sqltutorial.org','515.127.4566','1999-08-10',13,'2500.00',114,3 ), ( 120,'Matthew','Weiss','matthew.weiss@sqltutorial.org','650.123.1234','1996-07-18',19,'8000.00',100,5 ), ( 121,'Adam','Fripp','adam.fripp@sqltutorial.org','650.123.2234', '1997-04-10',19, '8200.00',100,5 ), ( 122,'Payam','Kaufling','payam.kaufling@sqltutorial.org', '650.123.3234', '1995-05-01',19,'7900.00',100,5 ), ( 123,'Shanta','Vollman','shanta.vollman@sqltutorial.org', '650.123.4234', '1997-10-10',19,'6500.00',100,5 ), ( 126,'Irene','Mikkilineni','irene.mikkilineni@sqltutorial.org', '650.124.1224', '1998-09-28',18,'2700.00',120,5 ), ( 145,'John','Russell','john.russell@sqltutorial.org',NULL, '1996-10-01',15, '14000.00',100,8 ), ( 146,'Karen','Partners','karen.partners@sqltutorial.org',NULL, '1997-01-05',15, '13500.00',100,8 ), ( 176,'Jonathon','Taylor','jonathon.taylor@sqltutorial.org',NULL, '1998-03-24',16, '8600.00',100,8 ), ( 177,'Jack','Livingston','jack.livingston@sqltutorial.org',NULL, '1998-04-23',16, '8400.00',100,8 ), ( 178,'Kimberely','Grant','kimberely.grant@sqltutorial.org',NULL, '1999-05-24',16, '7000.00',100,8 ), ( 179,'Charles','Johnson','charles.johnson@sqltutorial.org',NULL, '2000-01-04',16, '6200.00',100,8 ), ( 192,'Sarah','Bell','sarah.bell@sqltutorial.org','650.501.1876', '1996-02-04',17, '4000.00',123,5 ), ( 193,'Britney','Everett','britney.everett@sqltutorial.org', '650.501.2876', '1997-03-03',17,'3900.00',123,5 ), ( 200,'Jennifer','Whalen','jennifer.whalen@sqltutorial.org', '515.123.4444', '1987-09-17',3,'4400.00',101,1 ), ( 201,'Michael','Hartstein','michael.hartstein@sqltutorial.org', '515.123.5555', '1996-02-17',10,'13000.00',100,2 ), ( 202,'Pat','Fay','pat.fay@sqltutorial.org','603.123.6666', '1997-08-17',11, '6000.00',201,2 ), ( 203,'Susan','Mavris','susan.mavris@sqltutorial.org','515.123.7777','1994-06-07', 8,'6500.00',101,4 ), ( 204,'Hermann','Baer','hermann.baer@sqltutorial.org','515.123.8888','1994-06-07', 12,'10000.00',101,7 ), ( 205,'Shelley','Higgins','shelley.higgins@sqltutorial.org', '515.123.8080', '1994-06-07',2,'12000.00',101,11 ), ( 206,'William','Gietz','william.gietz@sqltutorial.org', '515.123.8181', '1994-06-07',1,'8300.00',205,11 );
INSERT INTO dependents (dependent_id,first_name,last_name,relationship,employee_id)
VALUES      ( 1,'Penelope','Gietz','Child',206 ), ( 2,'Nick','Higgins','Child',205 ), ( 3,'Ed','Whalen','Child',200 ), ( 4,'Jennifer','King','Child',100 ), ( 5,'Johnny','Kochhar','Child',101 ), ( 6,'Bette','De Haan','Child',102 ), ( 7,'Grace','Faviet','Child',109 ), ( 8,'Matthew','Chen','Child',110 ), ( 9,'Joe','Sciarra','Child',111 ), ( 10,'Christian','Urman','Child',112 ), ( 11,'Zero','Popp','Child',113 ), ( 12,'Karl','Greenberg','Child',108 ), ( 13,'Uma','Mavris','Child',203 ), ( 14,'Vivien','Hunold','Child',103 ), ( 15,'Cuba','Ernst','Child',104 ), ( 16,'Fred','Austin','Child',105 ), ( 17,'Helen','Pataballa','Child',106 ), ( 18,'Dan','Lorentz','Child',107 ), ( 19,'Bob','Hartstein','Child',201 ), ( 20,'Lucille','Fay','Child',202 ), ( 21,'Kirsten','Baer','Child',204 ), ( 22,'Elvis','Khoo','Child',115 ), ( 23,'Sandra','Baida','Child',116 ), ( 24,'Cameron','Tobias','Child',117 ), ( 25,'Kevin','Himuro','Child',118 ), ( 26,'Rip','Colmenares','Child',119 ), ( 27,'Julia','Raphaely','Child',114 ), ( 28,'Woody','Russell','Child',145 ), ( 29,'Alec','Partners','Child',146 ), ( 30,'Sandra','Taylor','Child',176 );

-- ORDER BY
SELECT first_name, hire_date, salary FROM employees
ORDER BY hire_date DESC;

-- LIMIT
SELECT employee_id, first_name FROM employees
ORDER BY first_name LIMIT 5 OFFSET 3;

-- LIMIT (with SUB QUERY)
SELECT employee_id, first_name, salary FROM employees
WHERE salary = (SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 1);

-- DISTINCT
SELECT DISTINCT salary FROM employees ORDER BY salary DESC;
SELECT DISTINCT job_id, salary FROM employees ORDER BY job_id , salary DESC;

-- WHERE AND
SELECT first_name, hire_date FROM employees
WHERE YEAR(hire_date) >= 1997
AND YEAR(hire_date) <= 1998;

-- WHERE OR
SELECT first_name, hire_date FROM employees
WHERE YEAR(hire_date) = 2000
    OR YEAR(hire_date) = 1999
    OR YEAR(hire_date) = 1990;

-- WHERE AND OR
SELECT first_name, hire_date, department_id FROM employees
WHERE department_id = 3
    AND (YEAR(hire_date) = 1997 OR YEAR(hire_date) = 1998)
ORDER BY first_name, hire_date;

-- IN (similar to OR)
SELECT first_name, hire_date FROM employees
WHERE YEAR(hire_date) IN (1990 , 1999, 2000)
ORDER BY hire_date;

-- NULL
SELECT employee_id, first_name, phone FROM employees
WHERE phone IS NULL;

SELECT employee_id, first_name, phone FROM employees
WHERE phone IS NOT NULL;

-- BETWEEN
SELECT employee_id, first_name, salary FROM employees
WHERE salary BETWEEN 2500 AND 2900;

SELECT employee_id, first_name, hire_date FROM employees
WHERE hire_date BETWEEN '1999-01-01' AND '2000-12-31'
ORDER BY hire_date;

-- LIKE
SELECT employee_id, first_name FROM employees
WHERE first_name LIKE '%an%';

SELECT employee_id, first_name FROM employees
WHERE first_name LIKE '%are_';

SELECT employee_id, first_name FROM employees
WHERE first_name LIKE 'S%'AND first_name NOT LIKE 'Sh%';

-- NOT
SELECT employee_id, first_name, salary FROM employees
WHERE department_id = 5 AND NOT salary > 5000
ORDER BY salary;

SELECT employee_id, first_name, department_id FROM employees
WHERE department_id NOT IN (1, 2, 3)
ORDER BY first_name;

SELECT employee_id, first_name, salary FROM employees
WHERE salary NOT BETWEEN 3000 AND 5000
ORDER BY salary;

-- EXISTS
SELECT employee_id, first_name FROM employees e
WHERE NOT EXISTS (SELECT employee_id FROM dependents d WHERE d.employee_id = e.employee_id);

-- INNER JOIN
SELECT e.first_name, d.name FROM employees e
INNER JOIN departments d ON d.department_id = e.department_id
WHERE e.department_id IN (1 , 2, 3);

SELECT first_name, last_name, title, name FROM employees e
INNER JOIN departments d ON d.department_id = e.department_id
INNER JOIN jobs j ON j.job_id = e.job_id
WHERE e.department_id IN (1 , 2, 3);

-- Outer joins also combine rows from two or more tables, but unlike inner joins, they do not require the join condition to be met. Instead, every row in the left table is returned in the result set, and if the join condition is not met, then NULL values are used to fill in the columns from the right table.
-- The left table is simply the first table that appears in the statement. That means whatever is in the FROM will be the left table.

-- GROUP BY
SELECT department_id, COUNT(employee_id) headcount FROM employees
GROUP BY department_id;

SELECT e.department_id, name, COUNT(employee_id) headcount FROM employees e
INNER JOIN departments d ON d.department_id = e.department_id
GROUP BY e.department_id;

SELECT e.department_id, name, SUM(salary) total_salary FROM employees e
INNER JOIN departments d ON d.department_id = e.department_id
GROUP BY e.department_id;

-- UNION (merge two queries)
SELECT first_name, last_name FROM employees
UNION
SELECT first_name, last_name FROM dependents
ORDER BY last_name;

-- INSERT
INSERT INTO dependents (first_name, last_name, relationship, employee_id )
VALUES ('Cameron', 'Bell', 'Child', 192 ), ('Michelle', 'Bell', 'Child', 192 );

INSERT INTO dependents_archive SELECT * FROM dependents;

-- UPDATE
UPDATE employees
SET last_name = 'Lopez'
WHERE employee_id = 192;

UPDATE dependents
SET last_name = (
 SELECT last_name FROM employees
 WHERE employee_id = dependents.employee_id
);

-- DELETE
DELETE FROM employees
WHERE employee_id = 192;

-- FUNCTIONS
SELECT ROUND(AVG(DISTINCT salary), 2)
FROM employees;

SELECT COUNT(*)
FROM employees;

SELECT department_id, COUNT(*) FROM employees
GROUP BY department_id;
```
