---
layout: post
title: How to update all Docker images
permalink: blog/how-to-update-all-docker-images
comments: True
---

Currently, Docker does not have a command to automatically update all images so we will have to do some good old fashioned script piping: `docker images | grep -v REPOSITORY | awk '{print $1}' | xargs -L1 docker pull`. Also, Docker does not overwrite old images, but rather removes the tag. In order to cleanup old images, run `docker images | grep "<none>" | awk '{print $3}' | xargs -L1 docker rmi`. Note that you must wait for the update process to finish. This is what each command does:
- `docker images` lists all images in the system
- `grep -v REPOSITORY` removes the header "REPOSITORY   TAG   IMAGE ID....."
- `awk '{print $1'}` prints the first column which is the image name
- `xargs -L1 docker pull` passes each line to the command `docker pull`

