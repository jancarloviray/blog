---
layout: post
title: Send Email in Linux Without An SMTP Server
permalink: blog/send-email-in-linux-without-an-smtp-server/
comments: True
excerpt_separator: <!--more-->
---

Typically, to send email, you would need an smtp server or connect to one. Let's use [Postfix](http://www.postfix.org/) to bypass that need. Basically, Postfix is a free and open-source mail transfer agent (MTA) that routes and delivers electronic mail. Let's start.

```shell
apt-get update
# install postfix
apt-get install postfix
```

As you continue the installation, Postfix prompts will appear. Here are some information to take note:

- **General type of mail configuration?**: Choose `Internet Site` - this matches our infrastructure needs.
- **System mail name**: Enter your FQDN, ie: `example.com` - this is the base domain used to construct a valid email address when only the account portion of the address is given. If the hostname of our server is mail.example.com, we want the system mail name to be `example.com` so that given a `user`, postfix will use the address `user@example.com`
- **Root and postmaster mail recipient**: this is the Linux account that will be forwarded mail addressed to `root@` and `postmaster@`. Use your primary account for this.
- **Other destinations to accept mail for**: This defines the mail destinations that this Postfix instance will accept. If you need to add any other domains that this server will be responsible for receiving, add those here, otherwise, the default should work fine.
- **Force synchronous updates on mail queue?**: No
- **Local Networks**: Should be `127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128` This is a list of the networks that your mail server is configured to relay messages for. The default should work for most scenarios. If you choose to modify it, make sure to be very restrictive in regards to the network range.
- **Mailbox size limit**: This can be used to limit the size of messages. Setting it to "0" disables any size restriction.
- **Internet protocols to use**: Choose whether to restrict the IP version that Postfix supports. We'll pick "all" for our purposes.

Check if it's installed: `telnet localhost 25`. If not, start postfix with `sudo postfix start`.

Now, let's configure postfix. Based on your entries, you may not have to modify these. Check out these settings:

**myhostname:** `example.com` - this is the hostname of your machine, but don't put the full hostname. If your machine hostname is mail.mydomain.com you will only use mydomain. Put your FQDN here like "example.com"

**virtual_alias_map:** If you would like to configuring mail to be forwarded to other domains or wish to deliver to addresses that don't map 1-to-1 with system accounts, we can remove the alias_maps parameter and replace it with virtual_alias_maps. We would then need to change the location of the hash to /etc/postfix/virtual, `virtual_alias_maps = hash:/etc/postfix/virtual`

**mydestination:** - `mydestination = $myhostname, localhost.$mydomain, $mydomain` this parameter specifies what destinations this machine will deliver locally. You can use the default. This should have been configured based on the FQDN you entered. For forwarding emails to another address other than your server, this is not required, only `virtual_alias_domains` and `virtual_alias_maps` are required. If you want to receive mail in this server, you may need to comment this line if it is causing errors.

**mynetworks:** - `mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128` this entry will define authorized destinations that mail can be relayed from. This defines the computers that are able to use this mail server. It should be set to local only (127.0.0.0/8 and the other representations). Modifying this to allow other hosts to use this is a huge vulnerability that can lead to extreme cases of spam. This should have been set automatically, but it should be something like this: `mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128`

**mydomain:** if your mail server serves up mail to your entire domain, you will need to add another entry, something like `mydomain = mydomain.com`

Once you are settled, restart Postfix with `systemctl restart postfix`

Install `mail` and other utilities: `apt-get install mailutils`

And you're done! Now, try sending a mail! `echo "This is the body of the email" | mail -s "This is the subject line" -r from@example.org to@recipient.com`

Note that you may need to change your computer hostname to match the hostname you entered. Check out this [link](http://askubuntu.com/questions/9540/how-do-i-change-the-computer-name) on how to do so.

Now, let's try mapping email accounts to linux system accounts. Append `virtual_alias_maps` settings in postfix config.

`postconf -e 'virtual_alias_maps= hash:/etc/postfix/virtual'`

Edit `/etc/postfix/virtual` by mapping email to system username like this:

```
contact@example.com joe
admin@example.com jane
```

Apply mapping with `postmap /etc/postfix/virtual`

Restart postfix: `systemctl restart postfix`

NOTE: you can change email headers as you send mail by adding `-a` param. For example: 

```shell
echo "body of email" | mail -s "Subject Line" -a "From: Admin Name <noreply@example.org>" mail@recipient.com
```
