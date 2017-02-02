---
layout: post
title: Simple Way to Install VespaCP in Ubuntu
permalink: blog/simple-way-to-install-vespacp-in-ubuntu/
comments: True
excerpt_separator: <!--more-->
---

VespaCP is one of the simplest and straightforward control panels in the market. Best of all, it is open source and free!

Here's a quick way to install it in your Ubuntu server

```shell
# connect to your server
ssh root@your_server_ip
# remove default `admin` group otherwise it will not work
groupdel admin
# download the install script
curl -O http://vestacp.com/pub/vst-install.sh
# run it!
bash vst-install.sh
# go to your control panel
# example.com:8083
```

Let me know in the comments if you have trouble installing it.
