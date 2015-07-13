---
layout: post
title: Run Terminal in Docker
permalink: blog/run-terminal-in-docker
comments: True
---

`docker run -i -t ubuntu:latest /bin/bash`

-i flag tells docker to connect to STDIN on the container
-t flag specifies to get a pseudo-terminal

note that you need to run a terminal process as your command (i.e.: /bin/bash)
