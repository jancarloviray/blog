---
layout: post
title: Add Firewall to Linux Server with UFW
permalink: blog/add-firewall-to-linux-server-with-ufw/
comments: True
excerpt_separator: <!--more-->
---

So, you have your own server? Setting up a firewall on your server is very important once it is up and running. Thanks to `ufw`, doing this is fairly easy!

<!--more-->

## Install UFW - The Uncomplicated Firewall

`sudo apt-get update`

`sudo apt-get install ufw`

## Setup Defaults

These are the default settings out of the box, but let's just make extra sure that they really are.

`sudo ufw default deny incoming`

`sudo ufw default allow outgoing`

## Let's Enable Some Ports by Service

If you don't need some of these services, you don't have to run them.

```shell
sudo ufw allow OpenSSH
sudo ufw allow http
sudo ufw allow https
sudo ufw allow ssh
```

Wondering what other services your server is running? Run `less /etc/services` to check.

## Let's Enable Some Ports Manually

You typically don't need to define ports if you have specific services running like how we did in the previous section, but here are some examples.

```shell
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

<!--more-->

## Want to read more on Firewalls?

- [UFW](https://help.ubuntu.com/community/UFW)
- [Firewall](https://help.ubuntu.com/community/Firewall)
