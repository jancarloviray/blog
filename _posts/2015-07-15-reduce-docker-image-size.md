---
layout: post
title: How To Reduce Docker Image Size?
permalink: blog/reduce-docker-image-size
comments: True
---

Docker containers built from Dockerfiles can grow very big in size. There are a few simple tricks to cut back on some of the container fat. Here are some of the ones I've used.

## Clean the APT

```
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

## Flatten the Image

```
ID=$(docker run -d image-name /bin/bash)
docker export $ID | docker import – flat-image-name
```

Then, you can save it for backup too.

```
ID=$(docker run -d image-name /bin/bash)
(docker export $ID | gzip -c > image.tgz)
gzip -dc image.tgz | docker import - flat-image-name
```
