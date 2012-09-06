---
layout: post
title: "Quick Setup Apache + uWSGI in Ubuntu"
description: ""
category: configuration
tags: [apache, python, ubuntu, uwsgi]
---
{% include JB/setup %}

## Introduction

Just a quick reference guide showing the steps and commands for configuring uWSGI to run behind Apache. 


## Install Apache

A fresh install of Ubuntu Server doesn't come with Apache installed by default, which is fair enough because we may want to use another web server. Plus Apache can be easily installed by issuing:

	sudo apt-get install apache2 apache2-threaded-dev 

* NOTE: The apache2-threaded-dev is required since it contains the apxs tool that is required to compiling the mod_uwsgi module.


## Compile mod_uwsgi Module

Download the stable version of uWSGI source (1.2.5 at the time of writing):

	$ wget http://projects.unbit.it/downloads/uwsgi-1.2.5.tar.gz
	$ tar zxvf uwsgi-1.2.5.tar.gz

Copy mod_uwsgi.c to the Apache modules directory and compile it there using apxs:
	
	$ sudo cp mod_uwsgi.c /usr/lib/apache2/modules
	$ sudo apxs2 -i -a -c mod_uwsgi.c

Include mod_uwsgi into Apache configuration:

	$ sudo vim /etc/apache2/sites-available/default

		...

		<Location /bin>
			SetHandler uwsgi-handler
			uWSGISocket 127.0.0.1:5000
		</Location>

		ScriptAlias ...

Restart Apache

	$ sudo service apache2 restart

Apache should now run with mod_uwsgi enabled. From the Apache configuration above, any requests that comes through &lt;domain_name&gt;/bin will be handled by uWSGI, and passed to the WSGI application that listens on **TCP socket 127.0.0.1 on port 5000**.


## Done

Now we are ready to proceed with setting up the development environment for creating Python-based web applications.


## Reference

* [Quickstart - uWSGI](http://projects.unbit.it/uwsgi/wiki/Quickstart)