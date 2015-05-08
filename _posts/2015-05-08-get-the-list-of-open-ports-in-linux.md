---
layout: post
title: "Get the List of Open Ports in Linux"
description: ""
category: quick tips
tags: [linux, centos]
---
{% include JB/setup %}


## The command

    $ netstat -ntlu 

For some distros, e.g. CentOS 7.x, the command has been deprecated and replaced by the following:

    $ ss -ntlu


## References

* [Get a list of Open Ports in Linux - Super User](http://superuser.com/a/529844)
* [Deprecated Linux networking commands and their replacements](https://dougvitale.wordpress.com/2011/12/21/deprecated-linux-networking-commands-and-their-replacements/)