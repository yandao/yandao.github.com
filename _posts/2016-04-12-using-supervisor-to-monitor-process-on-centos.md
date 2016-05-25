---
layout: post
title: "Using Supervisor to monitor process on CentOS"
description: ""
category: tutorial
tags: [python, centos, supervisor]
---
{% include JB/setup %}



## Prerequisite

Python 2.7 or newer. For CentOS 6.x, it may be necessary to [install Python 2.7 from external source](/tutorial/2016/04/11/howto-install-python-27-on-centos-6x).


## Install Supervisor with Python 2.7

    # First, install the required libraries.
    $ sudo yum install -y gcc libffi-devel openssl-devel

    # If you're installing Python 2.7 from SCL repo, it is necessary to enable it.
    $ . /opt/rh/python27/enable
    $ cd /opt/rh/python27/root/usr/bin/

    # Install Supervisor and its dependencies.
    # For CentOS 6.x
    $ sudo LD_LIBRARY_PATH=$LD_LIBRARY_PATH ./pip2.7 install 'requests[security]'
    $ sudo LD_LIBRARY_PATH=$LD_LIBRARY_PATH ./pip2.7 install supervisor

    # For CentOS 7
    $ pip install 'requests[security]'
    $ pip install supervisor


## Create Supervisor main configuration file

(**file: /etc/supervisord.conf**)

    $ echo_supervisord_conf > /home/sekaimon/supervisord.conf
    $ sudo mv /home/sekaimon/supervisord.conf /etc/supervisord.conf
    $ sudo sed -i "s/\/tmp\/supervisord.log/\/var\/log\/supervisord\/supervisord.log/g" /etc/supervisord.conf
    $ sudo sed -i "s/\;\[include\]/\[include\]/g" /etc/supervisord.conf
    $ echo "files = /etc/supervisord.d/*.conf" >> /etc/supervisord.conf


## Create the configuration file for the process to monitor

(**file: /etc/supervisord.d/selenium.conf**)

The following is an example of a configuration for monitoring Selenium grid. Since Selenium grid consists of hub and node processes, it is necessary to monitor those processes as a group. In this example, the Selenium server JAR file is stored in /usr/lib/selenium.

    [program:seleniumhub]
    command=su <user> -c "java -jar /usr/lib/selenium/selenium-server-standalone.jar -role hub -nodeTimeout 600 > /dev/null 2>&1"
    numprocs=1
    user=root
    stdout_logfile=/var/log/supervisord/selenium.log
    stderr_logfile=/var/log/supervisord/selenium.log
    redirect_stderr=true
    autostart=true
    autorestart=true
    startretries=0
    killasgroup=true
    stopwaitsecs=1
    priority=998

    [program:seleniumnode]
    environment=DISPLAY=:99
    command=su <user> -c "java -jar /usr/lib/selenium/selenium-server-standalone.jar -role node -hub http://127.0.0.1:4444/grid/register > /dev/null 2>&1"
    numprocs=1
    user=root
    stdout_logfile=/var/log/supervisord/selenium.log
    stderr_logfile=/var/log/supervisord/selenium.log
    redirect_stderr=true
    autostart=true
    autorestart=true
    startretries=0
    killasgroup=true
    stopwaitsecs=1
    priority=998

    [group:selenium]
    programs=seleniumhub,seleniumnode
    priority=999


## Create init script for Supervisor

(**file: /etc/init.d/supervisord**)

    #!/bin/sh
    #
    # /etc/rc.d/init.d/supervisord
    #
    # Supervisor is a client/server system that
    # allows its users to monitor and control a
    # number of processes on UNIX-like operating
    # systems.
    #
    # chkconfig: - 64 36
    # description: Supervisor Server
    # processname: supervisord

    # Source init functions
    . /etc/rc.d/init.d/functions

    prog="supervisord"

    # For CentOS 6.x, it is necessary to enable Python 2.7.
    # Otherwise, simply specify the full path to supervisord.
    python27_root="/opt/rh/python27"
    prog_bin=". ${python27_root}/enable;${python27_root}/root/usr/bin/supervisord"
    PIDFILE="/var/run/$prog.pid"

    start()
    {
           echo -n $"Starting $prog: "
           daemon $prog_bin --pidfile $PIDFILE
           [ -f $PIDFILE ] && success $"$prog startup" || failure $"$prog startup"
           echo
    }

    stop()
    {
           echo -n $"Shutting down $prog: "
           [ -f $PIDFILE ] && killproc $prog || success $"$prog shutdown"
           echo
    }

    case "$1" in

     start)
       start
     ;;

     stop)
       stop
     ;;

     status)
           status $prog
     ;;

     restart)
       stop
       start
     ;;

     *)
       echo "Usage: $0 {start|stop|restart|status}"
     ;;

    esac


Enable the init script and create the necessary directories:

    $ sudo chmod +x /etc/init.d/supervisord
    $ sudo chkconfig --add supervisord
    $ sudo chkconfig supervisord on

    $ sudo mkdir -p /etc/supervisord.d
    $ sudo mkdir -p /var/log/supervisord


## Start supervisord to start monitoring processes

    $ sudo service supervisord start

