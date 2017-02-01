---
layout: post
title: How to Harden and Secure Your Linux Server (Ubuntu/Debian)
permalink: blog/harden-and-secure-your-linux-server-ubuntu/
comments: True
excerpt_separator: <!--more-->
---

## Set Your Domain Name Server

First, check out your domain settings from your registrar (Godaddy, Namecheap, etc) and see if the **Domain Name Server** options are correct. If not, change it to point to your server. For example, if your servers are in Digital Ocean, change the domain servers to: **ns1.digitalocean.com, ns2.digitalocean.com, ns3.digitalocean.com**. Note that you still have to configure the network settings in your server provider so that connections will direct to your server IP.

Once you are done, run `whois example.com` to see if the "Name Server" key matches the ones you entered. If not, wait an hour or two for the changes to propagate. But don't worry about that for now - we can do the rest of the process without that.

## Configure Your Domain

Note that these options must be done through your server provider. First, add a domain by entering a valid fully qualified domain name that you own, ie: `example.com`. Then, add **A record** to set a host name and **CNAME records** to add aliases like if you want to prepend "www". If you need to set up a mail server, add **MX Records**. Your server provider should have detailed documentations on this.

## Login to your Server from your Local Machine

Let's get to the good stuff. If you are using accessing your server from a web terminal provided by your server provider, you don't need to do this. If you are going to be accessing your server from your local machine, do this step.

`ssh root@your_server_ip`

## Let's update Apt-Get

Once you're in, update apt-get.

`apt-get update`

## Install Fail2Ban to Prevent Active Attacks

Let's install `fail2ban`. The default configurations are enough to cover you, so we'll just install it. Read more about it if you'd like to customize the service.

`apt-get install fail2ban`

Installing it should automatically make it run. If not, run `sudo service fail2ban start`

## Create a New User

Never work under the root user. It is not secure to do so, and you may cause more damage than you would want - if not now, most likely later. So, let's create a new user with regular privileges. Make sure you enter a strong password.

`adduser jancarloviray`

## Add Root Privileges

Let's add root privileges, or "superuser" privileges to our regular account. This will allow us to run administrative privileges by prepending the word `sudo` before each command.

We need to add `jancarloviray` to the **sudo group**. Users who belong to the **sudo group** are allowed to use the `sudo` command.

`usermod -aG sudo jancarloviray`

An alternative command that does this is `gpasswd -a jancarloviray sudo`. They both accomplish the same thing.

If you are not able to add the user to the group immediately, you may have to edit the `/etc/sudoers` file to uncomment the group name. Do this by running `sudo visudo`.

## Add Public Key Authentication

Set up public key authentication for your new user. This allows us to restrict access by requiring a private SSH key to log in. To do this, you must enter these commands at your **local machine**

`ssh-keygen`

Running that command generates the following files:

```
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

Do not share the private key! Now, copy the public key to your new server.

`ssh-copy-id jancarloviray@your_server_ip`

If you have a mac and you don't have that command installed, run `brew install ssh-copy-id`. If that doesn't work, then you have to do things manually. It's a little bit more work, but you'll be grateful for it. Let's manually install the key:

```shell
# in your local computer, print the contents of your public key
cat ~/.ssh/id_rsa.pub

# copy that in your clipboard

# on your server, switch to the new user
# this will bring your to your new user's home directory
su - jancarloviray

# create .ssh directory and restrict its permission
mkdir ~/.ssh
chmod 700 ~/.ssh

# open the authorized_keys file and insert your public key there
vim ~/.ssh/authorized_keys

# restricts the permission of the authorized_keys
chmod 600 ~/.ssh/authorized_keys

# exit and go back to the root user
exit
```

## Disable Password Authentication

Let's make sure no one can really enter in your server - except yourself of course! Well, technically, only to those who have access to your local computer. Here, we will disable logging in by password. This means that the only way to enter your server is if you have the private key that pairs with the public key you copied in *authorized_keys*. We created that in the previous step.

It is very important to note to only do this AFTER you have completed the previous step, otherwise you are forever locked out from the server.

Let's now edit ssh config:

`sudo vim /etc/ssh/sshd_config`

Change the line `# PasswordAuthentication no` to `PasswordAuthentication yes`. Notice that it is now "yes" and is uncommented.

Reload the SSH daemon:

`sudo systemctl reload sshd`

## Set Up a Basic Firewall

Install `ufw` with `sudo apt-get install ufw`

Set up some defaults. These are out-of-the-box settings, but let's just make sure:

`sudo ufw default deny incoming`

`sudo ufw default allow outgoing`

Check out your current services:

`sudo ufw app list`

Enable the services that you'll need to have connections on:

```shell
sudo ufw allow OpenSSH
sudo ufw allow http
sudo ufw allow https
```

If you need to enable certain ports explicitly:

```shell
# if you want to allow mail, for example:
sudo ufw allow 25   #smtp
sudo ufw allow 143  #imap
sudo ufw allow 993  #imaps
sudo ufw allow 110  #incoming pop3
sudo ufw allow 995  #incoming pop3s
```

Enable the firewall:

`sudo ufw enable`

Check the current status of your firewall:

`sudo ufw status`

That's all for the basics! Let me know if the comments below if you would like to learn more steps! There is always another step to secure and harden your server!
