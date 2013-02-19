---
layout: post
title: "Running uWSGI via Supervisor + upstart"
description: ""
category: configuration
tags: [ubuntu, supervisor, upstart, uwsgi]
---
{% include JB/setup %}


## Introduction

Read the official documentations directly to get a better idea on what [Supervisor](http://supervisord.org) and [upstart](https://help.ubuntu.com/community/UpstartHowto) do. Basically they launch processes upon startup and manage them as changes to source code and configuration files are being made and deployed.

Since upstart is only available on Ubuntu-based systems, this post may only apply on those systems.

## Install Supervisor

Supervisor may be installed directly as a system process via apt-get, or installed in a virtual environment using pip.

	$ sudo apt-get install supervisor

	OR

	(virtual_env_name)$ pip install supervisor


## Tell Supervisor to handle uWSGI Emperor instance

Based on the [documentation](http://supervisord.org/installing.html#creating-a-configuration-file), create the configuration file by following these steps:

	$ echo_supervisord_conf
	$ sudo echo_supervisord_conf > /etc/supervisord.conf

Add the uWSGI Emperor process to the configuration file:

	$ sudo vim /etc/supervisord.conf

		...

		[program:uwsgi-emperor]
		command=/path/to/venv/containing/bin/uwsgi --emperor "/path/to/uwsgi/conf/files" --die-on-term --master --uid <process_owner> --gid <process_owner_group> --pidfile /tmp/uwsgi-emperor.pid --logto /var/log/uwsgi/emperor.log
		autostart=true
		autorestart=true
		redirect_stderr=true


## Launch Supervisor via upstart

Create an upstart script to launch Supervisor at /etc/init, e.g. /etc/init/supervisor.conf:

	start on runlevel [2345]
	stop on runlevel [!2345]

	respawn

	script
    	exec start-stop-daemon --start --chuid <process_owner> --exec /path/to/venv/containing/bin/supervisord -- --nodaemon --configuration /etc/supervisord.conf
	end script

The service will be run automatically on startup, or manually via the following command:

	$ sudo start supervisor

	NOTE: The name of the service corresponds to the name of the upstart script created earlier.


## Run uWSGI via upstart directly

There may be times where using a combination of Supervisor and upstart to manage uWSGI applications may seem like an overkill. In that case, it may be sufficient to [run them directly via upstart](http://uwsgi-docs.readthedocs.org/en/latest/Upstart.html) by including it in an upstart script:

	$ sudo vim /etc/init/uwsgi.conf

		description "SDP adaptor"
		start on runlevel [2345]
		stop on runlevel [!2345]

		script
    		exec start-stop-daemon --start --chuid <process_owner> --exec /path/to/venv/containing/bin/uwsgi -- --emperor /path/to/uwsgi/conf/files --master --die-on-term --uid <process_owner> --gid <process_owner_group> --pidfile /tmp/uwsgi-emperor.pid --logto /var/log/uwsgi/emperor.log
		end script

uWSGI can then be run via the following command:

	$ sudo start uwsgi


## Run service via upstart as non-root user

By default, running a service via upstart requires **sudo**. but we can make a few modifications to allow non-root users to run the services directly. As usual, be aware of any security implications.

Modify dbus Upstart configuration file at /etc/dbus-1/system.d/Upstart.conf to be less restrictive:

	<?xml version="1.0" encoding="UTF-8" ?>
	<!DOCTYPE busconfig PUBLIC
  	"-//freedesktop//DTD D-BUS Bus Configuration 1.0//EN"
  	"http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">

	<busconfig>
  		<!-- Only the root user can own the Upstart name -->
  		<policy user="root">
    		<allow own="com.ubuntu.Upstart" />
  		</policy>

	 	<!-- Allow any user to introspect Upstart's interfaces, to obtain the
             values of properties (but not set them) and to invoke selected
             methods on Upstart and its jobs that are used to walk information. -->
  		<policy context="default">
    		<allow send_destination="com.ubuntu.Upstart"
	   			   send_interface="org.freedesktop.DBus.Introspectable" />
    		<allow send_destination="com.ubuntu.Upstart"
	    		   send_interface="org.freedesktop.DBus.Properties" />
    		<allow send_destination="com.ubuntu.Upstart"
	   			   send_interface="com.ubuntu.Upstart0_6" />
    		<allow send_destination="com.ubuntu.Upstart"
	   			   send_interface="com.ubuntu.Upstart0_6.Job" />
    		<allow send_destination="com.ubuntu.Upstart"
	   			   send_interface="com.ubuntu.Upstart0_6.Instance" />
  		</policy>
	</busconfig>

Modify sudoers file to allow non-root user to run specific services only:

	$ sudo visudo

		...

		%admin ALL=(ALL) NOPASSWD: /sbin/start <service_name>, /sbin/stop <service_name>

Now you should be able to run the specific services as non-root user without sudo:

	$ start <service_name>


## References

* [Upstart Intro, Cookbook and Best Practises](http://upstart.ubuntu.com/cookbook/)
* [Upstart Howto](https://help.ubuntu.com/community/UpstartHowto)
* [Supervisor: A Process Control System](http://supervisord.org)
* [Running uWSGI via Upstart](http://uwsgi-docs.readthedocs.org/en/latest/Upstart.html)