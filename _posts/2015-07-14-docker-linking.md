---
layout: post
title: What Does Docker Link Do?
permalink: blog/what-does-docker-link-do/
comments: True
---

How does Docker link container? What happens during the process? Docker creates a secure tunnel between the containers that doesn't need to expose any ports externally on the container. Notice that there is no need to use either the `-P` or `-p` flags.

Docker exposes connectivity from the source container to the recipient container in two ways:

- Environmental Variables
- Updating the `/etc/hosts` file

## Environment Variables

Docker creates several environment variables when you link containers. It exposes all environment variables originating from Docker from the source container, which includes `ENV` commands, `-e` `--env` and `--env-file` options on the `docker run` command when the source container is started.

Docker sets an `<alias>_NAME` environment variable for each target container listed in the `--link` parameter. For example, if a new container called `web` is linked to a database container called `db` via `--link db:webbed`, then Docker creates a `WEBDB_NAME=/web/webdb` variable in the `web` container.

Docker also defines a set of environment variables for each port exposed by the source container. Each variable has a unique prefix:

`<name>_PORT_<port>_<protocol>`

The `<name>` is specified in the `--link` parameter (i.e.: webdb)
The `<port>` number is the one exposed
The `<protocol>` is either TCP or UDP

Docker uses this format to define environment variables such as:

The prefix_ADDR variable contains the IP Address from the URL, for example WEBDB_PORT_8080_TCP_ADDR=172.17.0.82.
The prefix_PORT variable contains just the port number from the URL for example WEBDB_PORT_8080_TCP_PORT=8080.
The prefix_PROTO variable contains just the protocol from the URL for example WEBDB_PORT_8080_TCP_PROTO=tcp.

Note that if the container exposes multiple ports, an environment variable set is defined for each one.

## Updating the /etc/hosts file

In addition to the environment variables, Docker adds a host entry for the source container to the /etc/hosts file. Here's a sample entry:

```
$ docker run -t -i --rm --link db:webdb training/webapp /bin/bash
root@aed84ee21bde:/opt/webapp# cat /etc/hosts
172.17.0.7  aed84ee21bde
. . .
172.17.0.5  webdb 6e5cdeb2d300 db
```

Notice two relevant host entries.

- The first is an entry for the `web` container that uses the Container ID as a host name
- The second uses the link alias to reference the IP address of the `db` container.

You can ping the host and try.

`ping webdb`

*Note that if you restart the source container, the linked containers `/etc/hosts` files will be automatically updated with the source container's new IP address, allowing linked communication to continue.
