---
layout: post
title: "Log Rotation in OS X"
description: ""
category: quick tips
tags: [osx, logrotate]
---
{% include JB/setup %}


## How-to

OS X uses [newsyslog](http://www.manpagez.com/man/8/newsyslog/) utility instead of [logrotate](http://linuxcommand.org/man_pages/logrotate8.html) as typically used in Linux-based systems.

The configuration file for newsyslog can be found at /etc/newsyslog.conf.

	$ sudo vim /etc/newsyslog.conf

In newsyslog, each entry is defined in a single line according to the following format:

	logfilename  [owner:group]  mode  count  size  when  flags  [/pid_file]  [sig_num]

A sample entry can then be defined as such (excluding the columns surrounded by [ ]):

	/var/log/named.stats    640  5  1000  *  J

Quick explanation for the entry above:

* Mode 640 is defined according to the UNIX file permission settings, rw for owner and r for group members.
* Keep up to a max. of 5 log files.
* Rotate when current log file reaches 1000 KB.
* when = * means that the rotation can happen at any time.
* flags = J means that the older files are to be compressed using bzip2.

## Reference

* [Automatic Log Rotation in Mac OS X](http://wiki.springsurprise.com/2011/10/15/automatic-log-rotation-in-mac-os-x/)