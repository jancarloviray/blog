---
layout: post
title: How to update all Docker images
permalink: blog/how-to-update-all-docker-images
comments: True
---

Currently, Docker does not have a command to do this so we will have to do some good old fashioned command piping. **To automatically update all images:**<br/><br/>```docker images | grep -v REPOSITORY | awk '{print $1}' | xargs -L1 docker pull```<br/><br/>Docker does not overwrite old images for us. **To cleanup old images**:<br/><br/>`docker images | grep "<none>" | awk '{print $3}' | xargs -L1 docker rmi`<br/><br/>Note that you must wait for the update process to finish. Here's a summary of what each command does...

- `docker images` lists all images in the system
- `grep -v REPOSITORY` removes the header "REPOSITORY   TAG   IMAGE ID....."
- `awk '{print $1'}` prints the first column which is the image name
- `xargs -L1 docker pull` passes each line to the command `docker pull`

