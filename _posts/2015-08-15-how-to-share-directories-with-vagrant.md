---
layout: post
title: How to share/sync directory inside Vagrant
permalink: blog/how-to-share-sync-directory-inside-vagrant/
excerpt_separator: <!--more-->
comments: True
---

To share directories, add this to your config. `config.vm.synced_folder "host/relative/path", "/guest/absolute/path"`. Below is an example within a full configuration file.

```
Vagrant.configure(2) do |config|
	# note that you can have this config multiple times but
	# it should only be used for source code since there is
	# heavy performance penalty on heavy I/O such as database files

	# first path is the host's path, which can by absolute
	# or relative to project's root directory

	# second path is the guest path, and it must be absolute.
	# It will always be created if it does not exist.
	config.vm.synced_folder ".", "/vagrant"

	config.vm.synced_folder "./some/dir/one", "/one", create:true
	config.vm.synced_folder "./some/dir/two", "/two", create:true

	# note that you can also add type. NFS is the faster
	# bidirectional file syncing. In order for this to work,
	# the host machine must have nfsd installed. It comes
	# preinstalled on OSX and is a simple package install on Linux
	config.vm.synced_folder "." "/vagrant", type: "nfs"

	# owner/group
	config.vm.synced_folder "." "/vagrant", owner: "root", group: "root"
end
```
<!--more-->

```
# To reconfigure the guest machine, vagrant reload must be run.
# This halts the machine and starts it up again with
# the new configuration.
# It skips the initial step to clone the box since it is
# already created.
vagrant reload
```
