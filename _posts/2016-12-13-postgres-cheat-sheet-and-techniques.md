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

# Change a User's Password
`ALTER ROLE [role_name] WITH PASSWORD '[new_password]';`

# Allow User to Create Databases
`ALTER USER <username> WITH CREATEDB;`

# List Roles
`\du`

## Tips and Techniques

### Write `where` before doing a `delete` or `update`

Before you run your code and most especially during deletes and updates, make sure you take a deep breath and do another scan. This will save you from pain and misery.

### Get list of databases and their users

`\l`

<!--more-->

Still in development...
