---
layout: post
title: Common Linux Commands
permalink: blog/common-linux-commands/
comments: True
---

Here are some of the more common linux commands I use when managing files and services in a unix environment. Here's a mind-map overview if you're into that. Expand this blog topic if you'd like to see the text version or read more. ![Linux Commands](https://raw.githubusercontent.com/jancarloviray/jancarloviray.github.io/master/_media/Linux.png)

## Archiving

### How to extract a tar file?

`tar xzf file.tar.gz`

### How to compress a file?

`tar czf zipped.tar.gz unzipped.pdf`

## Text Streams

### How to see contents in a file?

`cat somefile`

### How to see contents in a file without exiting?

`less somefile`

### How to find contents from within a file?

`cat /some/text/file | grep "text to find"`

### How to stream a command into a new file?

`cat /etc/group | grep ubuntu > somefile`

### How to stream a command and append to an existing file?

`cat /etc/group | grep ubuntu >> somefile`

### Check first 5 lines of a file

`head somefile`

### Check last 5 lines of a fail

`tail somefile`

### Parse a file into columns

`cut -d: -f3 getThirdColumn.txt`

Note that (-d:) means to use colon (:) as the delimiter and (-f3) means to process 3rd column of each line

### How to sort the contents in a file?

`cut -d: -f3 parseme.txt | sorn -n`

This pipes the content into a sorted stream and prints in screen

## Bash

### How to specify a file's runner?

Add `#!/bin/bash`, often called the "shebang" on top of the file

### Exit a script with success

Add `exit 0` at the end of the script file

### Make a script executable

`chmod +x myscript.sh`

Note that you must have the shebang on top of file

## Creating Jobs/Services

### How to run a script in the background?

Append `&` at the end of the command. For example, `tail -f /var/log/syslog &`.

Note that *when you exit the shell, the process will also terminate with a hangup signal (kill -SIGHUP [pid]).* This means that if you're ssh'd to a server, you run a process and put it in a background and you exit the server.. the process will also then terminate.

### How to run a script in the background without getting it terminated on shell exit?

Append `nohup` in your command, which means "no hangup". It's a poor man's way of running a process as a daemon. Use this only on processes that will take some time, but will not hang around too long.

### How to check for background tasks?

`jobs`

### How to bring back a task into the foreground?

`fg [job-id]`

### How to bring back a task in the background?

After you suspend it (CTRL+Z), you bring it back using `bg [job-id]`

## Managing System Services

```
sudo service [service-name] restart

sudo service [service-name] stop

sudo service [service-name] status
```

## Firewall

### How to display current firewall rules?

`sudo iptables -L`

## Files

### How to find files in a directory?

`find /usr/share/doc -name '*.pdf'`

### How to delete found files?

`find /usr/share/doc -name '*.pdf' -delete`

### How to execute a command on found files?

`find /usr/share/doc -name '*.pdf' -exec cp {} . \;`

### How to find files bigger than a certain size?

`find /boot -size +20000k -type f`

### How to find disk free space?

`df`

### How to find disk usage?

`du`

## Streams and Redirection

### How to redirect output and overwrite a file?

`>`

### How to redirect output and append a file?

`>>`

### How to redirect standard output (without errors) into a file?

`1> or 1>>`

### How to redirect error output into a file, while displaying standard output still?

`2> or 2>>`

### How to display output while redirecting error output and NOT displaying the errors in terminal?

`2>| or 2>>|`

### How to redirect both standard input and error output (STDIN and STDERR) into a file?

`&> or &>>`

## File Permissions and Ownerships

### How to change a file's permission?

Remember this:
- read (r) = 4
- write (w) = 2
- execute (x) = 1

Scenarios:
- just want read permissions? 4
- just want read and write permissions? 4 + 2 = 6
- just want execute and read? 4 + 1 = 5
- want all permissions? 7

Use this format: `chmod [user/group/others] file`

Example: `chmod 467 file` will allow read-only for user, read-write only for group, and full-permission for others.

You can also use the format: `chmod u=r,g=rw,o=rwx file1`

Important arguments:
- (-r) for recursive
- (-v) for verbose

### How to show current user/group of user?

`id`

### How to change file into a new group?

`chgrp [grpname] file`

### How to change file to a new owner?

`chown [uname].[grpname] file`

Examples:
- `chown www-data file1` changes owner to www-data
- `chown .www-data file1` changes group to www-data
- `chown root.root file1` changes both owner and group to root

### How to find current user's groups?

`groups`

### How to change user's current group?

`newgrp [grpname]`

### How to change to root user?

`su`

### How to copy a file without changing permissions?

note that when doing a cp, it changes the ownership of a file and the timestamp also... i.e., if you're as root, it will remove old ownership/groups and make the new files as root.. add (-a) to not touch file ownerships when copying

`cp -a old new`

## SSH

### Check what your hostname is

`hostname`

### How to SSH to a server?

`ssh 192.168.56.105`

Note that since you are not specifying a username there, it will use the same user name that you are currently on right now.

## How to create aliases for shortcuts?

`touch ~/.ssh/config`

Add this:

```
Host server1
  HostName 192.168...
  User root
  Port 22

Host server2
  HostName 192....
  User root
  Port 22
```

Now you can just say `ssh server1` or `ssh server2`

## How to create a public/private key?

`ssh-keygen -t rsa`

Here, we are specifying the type, which will be rsa.

## How to copy your public key into the server? (you must have root access still)

`ssh-copy-id -i ~/.ssh/id_rsa.pub server1`

## How to prevent root ssh access in server?

`vim /etc/ssh/sshd_config` and find PermitRoot.. and change yes to no.

## How to copy files securely through ssh?

```
scp /etc/hosts server1:/tmp

// origin to remote
scp /some/file someSvr:/some/dir

// remote to origin
scp someSvr:/tmp/hosts .
```

