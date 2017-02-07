---
layout: post
title: Simple Way to Install VespaCP in Ubuntu
permalink: blog/simple-way-to-install-vespacp-in-ubuntu/
comments: True
excerpt_separator: <!--more-->
---

Vespa is one of the simplest and straightforward control panels in the market. Best of all, it is open source! Why did I choose it? I played around with 3 control panels and this did the job out of the box, without bugs or failures. Here's a quick way to install it in your Ubuntu server. For more features, I suggest ISPConfig. If you want to go for paid? cPanel is awesome. Before you continue, check out my guide on [how to secure your linux server](http://www.jancarloviray.com/blog/harden-and-secure-your-linux-server-ubuntu/).

<!--more-->

```shell
# connect to your server from your local computer
ssh root@your_server_ip
```

```shell
# remove default `admin` group otherwise installation will fail
groupdel admin

# download the install script
curl -O http://vestacp.com/pub/vst-install.sh

# run it!
bash vst-install.sh

# go to your control panel
# example.com:8083
```

Let me know in the comments if you have trouble installing it.
