---
layout: post
title: Add Firewall to Linux Server with UFW
permalink: blog/add-firewall-to-linux-server-with-ufw/
comments: True
excerpt_separator: <!--more-->
---

So, you have your own server? Setting up a firewall on your server is very important once it is up and running. Thanks to `ufw`, doing this is fairly easy! Here's a basic setup flow:

## Setup Basic Firewall

`sudo apt-get update`

`sudo apt-get install ufw`

## Setup Defaults

These are the default setups out of the box, but let's just make extra sure.

`sudo ufw default deny incoming`

`sudo ufw default allow outgoing`

## Allow Common Ports

Feel free to not run some of these commands if you are not using the services.

```shell
sudo ufw allow OpenSSH
sudo ufw allow http
sudo ufw allow https
sudo ufw allow ssh

# using mail? allow these ports
sudo ufw allow 25   #smtp
sudo ufw allow 143  #imap
sudo ufw allow 993  #imaps
sudo ufw allow 110  #incoming pop3
sudo ufw allow 995  #incoming pop3s
```

## Enable UFW

`sudo ufw enable`

That's it!
