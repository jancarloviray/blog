---
layout: post
title: Simple Way to Install VespaCP in Ubuntu
permalink: blog/simple-way-to-install-vespacp-in-ubuntu/
comments: True
excerpt_separator: <!--more-->
---

Vespa is one of the simplest and straightforward control panels in the market. Best of all, it is open source! Here's a quick way to install it in your Ubuntu server. Why did I choose Vespa? I played around with 3 control panels and this did the job out of the box, without bugs, uncertainties or unclear documentation. In fact, you don't even need a documentation to manage this - the UI is just great. For more features, I suggest ISPConfig. If you want to go for paid? cPanel is awesome.

<!--more-->

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
