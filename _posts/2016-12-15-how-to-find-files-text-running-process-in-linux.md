---
layout: post
title: How to Find Files, Text, Processes and etc in Linux
permalink: blog/how-to-find-files-text-running-process-in-linux.md/
comments: True
excerpt_separator: <!--more-->
---

I'm building a "cheat sheet" on finding files, monitoring and everything related to that in Linux. I hope this helps!

## Recursively Find Files Containing a Specific Text

```shell
grep -rn "pattern" /path/to/dir

# include only certain files
grep -rn "pattern" --include="*.js" --exclude="*node_modules/*" --exclude="*.min.js*" /path/to/dir
```

- add `-w` to match whole words instead of partial
- add `-i` for case insensitive search

## How to Find Files in Linux

```shell
find /path/to/dir -name "*.js"

# execute a command on those files
# "{} \;" just means the command ends
find /path/to/dir -name "*.js" -exec rm -f {} \;
find /path/to/dir -name "*.js" -exec chmod 700 {} \;
```

The `find` command is very powerful and can do additional things like:

- finding files with certain permissions `find . -type f -perm 0664`
- finding files belonging to a specific user `find . -user joe`
- finding files modified N days back `find . -mtime 50`
- finding files of given size `find . -size 50M`

## Find If Your Process Is Running

```shell
ps aux | grep postgres
```

<!--more-->

## Find Open Ports and the Processes That Owns Them

```shell
lsof -i
lsof -i | grep apache

# list processes using tcp port 80
lsof -i tcp:80
```

`lsof` is a command that lists open files. Running it by itself will list all open files in active processes. Adding `-i` without option lists all that has network files. Adding an option

## Find Your Internal IP Address

```shell
/sbin/ifconfig
```
