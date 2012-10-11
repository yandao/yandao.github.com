---
layout: post
title: "First Things After Fresh Installation of a Linux Server"
description: ""
category: configuration
tags: [sysadmin, linux]
---
{% include JB/setup %}

## Introduction ##

Some good practice after a fresh installation of a Linux-based server. Fundamental stuff that somehow always escape my mind. Applicable for Debian-based and RPM-based distros.


## Add non-root user ##

	# useradd -m <user>
	# passwd <user>


## Allow sudo access ##

	# visudo

		...

		## Allows people in group admin to run all commands
		%admin ALL=(ALL) ALL

	# groupadd admin
	# usermod -a -G admin <user>
	# id <user>


## Setup SSH ##

Install OpenSSH

	$ sudo apt-get install openssh-server

	OR

	$ sudo yum -y install openssh-server

Check that sshd is running and port 22 is open

	$ ps ax | grep sshd
	$ netstat -tulpn | grep :22

		tcp     0     0 0.0.0.0:22     0.0.0.0:*     LISTEN     -  

	$ cat /etc/sysconfig/iptables | grep 22

		-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT

	$ sudo vim /etc/ssh/sshd_config

		...

		PermitRootLogin no

Restart SSH server

	$ sudo service ssh restart

	OR

	$ sudo /sbin/service sshd restart


NOTE: CentOS doesn't include /sbin in $PATH by default for whatever reason. Edit ~/.bash_profile accordingly.


## Done ##

Now we can SSH into the server from a remote host and start setting up stuff.


## References ##

* [UNIX Create User Account](http://www.cyberciti.biz/faq/unix-create-user-account/)
* [CentOS / Red Hat: Sudo Allows People In Group Admin To Run All Commands](http://www.cyberciti.biz/faq/linux-sudo-allows-people-in-group-admin/)
* [CentOS SSH Installation And Configuration](http://www.cyberciti.biz/faq/centos-ssh/)